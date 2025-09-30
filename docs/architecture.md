# Snack Misaki — Architecture

## アーキテクチャ概要
Snack Misaki は、フロントエンド・バックエンド・LLM の 3 層構成で設計されています。  
進化型の 3 段階ステージを通して、軽量 PoC から高度な外部 LLM 連携まで拡張可能です。

---

## 全体構成図

```mermaid
flowchart TD

subgraph Frontend["Frontend: React + TypeScript"]
  F1[ユーザー入力]
  F2[定型文マッチング]
end

subgraph Backend["Backend: AWS Lambda + Python"]
  B1[小型LLM - llama.cpp / GPT4All]
  B2[外部LLM API - OpenAI / Bedrock / HuggingFace]
end

F1 --> F2
F2 -->|定型文ヒット| F2
F2 -->|未対応入力| B1
B1 -->|応答| F1
B1 -->|処理困難/高度要求| B2
B2 -->|応答| F1
```

---

## ステージごとのアーキテクチャ

### Stage 1: フロントエンドのみ
- **対象**: フロントエンド
- **内容**: React による定型文レスポンス UI
- **目的**: まずはシンプルな PoC として対話体験を確認

```mermaid
flowchart TD
F1[ユーザー入力] --> F2[定型文マッチング]
F2 --> F1
```

---

### Stage 2: フロントエンド + バックエンド
- **対象**: フロントエンド + バックエンド
- **内容**:  
  - フロントエンドは Stage 1 を継続利用  
  - バックエンド (Lambda/Python) を追加し、小型 LLM を呼び出す  
- **目的**: 定型文で対応できない質問に小型 LLM で応答

```mermaid
flowchart TD
F1[ユーザー入力] --> F2[定型文マッチング]
F2 -->|定型文ヒット| F1
F2 -->|未対応入力| B1[小型LLM]
B1 --> F1
```

---

### Stage 3: 外部 LLM API 連携
- **対象**: バックエンド中心
- **内容**:  
  - フロントエンドは Stage 1/2 を継続利用  
  - バックエンドに外部 LLM API を統合し、小型 LLM で難しい質問は外部にエスカレーション  
- **目的**: 高度な応答を実現し、本格運用に対応可能

```mermaid
flowchart TD
F1[ユーザー入力] --> F2[定型文マッチング]
F2 -->|定型文ヒット| F1
F2 -->|未対応入力| B1[小型LLM]
B1 -->|応答| F1
B1 -->|処理困難| B2[外部LLM API]
B2 --> F1
```

---

## コンポーネントの役割
- **フロントエンド**: ユーザー入力、定型文レスポンス UI
- **バックエンド**: Lambda + Python、小型 LLM 呼び出し、外部 API 連携
- **外部 API**: 高度な質問や長文応答をサポート

