README（日本語版）
スナックミサキ
プロジェクト概要

AWS Lambda 上で「スナックのママ」キャラクターがユーザーの相談に応じるアプリです。
コスト・拡張性・実用性を考慮し、以下の 3段階構成 で開発を進めています。

開発ロードマップ
第1段階（フロントのみ）

定型文をフロント側で直接返答

即レス & サーバーコストゼロ

第2段階（定型文 + 小型モデル）

定型文はフロントで処理

未対応の入力は Lambda (Python) + 軽量化LLM（llama.cpp / GPT4All）で返答

第3段階（定型文 + 小型モデル + 外部LLM）

基本は第2段階と同じ

高度な入力は OpenAI API / AWS Bedrock / HuggingFace Hub へフォールバック

技術スタック

フロントエンド: React / TypeScript（Codex連携）

バックエンド: AWS Lambda (Python 3.11, Docker)

LLM: llama.cpp, GPT4All, OpenAI API, AWS Bedrock, HuggingFace Hub

アーキテクチャ図
flowchart TD

subgraph Frontend [Frontend (React / TypeScript)]
    F1[ユーザー入力]
    F2[定型文マッチング]
end

subgraph Backend [Backend (AWS Lambda / Python)]
    B1[小型LLM (llama.cpp / GPT4All)]
    B2[外部LLM API (OpenAI / Bedrock / HuggingFace)]
end

F1 --> F2
F2 -->|定型文ヒット| F2
F2 -->|未対応入力| B1
B1 -->|処理困難 or 高度要求| B2
B1 -->|応答| F1
B2 -->|応答| F1

実行方法（ローカル）
# Docker build
docker build -t snack-misaki .
docker run -p 9000:8080 snack-misaki

# Lambda エミュレータ経由でイベント送信
curl -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" \
  -d '{"input":"こんばんは"}'

応用可能なユースケース

キャラクターボット

FAQ自動応答

社内ヘルプデスクPoC

軽量な会話体験アプリ

README（English Version）
Snack Misaki
Project Overview

A conversational bot with a “snack bar mama” persona, implemented on AWS Lambda.
Designed to balance cost, scalability, and practicality, built in three progressive stages.

Roadmap
Stage 1 (Frontend only)

Return predefined responses directly from frontend

Zero cost, instant response

Stage 2 (Predefined + Small Model)

Frontend handles predefined responses

Unmatched inputs forwarded to Lambda (Python) + lightweight LLM (llama.cpp / GPT4All)

Stage 3 (Predefined + Small Model + External LLM)

Same as Stage 2 by default

Complex/free-text inputs fallback to OpenAI API / AWS Bedrock / HuggingFace Hub

Tech Stack

Frontend: React / TypeScript (Codex integration)

Backend: AWS Lambda (Python 3.11, Docker)

LLM: llama.cpp, GPT4All, OpenAI API, AWS Bedrock, HuggingFace Hub

Architecture Diagram
flowchart TD

subgraph Frontend [Frontend (React / TypeScript)]
    F1[User Input]
    F2[Predefined Response Matching]
end

subgraph Backend [Backend (AWS Lambda / Python)]
    B1[Lightweight LLM (llama.cpp / GPT4All)]
    B2[External LLM API (OpenAI / Bedrock / HuggingFace)]
end

F1 --> F2
F2 -->|Matched| F2
F2 -->|Unmatched| B1
B1 -->|Escalation| B2
B1 -->|Response| F1
B2 -->|Response| F1

Run Locally
# Docker build
docker build -t snack-misaki .
docker run -p 9000:8080 snack-misaki

# Send event via Lambda Emulator
curl -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" \
  -d '{"input":"Hello"}'

Potential Use Cases

Character bots with persona

Automated FAQ

Internal helpdesk PoC

Lightweight conversational apps
