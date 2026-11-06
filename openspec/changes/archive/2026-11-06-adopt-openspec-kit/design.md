# Design

## Context

背景・目的については `proposal.md`（Why / What Changes）を参照。
現在、`D:\programming\repos\MyGitHubRepos\AIs\AIUtils`（`feature/OpenSpec` ブランチ）の `openspec-kit/` には、他リポジトリへ一括導入するためのインストーラ（`install.ps1`）とテンプレート一式が存在する。一方、`ProjectHeimdall` には初期セットアップ時の `AGENTS.md`（手動記述の OpenSpec ルールを含む）と旧 `openspec/config.yaml` が存在する。

## Goals / Non-Goals

**Goals:**

- `AIUtils/openspec-kit/install.ps1` を用いて、再現性のある形で `ProjectHeimdall` に最新キットを導入する。
- `AGENTS.md` の手動記述部分と `openspec-kit` 管理区間（`<!-- openspec-kit:begin/end -->`）の重複を解消し、今後 `install.ps1 -Force -SkipConfig` で安全に再同期できる構造にする。
- ベースブランチを `develop`、テストエビデンス出力先を `test-results/` として `openspec/config.yaml` および `AGENTS.md` に一貫して反映する。

**Non-Goals:**

- `AIUtils/openspec-kit` 側のスクリプトやテンプレート自体の改修は行わない。
- UI/UX 変更を伴わないため、モックアップ（`mockup.html`）は作成しない。

## Decisions

1. **導入方式: 手動コピーではなく `install.ps1` を使用**
   - **選択**: `pwsh "D:\programming\repos\MyGitHubRepos\AIs\AIUtils\openspec-kit\install.ps1" -TargetPath . -BaseBranch develop -EvidenceDir test-results/ -Force -OverwriteConfig` を実行する。
   - **理由**: プレースホルダ（`{{BASE_BRANCH}}` → `develop`、`{{EVIDENCE_DIR}}` → `test-results/`）の展開、PowerShell スクリプトの UTF-8 BOM 付与、Bash スクリプトの LF 改行統一、`AGENTS.md` のマーカー区間挿入を自動かつ確実に処理できるため。
   - **代替案**: 個別ファイルの `Copy-Item`。プレースホルダ置換漏れや改行コード不整合のリスクがあるため却下。

2. **`AGENTS.md` の構造整理（管理区間外と管理区間の分離）**
   - **選択**: `AGENTS.md` の上部（`openspec-kit:begin` より前）には `ProjectHeimdall` 固有の「プロジェクト概要・ディレクトリ構成」「自動フォーマット実行コマンド」「実装ガイドライン」のみを残し、旧来の手動 OpenSpec 運用ルール記述（Git 操作ルール・旧サブエージェントレビュープロセス等）は削除して `openspec-kit` 管理区間に一本化する。
   - **理由**: `install.ps1` を既存 `AGENTS.md` に対してそのまま実行すると、末尾にマーカー区間が追記されて旧記述と新記述が二重になるため。

3. **エントリファイル（`GEMINI.md` / `CLAUDE.md`）の統一**
   - **選択**: `GEMINI.md`（現在は絶対パス `@D:\...` 記述）および新規 `CLAUDE.md` を相対参照 `@AGENTS.md` に統一する。
   - **理由**: クローン先環境に依存しないポータビリティを確保するため。

4. **バックアップファイルのクリーンアップ**
   - **選択**: `-OverwriteConfig` 実行時に生成される `openspec/config.yaml.bak-*` は差分確認後に削除し、Git 管理対象に残さない。

## Risks / Trade-offs

- **[Risk] `install.ps1` による `openspec/config.yaml` 上書きでプロジェクト固有設定が失われる** → **Mitigation**: 現在の `ProjectHeimdall/openspec/config.yaml` の内容はすべて `openspec-kit/config.yaml` の上位互換に含まれていることを事前確認済み。導入後に `context` の技術スタック欄（`TODO` プレースホルダ）およびソースコードとテストコードの 1:1 ファイル構成ルールを確認・反映する。
- **[Risk] `verify-gate.ps1` の `-BaseBranch` プレースホルダ置換確認** → **Mitigation**: `AIUtils/openspec-kit/scripts/verify-gate.ps1` の `$BaseBranch = '{{BASE_BRANCH}}'` が `install.ps1 -BaseBranch develop` 実行によって `'develop'` に正しく置換されていることを確認する。

## Migration Plan

1. Gate 0 相当のブランチ切り替え（リモート `origin/develop` からローカル `develop` を作成し、トピックブランチ `chore/adopt-openspec-kit` を切る）。
2. `install.ps1` を実行してファイル一式を展開・更新する。
3. `AGENTS.md` 管理区間外・`openspec/config.yaml`・`GEMINI.md` を整え、一時バックアップファイルを削除する。
4. Prettier によるフォーマットと `openspec validate` / `verify-gate.ps1` による検証を実行する。
