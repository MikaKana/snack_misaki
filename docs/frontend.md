# Snack Misaki — Frontend

## 概要
フロントエンドは **Vite + React + TypeScript** をベースに実装されています。  
ユーザーの属性（無料 / 有料 / VIP）を判定し、以下のルートに振り分けます。

- 無料ユーザー: RunPod Serverless (GPU) 上の Phi-3 Mini エンドポイント
- 有料ユーザー（ログイン必須）: RunPod Serverless (GPU) 上の Mistral 7B エンドポイント
- VIP ユーザー（ログイン必須）: Claude / GPT など外部 LLM API

---

## 技術スタック
- **Vite**: 軽量で高速な開発サーバーとビルドツール  
- **React**: コンポーネントベースの UI フレームワーク  
- **TypeScript**: 型安全な開発環境を提供  

---

## 機能概要
1. **ユーザー入力フォーム**  
   テキスト入力欄を提供し、ユーザーが発話を送信可能。  

2. **定型文レスポンス**  
   - フロントエンド側でパターンマッチング  
   - 一致した場合は即時レスポンスを返却（低コストで高速応答）  

3. **バックエンド/外部 API への振り分け**  
   - 未対応入力はユーザー属性に応じて送信先を切替
     - 無料: Phi-3 Mini（RunPod Serverless）
     - 有料: Mistral 7B（RunPod Serverless）
     - VIP: Claude / GPT など外部 API
   - フロントエンドでのログイン状態確認を前提としたルーティング

---

## 実行方法（ローカル）
1. 依存関係のインストール  
   ```bash
   npm install
   ```

2. 開発サーバー起動  
   ```bash
   npm run dev
   ```

3. ブラウザでアクセス  
   [http://localhost:5173](http://localhost:5173)

---

## Docker 利用方法
1. `docker-compose up` を実行  
2. ブラウザで [http://localhost:5173](http://localhost:5173) を開く  

---

## カスタマイズポイント
- **定型文追加**: `src/constants/responses.ts` に追記  
- **UI 拡張**: `src/components/ChatWindow.tsx` を編集  
- **バックエンド/外部 API のエンドポイント**: `.env` または `src/config.ts` で、無料/有料/VIP 向け URL と API Key を管理  
- **ログイン/プラン判定ロジック**: `src/hooks/useAuth.ts` などでユーザー属性を取得し、送信先を分岐  
