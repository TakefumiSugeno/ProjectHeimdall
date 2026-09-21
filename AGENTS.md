# AGENTS

## プロジェクト基本方針（OpenSpec ネイティブ）

本プロジェクトは立ち上げ初期から **OpenSpec** による仕様駆動開発 (Spec-Driven Development) を完全採用しています。

- **Single Source of Truth**: システム仕様の唯一の真実（正本）は `openspec/specs/` です。OpenSpec 管理外に個別の仕様書（`doc/spec.md` 等）を作成・二重管理しません。
- **仕様駆動ワークフロー**: すべての機能追加・仕様変更・大規模リファクタリングは、OpenSpec の 3 フェーズ（Propose → Apply → Archive）に則って進めます。
- **サブエージェントレビュー必須**: 各フェーズで専門の役割（User / SE / PG / QA Agent）によるレビューを実施し、品質と整合性を担保します。

---

## 構成

- `src/`
  - ソースコードのルートディレクトリ
- `test/`
  - テストコードのルートディレクトリ
- `test-results/`
  - **テスト実行エビデンスおよびカバレッジレポートの出力ディレクトリ**（※Git管理対象外）
  - `test-results/evidence/` - テスト実行ログ、実行サマリ、スクリーンショット、レポート等
  - `test-results/coverage/` (または `coverage/`) - カバレッジレポート（HTML、lcov、cobertura等）
- `doc/`
  - 調査メモ・外部リファレンス・画像・補足資料等の配置ディレクトリ（※システム仕様・タスクはすべて `openspec/` で管理）
- `openspec/`
  - **OpenSpecフレームワークのルートディレクトリ**
- `openspec/config.yaml`
  - OpenSpec設定（スキーマ: `spec-driven`、運用ルール・レビュー定義）
- `openspec/specs/`
  - **メイン仕様書・デルタ仕様書の配置ディレクトリ**（システムの恒久的な仕様書）
- `openspec/changes/`
  - **変更管理（提案・タスク・設計・レビュー記録）の配置ディレクトリ**
- `openspec/changes/archive/`
  - 完了した変更のアーカイブ
- `openspec/templates/`
  - **OpenSpecレビュー・チェックリスト用テンプレート**

## 利用可能なMCPサーバ

## 利用可能なSkills

- `openspec-propose` - 新しい変更の提案・計画アーティファクト生成
- `openspec-apply-change` - 変更の実装・タスク実行
- `openspec-archive-change` - 完了した変更のアーカイブ
- `openspec-sync-specs` - デルタ仕様からメイン仕様への同期
- `openspec-update-change` - 既存変更の計画見直し
- `openspec-explore` - アイデア探索・要件明確化

---

## 開発運用ルール

### テストエビデンス・カバレッジレポート構成管理

本プロジェクトは **TDD (Test-Driven Development)** を前提とし、テストの実行結果およびカバレッジを測定・確認しながら開発を進めます。

#### 1. 出力先ディレクトリ構成

| 種別                   | 出力先パス                                  | 想定ファイル / 形式                                                                                       | Git管理                       |
| ---------------------- | ------------------------------------------- | --------------------------------------------------------------------------------------------------------- | ----------------------------- |
| **テストエビデンス**   | `test-results/evidence/`                    | 実行ログ (`.log`), 実行結果サマリ (`.txt`), スクリーンショット (`.png`), テストレポート (`.xml`, `.html`) | **不要**（`.gitignore` 対象） |
| **カバレッジレポート** | `test-results/coverage/` または `coverage/` | HTML レポート (`index.html` 等), lcov (`lcov.info`), Cobertura (`cobertura.xml`)                          | **不要**（`.gitignore` 対象） |

- 各変更（Change）やタスクで個別に出力・保存する場合は、`test-results/evidence/<change-name>/` や `test-results/evidence/<task-id>/` のようにサブディレクトリを作成して配置してもよい。
- テスト実行ツール（pytest, Jest, Vitest, dotCover, xUnit等）のデフォルト設定に応じて、出力先を上記パスに統一・設定する。

