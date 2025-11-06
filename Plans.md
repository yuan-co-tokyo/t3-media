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

**目的:** Drizzle + tRPC + Tailwind の基本構成をローカルで動かす。

- [ ] `npx create-t3-app@latest t3-media` を実行（Prisma は「No」）
- [ ] Drizzle ORM（SQLite）導入  
  ```bash
  npm install drizzle-orm better-sqlite3 drizzle-kit


## Phase 2: AWS バックエンド接続（RDS + S3）

**目的:** SQLite → RDS(PostgreSQL) に移行し、S3 を利用したアップロード基盤を構築する。  
ここからアプリは「クラウドで永続化＋大容量ファイル管理」が可能になる。

---

### ✅ 2-1. RDS (PostgreSQL) の構築

- [ ] AWS コンソールで **Amazon RDS → PostgreSQL** を作成  
  - インスタンスクラス：`db.t3.micro`（開発用）  
  - ストレージ：20GB〜  
  - パブリックアクセス：基本は **No**  
  - VPC セキュリティグループ：アプリからの 5432 を許可

- [ ] `.env` に接続文字列を追加：
  ```env
  DATABASE_URL="postgres://user:password@hostname:5432/dbname"

## Phase 3: 認証（NextAuth + Amazon Cognito）

**目的:** Cognito によるサインアップ/サインインを導入し、  
セッション（認証済みユーザー）と DB（assets.ownerId）を安全に紐付ける。

---

### ✅ 3-1. 依存パッケージ

```bash
npm install next-auth
# （App Router 前提、Auth.js v5）

## Phase 4: ファイル共有・ダウンロード

**目的:** 認可済みユーザーのファイルを、**本人**または**共有リンク（トークン）**経由でダウンロードできるようにする。  
S3 は引き続き **非公開**。アクセス時に **presigned GET URL** を短命で発行する。

---

### ✅ 4-1. `shares` テーブル追加（Drizzle / Postgres）

共有リンク（トークン）を管理するテーブルを追加。

```ts
// src/server/schema.ts の末尾あたりに追記
import { pgTable, text, timestamp } from "drizzle-orm/pg-core";
import { sql } from "drizzle-orm";

export const shares = pgTable("shares", {
  id: text("id").primaryKey(),          // cuid
  assetId: text("asset_id").notNull(),  // FK (論理参照)
  createdBy: text("created_by").notNull(),
  token: text("token").notNull().unique(), // ランダム英数（URLセーフ）
  expiresAt: timestamp("expires_at"),      // 期限なし可
  createdAt: timestamp("created_at").default(sql`now()`).notNull(),
});


### Phase 5: デプロイ（Amplify / App Runner）

**目的:** AWS 上に本番環境を構築し、環境変数・VPC・DB マイグレーションを含む運用可能な形にする。  
選択肢は **Amplify Hosting（最短）** か **App Runner（Docker ベースで柔軟）**。

---

### ✅ 5-0. 本番環境の前提

- RDS(Postgres) は **パブリックアクセス OFF**、VPC 内。  
- S3 バケットは **Block Public Access = ON**。  
- Cognito User Pool & App Client（本番ドメインの Callback/Sign-out URL を追加）。  
- **環境変数（共通）**：


## Phase 6: 発展（Codex を活かした改良）

**目的:** 本番運用で必要なパフォーマンス・UX・セキュリティ・運用性を強化し、規模拡大にも耐える構成へ進化させる。  
各項目は **独立タスク**として順次導入可能。Codex への依頼テンプレも併記。

---

### ✅ 6-1. 配信最適化（CloudFront 署名付き URL/クッキー）

- [ ] CloudFront を導入し、オリジンを S3（OAC 有効）に設定
- [ ] 画像/動画ダウンロードは **CloudFront 署名付き URL** を発行（短寿命）
- [ ] Range リクエスト（動画シーク）と大容量配信を最適化
- Codex 依頼例：
  > 「Node.js で CloudFront 署名付き URL を生成するユーティリティ関数を書いて。キーは環境変数から読む設計で」

---

### ✅ 6-2. アップロード UX（進捗表示・ドラッグ&ドロップ・マルチパート）

- [ ] XHR / fetch で **プログレスバー**を表示（%・残り時間）
- [ ] DnD 対応、複数ファイルの並列アップロード
- [ ] 100MB+ は **S3 マルチパート**に自動切替（クライアント→Presign→並列 Part PUT→Complete）
- Codex 依頼例：
  > 「presigned URL を使ったマルチパートアップロードの React フック（進捗・キャンセル可能）を実装して」

**マルチパートの流れ（サマリ）**
1) tRPC: `createMultipartUpload` → UploadId と Part 用 presign 一括払い出し  
2) クライアント: 各 Part を PUT（並列/リトライ）→ ETag 収集  
3) tRPC: `completeMultipartUpload` に ETag 一覧渡して完了

---

### ✅ 6-3. メディア処理（画像サムネ・動画トランスコード）

- [ ] **S3 イベント**（ObjectCreated）→ **Lambda** で画像のサムネ/EXIF 抽出
- [ ] 動画は **MediaConvert** で HLS/DASH 生成、プレイヤー（video.js など）で再生
- [ ] DB に派生アセット（`variants`）テーブルを追加（解像度・ビットレート・URL）
- Codex 依頼例：
  > 「Lambda（Node.js）で S3 から画像を取得→サムネ生成→`thumbs/` に PutObject し、DB を更新するコードを生成して」

