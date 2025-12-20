# Snack Misaki — Overview

## プロジェクト概要
**Snack Misaki** は「スナックのママ」キャラクターを持つ対話型アプリケーションです。  
ユーザーが自然に話しかけられるような親しみやすさと、実用的な回答の両立を目指しています。  

- **目的**  
  軽量かつ実用的な会話体験を提供し、コストとスケーラビリティのバランスをとる。  

- **構成**
  - フロントエンド: React / TypeScript（Vite ベース）
  - LLM バックエンド: RunPod Serverless (GPU, Python 3.11, Docker)
  - 認証/決済バックエンド: AWS Lambda + DynamoDB（Cognito 連携 / Stripe 決済）
  - LLM: ユーザー属性に応じた 3 ルート
    - 無料ユーザー: RunPod Serverless (GPU) 上の **Phi-3 Mini** 専用エンドポイント（再学習予定）
    - 有料ユーザー: Cognito ログイン後、Stripe 決済ステータスを認証/決済 API が確認し、RunPod Serverless (GPU) 上の **Mistral 7B** 専用エンドポイントへルーティング（再学習予定）
    - VIP ユーザー: Stripe の上位プランを認証/決済 API が判定し、**Claude / GPT API** など外部 LLM へルーティング

- **特徴**  
  - 定型応答に加え、ユーザー属性ごとにエンドポイントを変える **多経路ハイブリッド応答**  
  - フロントとバックを分離し、RunPod 2 系統 + 外部 LLM へのルーティングをフロントで制御  
  - Cognito + Stripe + Lambda + DynamoDB による認証/決済の薄いバックエンドを追加し、プラン判定と LLM ルーティングを連携  
  - PoC（FAQ / 社内ヘルプデスク / キャラクターボット）から有料/ VIP 提供まで拡張可能  

---

## 想定ユースケース
- **キャラクターボット**  
  エンタメ寄りの体験。親しみやすいキャラクターを通じてユーザーと対話。

- **FAQ 自動応答**  
  シンプルな問い合わせを即時回答。定型文ベースでコストを抑制。

- **社内ヘルプデスク PoC**  
  LLM を活用した問合せ対応。外部 API を利用して精度を強化。

- **軽量な会話体験アプリ**  
  小型 LLM を用いたローカル処理で、低コストかつレスポンス良好。

---

## 開発ステージ
Snack Misaki は **3 段階の進化型プロジェクト** として設計されています。  

1. **フロントエンド → Phi-3 Mini**  
   未ログインの無料ユーザーを RunPod Serverless (GPU) 上の Phi-3 Mini エンドポイントへ送る。

2. **フロントエンド（ログイン） → 認証/決済 API → Mistral 7B**  
   Cognito でログインし、Stripe の決済結果を認証/決済 API（Lambda + DynamoDB）で確認。  
   有料ユーザーを RunPod Serverless (GPU) 上の Mistral 7B エンドポイントへルーティング。

3. **フロントエンド（ログイン） → 認証/決済 API → 外部 LLM API**  
   Stripe の上位プランを持つ VIP ユーザーを認証/決済 API が判定し、Claude / GPT など外部 API にルーティングして高度な応答を提供。  

---

## 関連リポジトリ
- [snack-misaki-frontend](https://github.com/MikaKana/snack-misaki-frontend)  
  Vite + React + TypeScript によるフロントエンド  

- [snack-misaki-backend](https://github.com/MikaKana/snack-misaki-backend)
  RunPod Serverless (GPU, Python, Docker) による LLM バックエンド

- （新規追加予定）認証/決済バックエンド  
  AWS Lambda + DynamoDB による認証/決済 API（Cognito/Stripe 連携）

---

この `overview.md` は「全体像を掴む」ためのドキュメントです。  
細かい設計や利用方法は [`architecture.md`](./architecture.md), [`frontend.md`](./frontend.md), [`backend.md`](./backend.md) に続きます。  
