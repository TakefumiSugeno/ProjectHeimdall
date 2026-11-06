# Tasks

## 1. ブランチ準備と `openspec-kit` の一括導入

- [ ] 1.1 リモート `origin/develop` を fetch・チェックアウトしてローカル `develop` ブランチを作成し、トピックブランチ `chore/adopt-openspec-kit` を作成・切り替えする（依存: なし）
- [ ] 1.2 `AIUtils/openspec-kit/install.ps1` を `-TargetPath . -BaseBranch develop -EvidenceDir test-results/ -Force -OverwriteConfig` で実行し、`openspec/agents/`（6ファイル）、`openspec/scripts/verify-gate.{ps1,sh}`、`.agents/workflows/sdd-*.md`（3ファイル）、`openspec/templates/`（8ファイル）、`CLAUDE.md` が配置されたことを確認する（依存: 1.1）

## 2. プロジェクト固有設定の調整とクリーンアップ

- [ ] 2.1 `AGENTS.md` の管理区間外（上部）を `ProjectHeimdall` の構成・自動フォーマット表・実装ガイドラインに整理し、旧来の手動 OpenSpec 記述を削除して `<!-- openspec-kit:begin/end -->` 管理区間と重複がない状態にする（依存: 1.2）
- [ ] 2.2 `openspec/config.yaml` のプロジェクト固有部（`技術スタック: TODO` の更新、1:1 ファイル構成ルールの反映、ベースブランチ `develop` の確認）および `GEMINI.md`（`@AGENTS.md` 相対参照への明示的書き換え）を整え、`install.ps1` が生成したバックアップファイル（`openspec/config.yaml.bak-*`）を削除する（依存: 1.2）

## 3. フォーマット・仕様妥当性・フェーズゲート検証

- [ ] 3.1 Prettier（`npx prettier --write "**/*.md"` 等）を実行し、以下の受け入れ基準（仕様妥当性検証＋回帰確認）をテスト・検証してエビデンスを `test-results/evidence/adopt-openspec-kit/` に記録する（依存: 2.1, 2.2）:
  - `openspec context --json` および `openspec validate "adopt-openspec-kit"` が警告・エラーなしで成功すること
  - `pwsh ./openspec/scripts/verify-gate.ps1 -Gate apply -ChangeName "adopt-openspec-kit" -BaseBranch develop` が PASS し、`verify-gate.sh` が LF 改行で配置されていること

## Workflow follow-up

- 全タスク完了・レビュー LGTM 後、`handover.md` の作成と総括レビューを経て `openspec archive "adopt-openspec-kit"` を実行する。
- `chore/adopt-openspec-kit` ブランチを push し、`develop` 向けの Pull Request を作成する。
