# Snack Misaki — Project Overview

このリポジトリは **Snack Misaki プロジェクトの統括ドキュメント専用リポジトリ** です。  
コードは含まれず、全体の概要・アーキテクチャ・関連リポジトリへのリンクをまとめています。

---

## プロジェクト概要
Snack Misaki は「スナックのママ」キャラクターを持つ対話型アプリケーション

- 目的：軽量かつ実用的な会話体験を提供する

- 構成：フロントエンド（React/TypeScript）、バックエンド（AWS Lambda / Python）、複数の LLM（小型 LLM と外部 API）のハイブリッド構成

- 特徴：

  - 定型応答と LLM を組み合わせたコスト最適化

  - フロントとバックを分離したスケーラブルな開発体制

  - PoC（FAQ/社内ヘルプデスク/キャラクターボット）から実運用まで拡張可能

## リポジトリ一覧

- [snack-misaki-frontend](https://github.com/MikaKana/snack-misaki-frontend)  
  Vite + React + TypeScript によるフロントエンド。定型文レスポンスの UI を提供。

- [snack-misaki-backend](https://github.com/MikaKana/snack-misaki-backend)  
  AWS Lambda (Python 3.11, Docker) によるバックエンド。小型 LLM および外部 LLM API 連携を担当。

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

1. **フロントエンドのみ**  
   React で定型文レスポンスを提供

2. **バックエンド連携**  
   小型 LLM (llama.cpp / GPT4All) を Lambda 内で実行

3. **外部 LLM API 連携**  
   OpenAI API / AWS Bedrock / HuggingFace Hub を利用した高度応答
