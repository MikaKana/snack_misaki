# Snack Misaki — Frontend

## 概要
フロントエンドは **Vite + React + TypeScript** をベースに実装されています。  
Stage 1 では定型文レスポンスのみを提供し、PoC としての最小限の機能を担います。  
Stage 2 以降も UI は共通で、バックエンド連携による応答拡張に対応します。

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

3. **バックエンド連携**  
   - Stage 2 以降では未対応入力をバックエンド API へ送信  
   - 小型 LLM または外部 LLM API からの応答を表示  

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
- **バックエンド API のエンドポイント**: `.env` または `src/config.ts` で管理  