#### 2. エビデンスとカバレッジの取り扱い方針

- **Git管理不要**: テストエビデンスやカバレッジレポートは自動生成されるバイナリ/大容量ファイルを含むため、Git 管理には含めない（`.gitignore` に登録済み）。
- **レビューでの確認**: Apply フェーズの QA Agent レビューや Archive フェーズの総括レビューでは、ローカルの `test-results/` 配下に出力されたログやレポートを参照・検証して合否判定を行う。レビュー記録（`reviews.md`）には、レポートファイルの本文ではなく「カバレッジ率・全テスト通過のサマリ結果」をテキストで記録する。

### 自動フォーマット実行コマンド（ファイル種別ごと）

| ファイル種別                  | 対象拡張子         | 使用ツール | 実行コマンド                              |
| ----------------------------- | ------------------ | ---------- | ----------------------------------------- |
| **Markdown ドキュメント**     | `.md`              | Prettier   | `npx prettier --write "**/*.md"`          |
| **JSON 設定ファイル**         | `.json`            | Prettier   | `npx prettier --write "**/*.json"`        |
| **OpenSpec アーティファクト** | `openspec/**/*.md` | Prettier   | `npx prettier --write "openspec/**/*.md"` |

※実装言語（Python / TypeScript / Kotlin 等）決定時に対応するフォーマッタを追記する。

### Git操作ルール（Commit / Push タイミング）

OpenSpecのアーティファクト（`openspec/` 配下）もGit管理対象とする。

#### Commit タイミング（ローカルリポジトリへの記録）

以下のタイミングで **必ず commit** する:

1. **Propose 合意後** (Phase 1 完了時)
   - `openspec/changes/<name>/` 配下の全アーティファクト（`proposal.md`, `specs/`, `design.md`, `tasks.md`, `reviews.md`）
2. **Apply 中のタスク完了時** (Phase 2 中)
   - 実装成果物（ソースコード、テストコード）
   - `openspec/changes/<name>/tasks.md` の進捗更新
   - `openspec/changes/<name>/reviews.md` 等のレビュー記録
3. **Archive 完了後** (Phase 3 完了時)
   - `openspec/sync specs` で更新されたメイン仕様（`openspec/specs/`）
   - アーカイブされた変更（`openspec/changes/archive/`）
   - README や関連ドキュメント等の更新

**Commit メッセージ規約**:

```
<type>(<scope>): <subject>

<body>

<footer>
```

- type: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `review`, `spec`
- scope: 変更名 (例: `init-setup`) または `spec`、`design`、`impl` 等
- subject: 50文字以内で簡潔に

#### Push タイミング（リモートリポジトリへの反映）

1. **日次または作業区切りのタイミング**（1日1回以上推奨）
2. **Archive 完了・ユーザー最終合意後** (Phase 3 完了時)
   - 成果物が完成し、ユーザー確認済みの状態で push
3. **Pull Request 作成時**

#### 禁止事項

- **ユーザー合意なしでの push は禁止**（特にメインブランチへの直接 push）
- **未フォーマット・未テスト状態での commit は禁止**
- **テストエビデンス（`test-results/`）やカバレッジレポート（`coverage/`）の commit は禁止（Git管理対象外）**
- **作業中の一時的な変更（WIP）を commit する場合は、メッセージに `wip:` プレフィックスを付与し、後で整理すること**

---

### OpenSpec CLI コマンド リファレンス

```bash
# 変更の作成・一覧
openspec new change "<name>"           # 新規変更作成
openspec list --json                   # 変更一覧取得
openspec status --change "<name>" --json  # 進捗確認

# アーティファクト操作
openspec instructions <artifact> --change "<name>" --json  # 作成指示取得
openspec validate --change "<name>"    # 検証

# 実装・完了
openspec instructions apply --change "<name>" --json  # 実装指示取得
openspec sync specs --change "<name>"  # スペック同期
openspec archive --change "<name>"     # アーカイブ

# コンテキスト・設定
openspec context --json                # プロジェクトルート確認
openspec schemas --json                # 利用可能スキーマ一覧
```

