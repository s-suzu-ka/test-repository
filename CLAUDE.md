# 電車遅延通知Bot

本リポジトリは **AI 駆動開発（AIDD）× 仕様駆動開発（SDD）× テスト駆動開発（TDD）** を前提としたプロジェクトである。

> 本ファイルは Claude Code 向けのプロジェクト固有指示である。各エージェント定義（`.claude/agents/*.md`）と整合させて運用する。

## 開発哲学（三本柱）

本プロジェクトの開発はすべて以下の三本柱に従う。順序・優先度は **SDD → TDD → AIDD** とする。

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
- 本プロジェクトのエージェント構成は **Planner → Generator → Evaluator** の 3 体制（後述「エージェント構成」参照）
- AI への指示・コミット・ドキュメントはすべて **日本語**
- AI が出力したコードは必ずテストで検証する。AI を信用してテストを省略しない

## エージェント構成

本プロジェクトでは [.claude/agents/](.claude/agents/) に定義された 3 体のサブエージェントで spec-kit フローを駆動する。役割の境界は各エージェント定義（`*.md`）の Boundaries セクションに準拠する。

| エージェント | モデル | 担当範囲 | 主要スキル |
| --- | --- | --- | --- |
| **[planner](.claude/agents/planner.md)** | opus | spec-kit フロー **1〜7**（憲法・仕様・明確化・計画・タスク・チェックリスト・**Red テスト**） | `/speckit-constitution` `/speckit-specify` `/speckit-clarify` `/speckit-plan` `/speckit-tasks` `/speckit-checklist` |
| **[generator](.claude/agents/generator.md)** | sonnet | spec-kit フロー **8**（**Green 実装**：Red テストを通す最小実装） | `/speckit-implement` |
| **[evaluator](.claude/agents/evaluator.md)** | opus | 仕様適合性 / テスト健全性 / コード品質 / セキュリティ / プロジェクトルール準拠の **5 観点評価**（read-only） | `/speckit-analyze` |

### ハンドオフの原則

```text
Planner ──(ハンドオフサマリ)──▶ Generator ──(ハンドオフサマリ)──▶ Evaluator
   ▲                                                                  │
   └────────────────── FAIL（仕様/テスト不足の差し戻し）────────────────┘
                                  │
                              CONDITIONAL_PASS（軽微指摘の差し戻し）
                                  │
                                  ▼
                              Generator
```

- **Planner → Generator**: 機能名・ブランチ・spec/plan/tasks のパス・Red テスト一覧・着手順序を明記
- **Generator → Evaluator**: 完了タスク・編集ファイル・テスト結果（全 Green の実行ログ）を明記
- **Evaluator の判定**: `PASS` / `CONDITIONAL_PASS`（Generator に差し戻し）/ `FAIL`（Planner または Generator に差し戻し）
- 役割の越境は禁止（例: Generator は spec.md を書き換えない、Evaluator は実装を編集しない）

## 開発フロー（必須）

新機能・変更は spec-kit のスラッシュコマンドを起点に以下のフローで進める。担当エージェントを各ステップに併記する。

```text
[Planner]
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

[Generator]
8. /speckit-implement または手動実装（Green）
   ↓
9. リファクタリング                ← 構造改善（テストは通ったまま）

[Evaluator]
10. /speckit-analyze + テスト/Lint/Typecheck 実行 ← 5 観点で合否判定
    ↓
11. ドキュメント同期               ← spec.md / plan.md / ADR を更新（PASS 時、Generator が実施）
    ↓
12. /speckit-git-commit & プッシュ
```

**スキップ禁止のステップ:** 2（specify）/ 7（テスト）/ 10（評価）/ 11（ドキュメント同期）。

## 言語

- ユーザーとのコミュニケーションは日本語
- コード内のコメント・ドキュメント・コミットメッセージも日本語を基本とする
- 識別子（変数名・関数名・型名）は英語

## 技術スタック

`.specify/memory/constitution.md` に準拠する。プロジェクト要件に応じて `apps/web` / `apps/api` / インフラ等の採用可否は変動する（[README.md](README.md) も参照）。

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
│   ├── agents/            # サブエージェント定義（planner / generator / evaluator）
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

> 単一アプリケーションのみで構成する場合は `apps/` を展開せずフラットに置いてよい。不要なディレクトリは削除する。

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

以下のファイルは読み込み・コミット・出力を一切禁止する。

- `.env`、`.env.*`
- `credentials.json`、`secrets.json`、`serviceAccountKey.json`
- `*.pem`、`*.key`、`*_rsa`、`*_ecdsa`、`*_ed25519`
- `.npmrc`、`token.json`、`auth.json`

## AI エージェント向け指示（必読）

このリポジトリで作業する Claude Code は以下を厳守する。

### MUST

1. 作業開始前に **`.specify/memory/constitution.md` と該当 `specs/<feature>/spec.md` を必ず読む**
2. 機能追加・変更は **specify → plan → tasks → test → implement → evaluate** の順で行う（spec-kit フロー）
3. テストを書かずに実装をマージしない
4. すべての出力（応答・コミット・コメント）は日本語
5. 役割に応じて適切なサブエージェントへ委譲する（Planner / Generator / Evaluator）。各エージェントの Boundaries（`.claude/agents/*.md`）を越境しない
6. 各エージェントは完了時に **ハンドオフサマリ** を必ず出力し、次工程の前提を明示する
7. 評価（Evaluator）で `PASS` を得てからコミットする
8. 仕様変更を伴う実装をした場合は、同じ PR / コミット内で `specs/<feature>/spec.md` を更新する

### NEVER

1. `specs/<feature>/spec.md` に存在しない機能を勝手に実装しない
2. テストを削除・スキップしてビルドを通そうとしない（`it.skip` / `--no-verify` 等）
3. 機密ファイルを読み込み・出力しない
4. 既存の挙動を破壊するリファクタを依頼なしに行わない
5. `any` 型・型アサーション（`as`）を安易に使わない
6. AI 生成コードをテストせずに「完了」と報告しない
7. `.specify/` および `.claude/skills/speckit-*` を手動編集しない（spec-kit 管理領域）

## 初期セットアップチェックリスト

新規開発を開始する場合、以下を必ず実施する。

- [ ] `/speckit-constitution` でプロジェクト憲法を初期化（`.specify/memory/constitution.md`）
- [ ] `/speckit-specify` で初期機能の仕様を作成（`specs/<feature>/spec.md`）
- [ ] 不要なディレクトリ（モノレポ構成等）を削除
- [ ] CI（`.github/workflows/`）が動くことを確認

## 参照

- エージェント定義: [.claude/agents/planner.md](.claude/agents/planner.md) / [generator.md](.claude/agents/generator.md) / [evaluator.md](.claude/agents/evaluator.md)
- spec-kit スキル: [.claude/skills/](.claude/skills/)（`speckit-*`）
- プロジェクト憲法: [.specify/memory/constitution.md](.specify/memory/constitution.md)
- spec-kit ドキュメント: <https://github.github.io/spec-kit/>
- spec-kit リポジトリ: <https://github.com/github/spec-kit>

<!-- SPECKIT START -->

For additional context about technologies to be used, project structure,
shell commands, and other important information, read the current plan

<!-- SPECKIT END -->
