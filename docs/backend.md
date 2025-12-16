# Snack Misaki — Backend

## 概要
バックエンドは **RunPod Serverless (GPU, Python 3.11, Docker)** をベースに実装されています。
フロントエンドがユーザー属性を判定し、RunPod 上の 2 系統エンドポイント（Phi-3 Mini / Mistral 7B）と、VIP 向けの外部 LLM API へ振り分けます。

---

## 技術スタック
- **RunPod Serverless (GPU, Python 3.11)**: GPU 常駐で低レイテンシなサーバーレス推論
- **Docker**: ローカル開発およびデプロイの環境統一（RunPod テンプレートとして利用）
- **Phi-3 Mini / Mistral 7B**: RunPod 上にそれぞれ専用エンドポイントを用意（無料=Phi-3 Mini / 有料=Mistral 7B）。両モデルは再学習を予定。
- **Claude / GPT / OpenAI API / AWS Bedrock / HuggingFace Hub**: VIP ユーザー向けの外部 LLM API 連携

---

## 機能概要
1. **イベント受信**
   - フロントエンドがログイン・VIP 判定を行い、対象エンドポイントに POST

2. **RunPod エンドポイントでの推論**
   - 無料ユーザー: Phi-3 Mini エンドポイントが受信し推論
   - 有料ユーザー: Mistral 7B エンドポイントが受信し推論
   - いずれも再学習モデルに差し替え予定

3. **外部 API 呼び出し (VIP)**
   - VIP ユーザーの入力は Claude / GPT など外部 LLM API へ中継

4. **レスポンス返却**  
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

---

## カスタマイズポイント
- **エンドポイント分離**: Phi-3 Mini（無料）と Mistral 7B（有料）を別 RunPod テンプレートとして管理
- **外部 API の利用有無**: VIP ルートを Claude / GPT / Bedrock / HuggingFace など環境変数で切替
- **レスポンスロジック**: `app/handler.py` などで、入出力フォーマットやリトライ・レート制限を制御
- **再学習パイプライン**: Phi-3 Mini / Mistral 7B は再学習を前提とし、モデル置換時の互換インターフェースを維持

---

## 今後の拡張
- **認証/認可**: API Gateway + Cognito 連携（ログイン属性をフロントが取得し、ヘッダーで伝搬）  
- **監視/ロギング**: CloudWatch 連携  
- **CI/CD**: GitHub Actions から RunPod テンプレート更新やモデル再学習パイプラインの自動化  
