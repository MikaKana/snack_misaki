# Snack Misaki — Backend

## 概要
バックエンドは **RunPod Serverless (GPU, Python 3.11, Docker)** をベースに実装されています。
Stage 2 以降で利用され、フロントエンドからの入力を受け取り、RunPod 上の小型 LLM または外部 LLM API を用いて応答を生成します。

---

## 技術スタック
- **RunPod Serverless (GPU, Python 3.11)**: GPU 常駐で低レイテンシなサーバーレス推論
- **Docker**: ローカル開発およびデプロイの環境統一（RunPod テンプレートとして利用）
- **Phi-3 Mini / Mistral 7B**: RunPod 上での推論モデル。通常は Phi-3 Mini、課金や利用条件クリア時に Mistral 7B へ切替
- **OpenAI API / AWS Bedrock / HuggingFace Hub**: 外部 LLM API との連携

---

## 機能概要
1. **イベント受信**
   - RunPod Serverless エンドポイントがフロントエンドからの入力を受け取る

2. **RunPod での小型 LLM 呼び出し (Stage 2)**
   - 未対応入力を RunPod Serverless 上の Phi-3 Mini で処理
   - 課金/利用条件を満たす場合、同一 RunPod 環境で Mistral 7B に切り替えて応答

3. **外部 API 呼び出し (Stage 3)**
   - RunPod 上のモデルで処理困難な入力は外部 LLM API にエスカレーション

4. **レスポンス返却**  
   - 処理結果を JSON としてフロントエンドに返す  

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
- **小型 LLM の切替**: RunPod 上で Phi-3 Mini を基本とし、課金/利用条件で Mistral 7B に自動切替
- **外部 API の利用有無**: 環境変数で OpenAI / Bedrock / HuggingFace を切替
- **レスポンスロジック**: `app/handler.py` 内で、条件判定（課金状況・利用閾値）とモデル選択を制御

---

## 今後の拡張
- **認証/認可**: API Gateway + Cognito 連携  
- **監視/ロギング**: CloudWatch 連携  
- **CI/CD**: GitHub Actions から Lambda デプロイの自動化  

