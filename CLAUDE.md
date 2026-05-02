# 電車遅延通知Bot

本リポジトリは **AI 駆動開発（AIDD）× 仕様駆動開発（SDD）× テスト駆動開発（TDD）** を前提とした新規プロジェクトの **スタータテンプレート** である。

> 上位ルール: 組織憲法（`/CLAUDE.md`）と全エージェント共通ルール（`/AGENTS.md`）が常に優先される。本ファイルはその下位ルールとして、テンプレート由来プロジェクトの開発作法を定義する。

## 開発哲学（三本柱）

本テンプレートの開発はすべて以下の三本柱に従う。順序・優先度は **SDD → TDD → AIDD** とする。

### 1. 仕様駆動開発（SDD: Spec-Driven Development）

- **仕様が先、実装は後**。仕様書に書かれていない機能は実装しない
- 仕様の管理には [GitHub Spec Kit](https://github.com/github/spec-kit) を採用する
  - プロジェクト憲法: `.specify/memory/constitution.md`（`/speckit-constitution` で初期化・更新）
  - 機能仕様: `specs/<feature>/spec.md`（`/speckit-specify` で生成）
  - 実装計画: `specs/<feature>/plan.md`（`/speckit-plan` で生成）
  - タスク: `specs/<feature>/tasks.md`（`/speckit-tasks` で生成）
- 機能追加・変更は必ず仕様（`spec.md`）の更新から始める
- 仕様は **「何を / なぜ / どうあるべきか」** を記述し、実装詳細（HOW）は `plan.md` または ADR に分離する
- 受け入れ条件（Acceptance Criteria）を必ず明記する

### 2. テスト駆動開発（TDD: Test-Driven Development）

- **Red → Green → Refactor** サイクルを厳守する
  1. **Red**: 仕様の受け入れ条件を満たすテストを書く（落ちる状態）
  2. **Green**: テストが通る最小実装を書く
  3. **Refactor**: テストが通ったまま構造を整える
- テストのないコードはマージしない
- テスト種別:
  - **Unit**: Vitest
  - **Integration**: Vitest + 実 DB / 実依存
  - **E2E**: Playwright
  - **API モック**: MSW

### 3. AI 駆動開発（AIDD: AI-Driven Development）

- 各工程は AI エージェント（Claude Code）が駆動する
- エージェントは **組織憲法の指揮系統** に従う（CEO → 秘書 Iris → COO Hermes → 担当エージェント）
- AI への指示・コミット・ドキュメントはすべて **日本語**
- AI が出力したコードは必ずテストで検証する。AI を信用してテストを省略しない

## 開発フロー（必須）

新機能・変更は spec-kit のスラッシュコマンドを起点に以下のフローで進める。

```text
1. /speckit-constitution        ← プロジェクト憲法を初期化／更新（初回のみ）
   ↓
2. /speckit-specify              ← 機能仕様（spec.md）を生成・更新
   ↓
3. /speckit-clarify  (任意)      ← 曖昧点を構造化された質問で潰す
   ↓
4. /speckit-plan                 ← 実装計画（plan.md）を作成
   ↓
5. /speckit-tasks                ← タスク（tasks.md）を生成
   ↓
6. /speckit-checklist (任意)     ← 仕様の完全性を検証
   ↓
7. テスト作成（Red）              ← 受け入れ条件をテストコードに翻訳
   ↓
8. /speckit-implement または手動実装（Green）
   ↓
9. リファクタリング                ← 構造改善（テストは通ったまま）
   ↓
10. ドキュメント同期               ← spec.md / plan.md / ADR を更新
   ↓
11. /speckit-git-commit & プッシュ
```

**スキップ禁止のステップ:** 2（specify）/ 7（テスト）/ 10（ドキュメント同期）。

## 言語

- ユーザーとのコミュニケーションは日本語
- コード内のコメント・ドキュメント・コミットメッセージも日本語を基本とする
- 識別子（変数名・関数名・型名）は英語

## 技術スタック

組織憲法に準拠する。

| 種別             | 採用技術   |
| ---------------- | ---------- |
| 言語             | TypeScript |
| フロントエンド   | Next.js    |
| バックエンド     | NestJS     |
| ORM              | Prisma     |
| バリデーション   | Zod        |
| 単体・結合テスト | Vitest     |
| E2E テスト       | Playwright |
| API モック       | MSW        |
| インフラ         | AWS        |

## ディレクトリ構成（推奨）

```text
.
├── CLAUDE.md              # 本ファイル（プロジェクト固有の AI 指示）
├── README.md              # 人間向けプロジェクト説明
├── .specify/              # spec-kit 管理領域（templates / scripts / workflows）
│   └── memory/
│       └── constitution.md  # プロジェクト憲法
├── .claude/
│   └── skills/            # spec-kit スラッシュコマンド（speckit-*）
├── specs/                 # spec-kit が機能ごとに生成（spec.md / plan.md / tasks.md）
├── docs/
│   ├── adr/               # アーキテクチャ決定記録（ADR）
│   └── runbook/           # 運用手順
├── apps/
│   ├── web/               # Next.js（フロントエンド）
│   └── api/               # NestJS（バックエンド）
├── packages/
│   ├── shared/            # 共通型・ユーティリティ
│   └── schema/            # Zod スキーマ（仕様の単一の源）
├── prisma/
│   ├── schema.prisma
│   └── migrations/
├── tests/
│   ├── unit/              # Vitest 単体テスト
│   ├── integration/       # Vitest 結合テスト
│   ├── e2e/               # Playwright E2E
│   └── fixtures/          # テスト用データ
├── mocks/                 # MSW ハンドラ
└── .github/
    └── workflows/         # CI（lint / test / build）
```

> 単一プリケーションのみで構成する場合は `apps/` を展開せずフラットに置いてよい。テンプレート利用時に不要なディレクトリは削除する。

## ドキュメント運用

| ドキュメント                      | 役割                         | 更新タイミング              | 更新手段                |
| --------------------------------- | ---------------------------- | --------------------------- | ----------------------- |
| `.specify/memory/constitution.md` | プロジェクト憲法（不変原則） | 重要な原則の変更時          | `/speckit-constitution` |
| `specs/<feature>/spec.md`         | 機能仕様（What / Why）       | 機能追加・変更前に必ず      | `/speckit-specify`      |
| `specs/<feature>/plan.md`         | 実装計画（HOW）              | 仕様確定後                  | `/speckit-plan`         |
| `specs/<feature>/tasks.md`        | 機能タスク分解               | 計画確定後                  | `/speckit-tasks`        |
| `docs/adr/`                       | アーキテクチャ決定記録       | 不可逆 / 影響大の技術選定時 | 手動                    |
| `docs/runbook/`                   | 運用手順                     | 運用に変更があったとき      | 手動                    |
| `README.md`                       | 人間向けの概要・起動方法     | 起動方法が変わったとき      | 手動                    |

## コーディング規約

- 変更は最小限にとどめ、依頼範囲を超えるリファクタを勝手に行わない
- ハイポセティカルな将来要件のための抽象化はしない（YAGNI）
- 不要なエラーハンドリング・フォールバック・互換シムを足さない
- コメントはデフォルトで書かない。書く場合は **WHY** のみ
- バリデーションは境界（外部入力・外部 API）でのみ行う。内部呼び出しは型と契約を信頼する
- 型は `any` を避け、Zod スキーマから推論することを優先する

## セキュリティ

組織憲法のセキュリティ規定を継承する。以下のファイルは読み込み・コミット・出力を一切禁止する。

- `.env`、`.env.*`
- `credentials.json`、`secrets.json`、`serviceAccountKey.json`
- `*.pem`、`*.key`、`*_rsa`、`*_ecdsa`、`*_ed25519`
- `.npmrc`、`token.json`、`auth.json`

## AI エージェント向け指示（必読）

このリポジトリで作業する Claude Code は以下を厳守する。

### MUST

1. 作業開始前に **`.specify/memory/constitution.md` と該当 `specs/<feature>/spec.md` を必ず読む**
2. 機能追加・変更は **specify → plan → tasks → test → implement** の順で行う（spec-kit フロー）
3. テストを書かずに実装をマージしない
4. すべての出力（応答・コミット・コメント）は日本語
5. 組織憲法（`/CLAUDE.md`）の指揮系統に従う。担当エージェントへの作業振り分けは必ず **COO（Hermes）経由**
6. 作業完了ごとにコミット & リモートプッシュする（組織共通ルール）
7. 仕様変更を伴う実装をした場合は、同じ PR / コミット内で `specs/<feature>/spec.md` を更新する

### NEVER

1. `specs/<feature>/spec.md` に存在しない機能を勝手に実装しない
2. テストを削除・スキップしてビルドを通そうとしない（`it.skip` / `--no-verify` 等）
3. 機密ファイルを読み込み・出力しない
4. 既存の挙動を破壊するリファクタを依頼なしに行わない
5. `any` 型・型アサーション（`as`）を安易に使わない
6. AI 生成コードをテストせずに「完了」と報告しない
7. `.specify/` および `.claude/skills/speckit-*` を手動編集しない（spec-kit 管理領域）

## テンプレート利用時のチェックリスト

新規プロジェクトに本テンプレートを複製した場合、以下を必ず実施する。

- [ ] `CLAUDE.md` 冒頭の見出しをプロジェクト名に書き換え
- [ ] `README.md` をプロジェクト概要に書き換え
- [ ] `package.json` の `name` を変更
- [ ] `/speckit-constitution` でプロジェクト憲法を初期化（`.specify/memory/constitution.md`）
- [ ] `/speckit-specify` で初期機能の仕様を作成（`specs/<feature>/spec.md`）
- [ ] 不要なディレクトリ（モノレポ構成等）を削除
- [ ] CI（`.github/workflows/`）が動くことを確認
- [ ] 初回コミット & varet-corp Organization へ Private リポジトリとして push

## 参照

- 組織憲法: `/CLAUDE.md`
- 共通エージェントルール: `/AGENTS.md`
- プロジェクト初期化スキル: `/.claude/agents/coo/skills/project-bootstrap/`
- spec-kit ドキュメント: <https://github.github.io/spec-kit/>
- spec-kit リポジトリ: <https://github.com/github/spec-kit>

<!-- SPECKIT START -->

For additional context about technologies to be used, project structure,
shell commands, and other important information, read the current plan

<!-- SPECKIT END -->
