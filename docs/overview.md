# Snack Misaki — Overview

## プロジェクト概要
**Snack Misaki** は「スナックのママ」キャラクターを持つ対話型アプリケーションです。  
ユーザーが自然に話しかけられるような親しみやすさと、実用的な回答の両立を目指しています。  

- **目的**  
  軽量かつ実用的な会話体験を提供し、コストとスケーラビリティのバランスをとる。  

- **構成**
  - フロントエンド: React / TypeScript（Vite ベース）
  - バックエンド: RunPod Serverless (GPU, Python 3.11, Docker)
  - LLM: 小型 LLM（Phi-3 Mini on RunPod）を基盤に、条件クリア時は Mistral 7B に自動切替。必要に応じて外部 API (OpenAI / AWS Bedrock / HuggingFace Hub) と連携。

- **特徴**  
  - 定型応答と LLM を組み合わせた **ハイブリッド応答**  
  - フロントとバックを分離した **スケーラブルな開発体制**  
  - PoC（FAQ / 社内ヘルプデスク / キャラクターボット）から実運用まで拡張可能  

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

1. **フロントエンドのみ**  
   React による定型文レスポンス UI の提供。最小限の PoC。  

2. **バックエンド連携**
   RunPod Serverless (GPU) 上で Phi-3 Mini を常駐させ、未対応入力に応答。課金や利用条件を満たした場合は Mistral 7B に切り替えて返答するロジックを追加。

3. **外部 LLM API 連携**  
   OpenAI API / AWS Bedrock / HuggingFace Hub を利用し、高度な応答を実現。  

---

## 関連リポジトリ
- [snack-misaki-frontend](https://github.com/MikaKana/snack-misaki-frontend)  
  Vite + React + TypeScript によるフロントエンド  

- [snack-misaki-backend](https://github.com/MikaKana/snack-misaki-backend)
  RunPod Serverless (GPU, Python, Docker) によるバックエンド

---

この `overview.md` は「全体像を掴む」ためのドキュメントです。  
細かい設計や利用方法は [`architecture.md`](./architecture.md), [`frontend.md`](./frontend.md), [`backend.md`](./backend.md) に続きます。  
