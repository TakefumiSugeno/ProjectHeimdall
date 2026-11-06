# Proposal

## Why

現在の `ProjectHeimdall` の OpenSpec 構成は立ち上げ初期の簡易版（旧4ロール定義・手動管理の `AGENTS.md`・機械的ゲート検証なし）であり、`AIUtils`（`feature/OpenSpec` ブランチ）で確立された最新の `openspec-kit`（TDD 3権分立の6ロール分業、C0/C1/C2 カバレッジ基準、挑戦的レビュープロトコル、機械的フェーズゲート `verify-gate.ps1`）と乖離している。本格的なプロダクト開発を開始する前に `openspec-kit` を流用・導入し、品質保証とレビューの仕組みを標準化する。

## What Changes

- **変更種別 / 適用規模レベル**: `chore` / `L1`（開発ワークフロー・OpenSpec 基盤設定の導入、`skip_specs: true`）
- **`openspec-kit` の導入（`install.ps1` 実行）**:
  - ベースブランチを `develop`、テストエビデンス出力先を `test-results/` として `AIUtils/openspec-kit/install.ps1` を適用する。
  - **サブエージェント 6 ロール定義の追加** (`openspec/agents/`): `qa-tester.md`, `pg-implementer.md`, `user-reviewer.md`, `se-reviewer.md`, `pg-reviewer.md`, `qa-reviewer.md` を配置する。
  - **機械的フェーズゲートスクリプトの追加** (`openspec/scripts/`): `verify-gate.ps1` および `verify-gate.sh`（Gate 0〜3 検証）を配置する。
  - **SDD ワークフローの追加** (`.agents/workflows/`): `sdd-propose.md`, `sdd-apply.md`, `sdd-archive.md` を配置する。
  - **テンプレート群の更新・追加** (`openspec/templates/`): `test_plan.md`, `handover.md`, `pr_body.md` を新規追加し、`review_record.md` および `checklist_*.md` を挑戦的レビュープロトコル対応版へ更新する。
  - **`openspec/config.yaml` の刷新**: 6 ロール定義、規模レベル（L1/L2/L3）、C0/C1/C2 テスト網羅性基準、フェーズゲート運用ルール（ベースブランチ: `develop`）へ更新し、OpenSpec CLI で未サポートだった `operations.propose` キーの警告を解消する。
  - **`AGENTS.md` / `CLAUDE.md` / `GEMINI.md` の整理**: `AGENTS.md` 上部に `ProjectHeimdall` 固有の構成・自動フォーマット定義をまとめ、下部に `openspec-kit` 自動管理区間（`<!-- openspec-kit:begin/end -->`）を配置して旧記述との重複を解消する。`CLAUDE.md` および `GEMINI.md` は `@AGENTS.md` 参照に統一する。

## Non-goals

- `ProjectHeimdall` のアプリケーションコード（`src/`, `test/`）の実装・変更は行わない。
- `AIUtils` 固有の仕様書（`openspec/specs/review-framework/spec.md`）は流用せず、`ProjectHeimdall` の `openspec/specs/` はプロダクト仕様向けに空のまま維持する。
- `.agents/skills/` や `.gemini/` 等の `openspec init` / `openspec update` 生成ファイルの手動編集は行わない。

## Capabilities

### New Capabilities

なし（開発ツール・OpenSpec 設定の整備であり、プロダクト仕様の変更を伴わないため `skip_specs: true` を設定）

### Modified Capabilities

なし

## Acceptance Criteria

- **Given** `AIUtils`（`feature/OpenSpec`）の `openspec-kit` が存在するとき、**When** `install.ps1` を `-BaseBranch develop -EvidenceDir test-results/ -Force -OverwriteConfig` で適用しプロジェクト固有部を調整すると、**Then** `openspec/agents/`（6ファイル）、`openspec/scripts/verify-gate.{ps1,sh}`、`.agents/workflows/sdd-*.md`（3ファイル）、`openspec/templates/`（8ファイル）がすべて配置されること。
- **Given** 更新後の `openspec/config.yaml` が配置されたとき、**When** `openspec context --json` および `openspec validate "adopt-openspec-kit"` を実行すると、**Then** `Unknown operation ID 'propose'` の警告が出力されず、検証が正常終了すること。
- **Given** `verify-gate.ps1` が配置されたとき、**When** `pwsh ./openspec/scripts/verify-gate.ps1 -Gate apply -ChangeName "adopt-openspec-kit" -BaseBranch develop` を実行すると、**Then** エラー 0 件で PASS すること。

## Impact

- **追加ファイル**: `openspec/agents/*.md`, `openspec/scripts/verify-gate.{ps1,sh}`, `.agents/workflows/sdd-*.md`, `openspec/templates/{test_plan,handover,pr_body}.md`, `CLAUDE.md`
- **更新ファイル**: `openspec/config.yaml`, `openspec/templates/{review_record,checklist_*}.md`, `AGENTS.md`, `GEMINI.md`
- **既存仕様・コードへの影響**: `openspec/specs/` および `src/` への破壊的変更なし。
