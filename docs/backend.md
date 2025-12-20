# Snack Misaki — Backend

## 概要
バックエンドは **LLM 推論用 (RunPod Serverless, GPU, Python 3.11, Docker)** と **認証/決済用 (AWS Lambda + DynamoDB)** の 2 系統で構成されています。  
**認証/決済 API と RunPod 推論エンドポイントは別ドメイン・別 URL で運用** し、フロントエンドが用途ごとに呼び分けます。Cognito でログインし、Stripe の決済結果を認証/決済 API で参照してプランを判定したうえで、RunPod 上の 2 系統エンドポイント（Phi-3 Mini / Mistral 7B）と、VIP 向けの外部 LLM API へ振り分けます。

---

## エンドポイントの分離
- **認証/決済 API**: API Gateway 経由で公開される Lambda。Cognito トークン検証や Stripe 決済結果の照会専用で、`/auth` や `/billing` のようなパスを持つ。RunPod の推論 URL とは別ドメインで提供し、推論リクエストは受け付けない。
- **LLM 推論エンドポイント**: RunPod Serverless 上でモデルごとに分けた URL（Phi-3 Mini / Mistral 7B）。認証/決済 API と混在させず、推論リクエストのみを受ける。
- **フロントエンドの呼び出し順序**: ログイン後に **認証/決済 API でプランを取得 → プラン別に RunPod または外部 LLM へ送信** の 2 段階で呼び分ける。単一のベース URL に集約しない前提。

---

## 技術スタック
- **RunPod Serverless (GPU, Python 3.11)**: GPU 常駐で低レイテンシなサーバーレス推論
- **Docker**: ローカル開発およびデプロイの環境統一（RunPod テンプレートとして利用）
- **Phi-3 Mini / Mistral 7B**: RunPod 上にそれぞれ専用エンドポイントを用意（無料=Phi-3 Mini / 有料=Mistral 7B）。両モデルは再学習を予定。
- **Claude / GPT / OpenAI API / AWS Bedrock / HuggingFace Hub**: VIP ユーザー向けの外部 LLM API 連携
- **AWS Cognito**: ログインとセッション管理。Hosted UI を想定。
- **API Gateway + Lambda (Python)**: 認証/決済 API を提供。Cognito ID トークンを検証し、プラン情報を返却。
- **DynamoDB**: ユーザー属性・サブスクステータスの永続化。
- **Stripe**: 課金・サブスクリプション管理。Webhook を受けて DynamoDB を更新。

---

## 機能概要
1. **認証/決済 (Lambda + DynamoDB)**
   - フロントエンドが Cognito でログイン
   - Stripe で決済/サブスクを実行、Webhook で Lambda が受信し DynamoDB にプランを保存
   - API Gateway 経由の Lambda がユーザーのプラン種別（無料/有料/VIP）を返却

2. **イベント受信 (LLM バックエンド)**
   - フロントエンドが認証/決済 API から得たプランで振り分け、対象エンドポイントに POST

3. **RunPod エンドポイントでの推論**
   - 無料ユーザー: Phi-3 Mini エンドポイントが受信し推論
   - 有料ユーザー: Mistral 7B エンドポイントが受信し推論
   - いずれも再学習モデルに差し替え予定

4. **外部 API 呼び出し (VIP)**
   - VIP ユーザーの入力は Claude / GPT など外部 LLM API へ中継

5. **レスポンス返却**  
   - 各エンドポイントが処理結果を JSON としてフロントエンドに返す  

---

## 実行方法（ローカル）
### 1. コンテナイメージのビルド
```bash
docker build -t snack-misaki-backend .
```

### 2. RunPod Serverless への登録
- ビルドしたイメージをコンテナレジストリへ push
- RunPod Serverless のテンプレートとして登録し、エンドポイントを作成

### 3. エンドポイント呼び出し
```bash
curl -X POST "https://api.runpod.ai/v2/<ENDPOINT_ID>/run" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${RUNPOD_API_KEY}" \
  -d '{"input":{"text":"こんばんは"}}'
```

### 4. 認証/決済 API のデプロイ (AWS)
- **Cognito**: ユーザープールと Hosted UI を作成。フロントエンドは ID トークンを取得。
- **API Gateway + Lambda (Python)**:  
  - Cognito のトークン検証を行い、DynamoDB からプランを取得して返却。  
  - Stripe の Webhook を受ける Lambda で DynamoDB のプラン/ステータスを更新。  
- **DynamoDB**: `user_id` をパーティションキーとし、`plan`, `expires_at`, `stripe_status` などを保存。
- **Stripe**: サブスク/決済。`checkout.session.completed` を Lambda で処理し、DynamoDB に反映。

---

## カスタマイズポイント
- **エンドポイント分離**: Phi-3 Mini（無料）と Mistral 7B（有料）を別 RunPod テンプレートとして管理
- **外部 API の利用有無**: VIP ルートを Claude / GPT / Bedrock / HuggingFace など環境変数で切替
- **レスポンスロジック**: `app/handler.py` などで、入出力フォーマットやリトライ・レート制限を制御
- **再学習パイプライン**: Phi-3 Mini / Mistral 7B は再学習を前提とし、モデル置換時の互換インターフェースを維持
- **認証/決済 API**: Lambda 間で処理を分割（Webhook / プラン取得）、DynamoDB のスキーマや TTL を調整
- **認証連携**: Cognito Hosted UI のコールバック URL をフロントエンド環境に合わせて設定

---

## 今後の拡張
- **監視/ロギング**: CloudWatch 連携（Lambda/RunPod 双方のメトリクス収集）  
- **CI/CD**: GitHub Actions から RunPod テンプレート更新やモデル再学習パイプライン、Lambda デプロイの自動化  
- **課金イベント拡張**: Stripe Billing ポータル連携、決済失敗時の Grace Period ハンドリング  
