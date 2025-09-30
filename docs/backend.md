# Snack Misaki — Backend

## 概要
バックエンドは **AWS Lambda (Python 3.11, Docker)** をベースに実装されています。  
Stage 2 以降で利用され、フロントエンドからの入力を受け取り、小型 LLM または外部 LLM API を用いて応答を生成します。

---

## 技術スタック
- **AWS Lambda (Python 3.11)**: サーバーレス基盤でスケーラブルな処理  
- **Docker**: ローカル開発およびデプロイの環境統一  
- **llama.cpp / GPT4All**: 軽量なローカル推論用 LLM  
- **OpenAI API / AWS Bedrock / HuggingFace Hub**: 外部 LLM API との連携  

---

## 機能概要
1. **イベント受信**  
   - Lambda ハンドラがフロントエンドからの入力を受け取る  

2. **小型 LLM 呼び出し (Stage 2)**  
   - 未対応入力を llama.cpp / GPT4All で処理  

3. **外部 API 呼び出し (Stage 3)**  
   - 小型 LLM で処理困難な入力は外部 LLM API にエスカレーション  

4. **レスポンス返却**  
   - 処理結果を JSON としてフロントエンドに返す  

---

## 実行方法（ローカル）
### 1. Docker build
```bash
docker build -t snack-misaki-backend .
```

### 2. Lambda エミュレータ起動
```bash
docker run -p 9000:8080 snack-misaki-backend
```

### 3. イベント送信
```bash
curl -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" -d '{"input":"こんばんは"}'
```

---

## カスタマイズポイント
- **小型 LLM の切替**: `llama.cpp` または `GPT4All` を選択可能  
- **外部 API の利用有無**: 環境変数で OpenAI / Bedrock / HuggingFace を切替  
- **レスポンスロジック**: `app/handler.py` 内で制御  

---

## 今後の拡張
- **認証/認可**: API Gateway + Cognito 連携  
- **監視/ロギング**: CloudWatch 連携  
- **CI/CD**: GitHub Actions から Lambda デプロイの自動化  