---

### ✅ 6-4. セキュリティ強化（検疫・レート制限・共有無効化）

- [ ] **ウイルススキャン**：S3 → Lambda + ClamAV（検疫バケット／タグで状態管理）
- [ ] 共有リンク API に **レート制限**（IP + user.id）＆ **短寿命トークン**（60–120s presigned）
- [ ] 共有リンクの **失効 API**（`DELETE /shares/:id`）と UI を追加
- Codex 依頼例：
  > 「tRPC の middleware でユーザー別レート制限を実装して。1分あたりの共有リンク作成を N 回に制限」

---

### ✅ 6-5. 信頼性（再試行・耐障害・キューイング）

- [ ] フロントのアップロードで **指数バックオフ**＋ **中断/再開**
- [ ] サムネ/転送などの非同期処理は **SQS** キューでリトライ管理
- [ ] 致命的失敗は **DLQ（Dead-Letter Queue）**へ
- Codex 依頼例：
  > 「SQS で非同期サムネジョブを送受信するユーティリティ（キュー送信・可視性タイムアウト・DLQ対応）を作って」

---

### ✅ 6-6. 観測性（ログ・トレース・メトリクス）

- [ ] **構造化ログ**（pino 等）、リクエスト ID 付与、ログ相関
- [ ] **OpenTelemetry** で tRPC / Next.js のトレースを CloudWatch / X-Ray に送出
- [ ] **S3 Access Logs** / CloudFront Logs 有効化、エラー率や4xx/5xxを可視化
- Codex 依頼例：
  > 「Next.js(tRPC) に OpenTelemetry を組み込む最小設定と、CloudWatch に出す exporter のコードを追加して」

---

### ✅ 6-7. コスト最適化

- [ ] S3 Intelligent-Tiering、ライフサイクルで古いオブジェクトを低コスト階層へ移動
- [ ] CloudFront キャッシュポリシー最適化（`Cache-Control` / `ETag`）
- [ ] AWS Budgets とアラート設定
- Codex 依頼例：
  > 「S3 ライフサイクルルール（30日で IA、90日で Glacier）を IaC（CDK）で書いて」

---

### ✅ 6-8. ガバナンス（権限・監査・ポリシー）

- [ ] フォルダ/スペース単位の **ロール & 権限モデル**（owner / editor / viewer）
- [ ] 重要操作（共有作成/削除、ダウンロード生成）の **監査ログ**（append-only）
- [ ] **CSP / CORS / Security Headers** の明示化（helmet-equivalent）
- Codex 依頼例：
  > 「assets と spaces のリレーションと RBAC を Drizzle スキーマで定義して、tRPC の認可チェックを追加して」

---

### ✅ 6-9. QA・安全な変更（E2E / Feature Flags / 青緑）

- [ ] **Playwright** で E2E（サインイン→アップロード→共有→閲覧）
- [ ] **Feature Flags**（ConfigCat / LaunchDarkly / 自前）で段階リリース
- [ ] **Blue/Green** / ロールバック手順の自動化（App Runner / Amplify で前バージョン復帰）
- Codex 依頼例：
  > 「Upload→Share→Download の一連の E2E を Playwright で書いて。Cognito モック/スタブ前提」

---

### ✅ 6-10. CI/CD と IaC

- [ ] GitHub Actions：
  - `lint → test → build → drizzle generate/migrate → deploy`
  - 環境毎（dev/stg/prd）にジョブ分割、DB マイグレーション手順を明記
- [ ] **IaC（CDK / SST / Terraform）** で S3, CF, RDS, Cognito, App Runner をコード化
- Codex 依頼例：
  > 「App Runner サービス + VPC Connector + S3 + RDS + Cognito を CDK(TypeScript)で最小構成として記述して」

---

### ✅ 6-11. UI/UX 強化（ダッシュボード・検索・アクセシビリティ）

- [ ] 使用量ダッシュボード（総容量・トラフィック・ダウンロード回数）  
- [ ] タグ/メタデータ検索（ファイル名・MIME・期間）  
- [ ] キーボード操作・スクリーンリーダ対応（a11y）  
- Codex 依頼例：
  > 「Recharts で容量推移とダウンロード回数を表示するダッシュボードカードを作って」

---

### ✅ 成果物（Phase 6）

- CloudFront 経由の高速・安全配信（署名付き URL/クッキー）  
- 大容量・並列・再開可能なアップロード UX  
- 自動サムネ/トランスコードと非同期パイプライン（SQS/Lambda/MediaConvert）  
- レート制限・検疫・監査ログによるセキュア運用  
- 観測性・コスト最適化・IaC・CI/CD による再現性の高い運用

---

### ▶ 次フェーズ候補

- 組織/チーム機能（課金・請求連携、Stripe / Paddle）  
- 共有スペースの権限テンプレート（リンク共有 vs メンバー制）  
- DLP/コンテンツモデレーション（Rekognition / Comprehend 連携）  
- Edge Functions（CloudFront Functions / Lambda@Edge）で軽量制御