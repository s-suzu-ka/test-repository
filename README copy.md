# 開発テンプレートキット

**AI 駆動開発（AIDD）× 仕様駆動開発（SDD）× テスト駆動開発（TDD）** を前提とした新規プロジェクトのスタータテンプレート。

## このテンプレートが目指すもの

- 仕様 → テスト → 実装の順を **[GitHub Spec Kit](https://github.com/github/spec-kit)** で強制し、AI エージェントが暴走しない開発フローを定着させる
- フロント / バックエンド / DB / E2E まで、varet 標準スタックを最短で立ち上げられる土台を提供する
- AI（Claude Code）と人間が同じドキュメントを起点に協働できるようにする

## 開発哲学（三本柱）

| 柱 | 略称 | 内容 |
| -- | ---- | ---- |
| 仕様駆動 | SDD | `specs/<feature>/spec.md` に書かれていない機能は実装しない |
| テスト駆動 | TDD | Red → Green → Refactor。テストのないコードはマージしない |
| AI 駆動 | AIDD | エージェントが各工程を駆動。出力は必ずテストで検証する |

詳細は [`CLAUDE.md`](CLAUDE.md) を参照。

## 技術スタック

| 種別 | 採用技術 |
| ---- | -------- |
| 言語 | TypeScript |
| フロントエンド | Next.js |
| バックエンド | NestJS |
| ORM | Prisma |
| バリデーション | Zod |
| 単体・結合テスト | Vitest |
| E2E テスト | Playwright |
| API モック | MSW |
| インフラ | AWS |
| パッケージマネージャ | pnpm |

## セットアップ

```bash
# 依存関係インストール
pnpm install

# Playwright のブラウザバイナリ取得（E2E を使う場合のみ）
pnpm exec playwright install
```

### spec-kit（Spec-Driven Development）

本テンプレートには [spec-kit](https://github.com/github/spec-kit) が同梱済み（`.specify/` および `.claude/skills/speckit-*`）。
Claude Code を本ディレクトリで起動すると、以下のスラッシュコマンドが利用可能になる。

| コマンド | 用途 |
| -------- | ---- |
| `/speckit-constitution` | プロジェクト憲法（不変原則）を初期化・更新 |
| `/speckit-specify` | 機能仕様（`specs/<feature>/spec.md`）を生成 |
| `/speckit-clarify` | 仕様の曖昧点を構造化された質問で潰す（任意） |
| `/speckit-plan` | 実装計画（`plan.md`）を作成 |
| `/speckit-tasks` | タスクリスト（`tasks.md`）を生成 |
| `/speckit-checklist` | 仕様の完全性を検証（任意） |
| `/speckit-analyze` | 仕様・計画・タスクの一貫性レポート（任意） |
| `/speckit-implement` | 計画に従って実装を実行 |
| `/speckit-git-*` | git 操作（feature ブランチ作成・コミット等） |

CLI を別途使う場合：

```bash
# 永続インストール（uv 必須）
brew install uv
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@v0.8.2
specify version
```

## スクリプト

| コマンド | 用途 |
| -------- | ---- |
| `pnpm typecheck` | TypeScript の型チェック |
| `pnpm lint` | ESLint |
| `pnpm format` | Prettier で整形 |
| `pnpm format:check` | Prettier 整形差分チェック |
| `pnpm test` | Vitest（単体・結合）を 1 回実行 |
| `pnpm test:watch` | Vitest をウォッチモードで実行 |
| `pnpm test:coverage` | カバレッジ計測付きで Vitest を実行 |
| `pnpm test:e2e` | Playwright で E2E テストを実行 |
| `pnpm test:e2e:ui` | Playwright UI モード |

## ディレクトリ構成（初期状態：フラット）

```text
.
├── CLAUDE.md              # AI エージェント向け指示・開発哲学
├── README.md              # 本ファイル（人間向け）
├── package.json
├── tsconfig.json
├── vitest.config.ts
├── playwright.config.ts
├── .specify/              # spec-kit 管理領域（templates / scripts / workflows）
│   └── memory/
│       └── constitution.md  # プロジェクト憲法
├── .claude/
│   └── skills/            # spec-kit スラッシュコマンド（speckit-*）
├── specs/                 # spec-kit が機能ごとに生成（spec.md / plan.md / tasks.md）
├── src/                   # 実装コード
├── tests/
│   ├── unit/              # Vitest 単体テスト
│   ├── integration/       # Vitest 結合テスト
│   ├── e2e/               # Playwright E2E
│   └── fixtures/          # テスト用データ
├── mocks/                 # MSW ハンドラ
└── docs/
    ├── adr/               # アーキテクチャ決定記録
    └── runbook/           # 運用手順
```

> 規模が大きくなり Next.js / NestJS を併存させる必要が出たら、`apps/web`・`apps/api`・`packages/*` 構成（pnpm workspaces）へ移行する。

## 開発フロー（spec-kit ベース）

```text
1.  /speckit-constitution        ← プロジェクト憲法（初回のみ）
2.  /speckit-specify             ← 機能仕様を生成・更新
3.  /speckit-clarify  (任意)     ← 仕様の曖昧点を解消
4.  /speckit-plan                ← 実装計画
5.  /speckit-tasks               ← タスク分解
6.  /speckit-checklist (任意)    ← 仕様の完全性を検証
7.  テスト作成（Red）             ← 受け入れ条件をテストに翻訳
8.  /speckit-implement または手動実装（Green）
9.  リファクタリング              ← テストは通ったまま構造改善
10. ドキュメント同期               ← spec.md / plan.md / ADR を更新
11. /speckit-git-commit & プッシュ
```

スキップ禁止: **2（specify）/ 7（テスト）/ 10（ドキュメント同期）**。

## このテンプレートから新規プロジェクトを作る

1. `projects/template/` を別名でコピー、もしくは COO（Hermes）の `project-bootstrap` スキルを起動
2. `CLAUDE.md` 冒頭の見出し・`README.md` の名称をプロジェクト名に書き換え
3. `package.json` の `name` を変更
4. Claude Code を起動し `/speckit-constitution` でプロジェクト憲法を初期化
5. `/speckit-specify` で初期機能の仕様を作成
6. 不要なディレクトリ・スクリプトを削除
7. 初回コミット & varet-corp Organization へ Private リポジトリとして push

## 関連ドキュメント

- [`CLAUDE.md`](CLAUDE.md) — AI エージェント向け指示・規約・哲学（必読）
- [`.specify/memory/constitution.md`](.specify/memory/constitution.md) — プロジェクト憲法（spec-kit）
- 上位ルール: ルートの [`/CLAUDE.md`](../../CLAUDE.md)（組織憲法）, [`/AGENTS.md`](../../AGENTS.md)
