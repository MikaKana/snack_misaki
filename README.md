# Snack Misaki — Project Overview

このリポジトリは **Snack Misaki プロジェクトの統括ドキュメント専用リポジトリ** です。  
コードは含まれず、全体の概要・アーキテクチャ・関連リポジトリへのリンクをまとめています。

---

## プロジェクト概要
Snack Misaki は「スナックのママ」キャラクターを持つ対話型アプリケーション

- 目的：軽量かつ実用的な会話体験を提供する

- 構成：フロントエンド（React/TypeScript）、バックエンド（RunPod Serverless (GPU) / Python）

- LLM 構成：
  - **無料ユーザー**: RunPod Serverless (GPU) 上の Phi-3 Mini 専用エンドポイント
  - **有料ユーザー**: RunPod Serverless (GPU) 上の Mistral 7B 専用エンドポイント
  - **VIP ユーザー**: Claude / GPT など外部 LLM API へのルーティング
  - Phi-3 Mini / Mistral 7B は再学習済みモデルへの置換を前提

- 特徴：

  - 定型応答と複数エンドポイントを組み合わせたコスト最適化

  - フロントとバックを分離し、ユーザー属性に応じてモデルを選択するスケーラブル構成

  - PoC（FAQ/社内ヘルプデスク/キャラクターボット）から実運用まで拡張可能

## リポジトリ一覧

- [snack-misaki-frontend](https://github.com/MikaKana/snack-misaki-frontend)  
  Vite + React + TypeScript によるフロントエンド。定型文レスポンスの UI を提供。

- [snack-misaki-backend](https://github.com/MikaKana/snack-misaki-backend)
  RunPod Serverless (GPU, Python 3.11, Docker) によるバックエンド。小型 LLM および外部 LLM API 連携を担当。

---

## ドキュメント

- [docs/overview.md](docs/overview.md)  
  プロジェクト全体の概要

- [docs/architecture.md](docs/architecture.md)  
  プロジェクト全体のアーキテクチャ図と 3 段階の開発ステージ

- [docs/frontend.md](docs/frontend.md)  
  フロントエンドの設計・補足説明

- [docs/backend.md](docs/backend.md)  
  バックエンドの設計・補足説明

---

## 開発ステージ

1. **フロントエンド → Phi-3 Mini**  
   未ログインの無料ユーザーを RunPod Serverless (GPU) 上の Phi-3 Mini エンドポイントへルーティング

2. **フロントエンド（ログイン） → Mistral 7B**  
   ログイン済みの有料ユーザーを RunPod Serverless (GPU) 上の Mistral 7B エンドポイントへルーティング

3. **フロントエンド（ログイン） → 外部 LLM API**  
   VIP ユーザーを Claude / GPT など外部 API にルーティングし高度応答を提供