---

### サブエージェントレビュープロセス

OpenSpecの3フェーズ（Propose → Apply → Archive）それぞれで、定義された役割のサブエージェントによるレビューを**必須**とする。レビュー記録は `openspec/changes/<name>/reviews.md` にテンプレート準拠で蓄積する。

#### 1. Propose Phase Review（必須）

- **トリガー**: `openspec new change` 完了、`proposal.md` 作成後、Propose合意前
- **レビュアー**:
  - **User Agent**: 要件妥当性・ビジネス価値・UX・受け入れ基準
  - **SE Agent**: 技術的実現性・アーキテクチャ整合・影響範囲・非機能要件
- **観点チェックリスト** (`checklist_propose.md` 使用):
  - What/Why が明確かつ500語以内か
  - Non-goals（やらないこと）が明記されているか
  - 既存仕様（`openspec/specs/`）との矛盾・影響範囲が整理されているか
  - タスク分解粒度が「2時間以内」目安で妥当か
  - 見積もり・スコープに過不足がないか
- **合否**: 全役割LGTMで次フェーズへ。指摘がある場合は `proposal.md` 更新して再レビュー
- **成果物**: `reviews.md` に記録追記

#### 2. Apply Phase Review（各タスク必須）

- **トリガー**: `tasks.md` のチェックボックス `[x]` 更新前（タスク完了宣言前）
- **レビュアー・観点**:
  - **実装タスク** → **PG Agent**（`checklist_apply_impl.md`）
    - 設計書（`design.md`）との整合性
    - コード品質：可読性・命名・関数分割・DRY
    - 境界値・異常系・エラー処理の網羅
    - リファクタの妥当性（テスト変更なし）
    - 自動フォーマット実行済みか
  - **テストタスク** → **QA Agent**（`checklist_apply_test.md`）
    - 仕様妥当性検証テスト：仕様書の期待値をコード化できているか
    - 回帰テスト：既存機能を壊さない観点で網羅できているか
    - 境界値・異常系・エッジケースのテストケース
    - E2E/結合テスト：受け入れシナリオから導出できているか
    - カバレッジ基準達成・エビデンス出力済みか（出力先: `test-results/`）
- **フロー**:
  1. 実装者がタスク完了宣言（PR/コミット前）
  2. 該当役割エージェントがレビュー実施、指摘を `reviews.md` に記録
  3. 指摘への対応方針（採用/保留/却下＋理由）を明記
  4. 実装者が対応コミット
  5. 再レビュー → 全指摘クローズ（LGTM）でタスク完了 `[x]`
- **保留指摘の扱い**: 当該タスクでは完了扱いとし、内容を「申し送り事項」として `reviews.md` および Archive時の別ファイル（`handover.md`）に記録
- **成果物**: `reviews.md` 記録更新、`tasks.md` チェックボックス更新

#### 3. Archive Phase Review（必須）

- **トリガー**: `openspec archive` 実行前
- **レビュアー**: 全役割（User Agent / SE Agent / PG Agent / QA Agent）
- **観点チェックリスト** (`checklist_archive.md` 使用):
  - デルタ仕様がメイン仕様（`openspec/specs/`）へ正しく反映されているか（`openspec sync specs` 実行済み）
  - 全テスト通過・カバレッジ基準達成のエビデンスがあるか（`test-results/` 配下で確認）
  - README や関連ドキュメントが同期更新済みか
  - 既知の課題・技術的負債・保留指摘が `handover.md` に整理記録されているか
  - 変更サマリ（何が変わったか、テスト結果、レビュー指摘対応状況）が記録されているか
- **合否**: ユーザー最終合意で Archive 実行
- **成果物**: 総括レビュー記録を `reviews.md` に追記、`handover.md` 作成
