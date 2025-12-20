# Snack Misaki — Architecture

## アーキテクチャ概要
Snack Misaki は、フロントエンド・LLM バックエンド・認証/決済バックエンドの 3 層構成で設計されています。
フロントエンドが Cognito でログインし、Stripe で支払い/プランを確認するための **認証/決済バックエンド（AWS Lambda + DynamoDB）** を経由し、ユーザー属性（無料/有料/VIP）を RunPod 上の 2 系統エンドポイント（Phi-3 Mini / Mistral 7B）と外部 LLM API へ振り分けます。
認証/決済バックエンドと RunPod の推論エンドポイントは **別ドメイン・別 URL で提供** し、フロントエンドがログイン後に認証/決済 API へプラン照会 → 推論系へ送信の順に呼び分ける前提です。
3 段階のステージを通じて、無料→有料→VIP と段階的に高性能モデルへ拡張します。

---

## 全体構成図

```mermaid
flowchart TD

subgraph Frontend["Frontend: React + TypeScript"]
  F1[ユーザー入力]
  F2[定型文マッチング]
  F3[ログイン/属性判定\nルーティング]
end

subgraph AuthBackend["Auth/Billing Backend: API Gateway + Lambda + DynamoDB"]
  A1[Cognito Hosted UI / User Pool]
  A2[API Gateway / Lambda\n認証・決済 API]
  A3[DynamoDB\nユーザー属性/プラン管理]
  A4[Stripe\n決済・サブスク]
end

subgraph Backend["LLM Backend: RunPod Serverless (GPU) + Python"]
  B1[RunPod エンドポイント\nPhi-3 Mini\n(無料ユーザー向け)]
  B2[RunPod エンドポイント\nMistral 7B\n(有料ユーザー向け)]
end

subgraph External["External LLM API"]
  B3[Claude / GPT API\n(VIP 向け)]
end

F1 --> F2
F2 -->|定型文ヒット| F2
F2 -->|未対応入力| F3
F3 -->|ログイン/プラン確認| A1
F3 -->|ユーザー属性取得| A2
A2 -->|決済/プラン照会| A4
A2 -->|ユーザー属性更新| A3
F3 -->|無料| B1
F3 -->|有料 (ログイン)| B2
F3 -->|VIP (ログイン)| B3
B1 --> F1
B2 --> F1
B3 --> F1
```

---

## ステージごとのアーキテクチャ

### Stage 1: フロントエンド → Phi-3 Mini
- **対象**: フロントエンド + RunPod (Phi-3 Mini)
- **内容**: 未ログインの無料ユーザーを RunPod Serverless (GPU) 上の Phi-3 Mini エンドポイントへ送る
- **目的**: 低コストで LLM 応答を提供しつつ、UI/ルーティングの骨格を確立

```mermaid
flowchart TD
F1[ユーザー入力] --> F2[定型文マッチング]
F2 -->|定型文ヒット| F1
F2 -->|未対応入力| B1[RunPod GPU\nPhi-3 Mini]
B1 --> F1
```

---

### Stage 2: フロントエンド（ログイン） → 認証/決済 API → Mistral 7B
- **対象**: フロントエンド + 認証/決済バックエンド + RunPod (Phi-3 Mini / Mistral 7B)
- **内容**:
  - Cognito でログインを実装し、Stripe で支払いステータスを管理
  - Lambda + DynamoDB の認証/決済 API がプランを返却し、有料ユーザーは Mistral 7B 専用エンドポイントにルーティング
  - 無料ユーザーは引き続き Phi-3 Mini を利用
- **目的**: 有料ユーザー向けに高性能モデルを提供しつつ、課金と属性管理を確立

```mermaid
flowchart TD
F1[ユーザー入力] --> F2[定型文マッチング]
F2 -->|定型文ヒット| F1
F2 -->|ログイン/プラン確認| A1[Cognito]
A1 --> A2[API Gateway\n+ Lambda]
A2 -->|決済/サブスク| A4[Stripe]
A2 -->|プラン保持| A3[DynamoDB]
A2 --> F3[ユーザー属性返却]
F3 -->|未対応入力 (無料)| B1[RunPod GPU\nPhi-3 Mini]
F3 -->|未対応入力 (有料)| B2[RunPod GPU\nMistral 7B]
B1 --> F1
B2 --> F1
```

---

### Stage 3: フロントエンド（ログイン） → 認証/決済 API → 外部 LLM API
- **対象**: フロントエンド + 認証/決済バックエンド + RunPod + 外部 LLM
- **内容**:  
  - 有料/無料の経路に加え、VIP ユーザーを Stripe のプラン情報に基づき判定し、Claude / GPT など外部 API にルーティング
  - RunPod 上のモデルで得た知見を外部 LLM へのプロンプト拡張に活用
- **目的**: VIP 向けに最も高精度な応答を提供し、上位プランを成立させる

```mermaid
flowchart TD
F1[ユーザー入力] --> F2[定型文マッチング]
F2 -->|定型文ヒット| F1
F2 -->|ログイン/プラン確認| A1[Cognito]
A1 --> A2[API Gateway\n+ Lambda]
A2 -->|決済/サブスク| A4[Stripe]
A2 -->|プラン保持| A3[DynamoDB]
A2 --> F3[ユーザー属性返却]
F3 -->|未対応入力 (無料)| B1[RunPod GPU\nPhi-3 Mini]
F3 -->|未対応入力 (有料)| B2[RunPod GPU\nMistral 7B]
F3 -->|未対応入力 (VIP)| B3[外部 LLM API\nClaude / GPT]
B1 --> F1
B2 --> F1
B3 --> F1
```

---

## コンポーネントの役割
- **フロントエンド**: ユーザー入力、定型文レスポンス UI、ログイン状態に基づくルーティング
- **認証/決済バックエンド**: API Gateway + Lambda + DynamoDB。Cognito でのログイン、Stripe での課金・プラン確認、ユーザー属性の保存/返却を担当
- **LLM バックエンド**: RunPod Serverless (GPU) + Python、Phi-3 Mini / Mistral 7B の 2 系統エンドポイント
- **外部 API**: Claude / GPT など VIP 向けの高精度応答を提供
