# 🧭 Plans.md — AWS × T3 Stack × Codex プロジェクト計画書

## 🎯 プロジェクト概要

AWS 上で動作する **T3 Stack (Next.js + tRPC + Tailwind + Drizzle)** ベースの  
**画像・動画共有ストレージアプリ**を開発する。

ユーザーがサインアップ／サインインして、ファイルを S3 にアップロード・ダウンロードし、  
他ユーザーと共有できる安全なストレージサービスを構築する。

---

## 🧱 使用技術スタック

| カテゴリ | 使用技術 | 補足 |
|-----------|-----------|------|
| フロントエンド | Next.js (App Router) + tRPC + Tailwind | T3 Stack 基盤 |
| バックエンド | Node.js (SSR) + AWS SDK v3 | tRPC サーバ側で S3/DB 操作 |
| ORM / DB | Drizzle ORM + PostgreSQL (Amazon RDS) | Prisma 不使用 |
| 認証 | Auth.js (NextAuth) + Amazon Cognito | OIDC 認証 |
| ストレージ | Amazon S3 | presigned URL でアップロード／ダウンロード |
| 配信 | Amazon CloudFront | 署名付き URL・Cookie 対応 |
| インフラ | AWS Amplify / App Runner | 初期は Amplify を想定 |
| AI 支援 | ChatGPT Codex / GitHub Copilot | コード補完・自動生成支援 |

---

## 🪜 開発フェーズ

### Phase 1: ローカル MVP 構築（Codex 開発環境確立）

**目的:**
Drizzle + tRPC + Tailwind の基本構成をローカルで動かす。

**環境:**
DBはローカルのPostgreSQL、ストレージはminio、ローカルはログイン認証を行わない
利用はスマートフォンのブラウザを想定した画面とする

- [ ] PostgreSQLやminioなどの環境を構築し、構築手順をREADME.mdに追加
- [ ] ログイン画面の開発（認証機能は無しでログインボタンのみとし、まずはダミーのユーザーとして処理されるようにする）
- [ ] トップ画面を作成し、アップロードのためのボタンを設置
- [ ] アップロードボタンを押下すると、端末にある写真・動画を格納しているフォルダを表示し、それぞれのサムネイルを表示する
- [ ] サムネイルをタップすることで選択ができる
- [ ] アップロードボタンを押下するとS3（ローカルだとminio）にアップロードが行われる。アップロードは各ユーザーごとのフォルダに実行される。ひとまずダミーのユーザーで処理を行う。


