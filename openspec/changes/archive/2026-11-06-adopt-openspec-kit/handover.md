# 申し送り事項 (Handover): adopt-openspec-kit

- **変更名**: `adopt-openspec-kit`
- **日付**: 2026-11-06
- **ブランチ**: `chore/adopt-openspec-kit`
- **関連 Issue / PR**: Archive 完了後に `develop` 向け PR を自動作成

---

## 1. 変更サマリ

`AIUtils`（`feature/OpenSpec` ブランチ）の汎用パッケージ `openspec-kit` を `install.ps1`（`-BaseBranch develop -EvidenceDir test-results/ -Force -OverwriteConfig`）によって `ProjectHeimdall` へ導入した。
サブエージェント 6 ロール定義（`openspec/agents/*.md`）、機械的フェーズゲート検証スクリプト（`openspec/scripts/verify-gate.{ps1,sh}`）、SDD ワークフロー（`.agents/workflows/sdd-*.md`）、挑戦的レビュープロトコル対応テンプレート群（`openspec/templates/*.md`）を整備し、`openspec/config.yaml` および `AGENTS.md`（管理区間外と `openspec-kit` 自動管理区間の分離）・`GEMINI.md`・`CLAUDE.md` を刷新した。

---

## 2. 品質・テストエビデンス

- **単体/ゲート検証**: `pwsh ./openspec/scripts/verify-gate.ps1 -Gate apply -ChangeName "adopt-openspec-kit" -BaseBranch develop` → 全項目 Pass（失敗 0 / スキップ 0）
- **結合/CLI検証**: `openspec context --json`（警告 0 件）、`openspec validate "adopt-openspec-kit"`（Valid）、`verify-gate.sh`（LF 改行・CR バイト 0 件）、配置ファイル数確認（6 agents / 2 scripts / 3 workflows / 8 templates）→ 全件 Pass
- **カバレッジ**: 設定・スクリプト・ドキュメント導入変更（`L1` / `skip_specs: true`）のためプロダクションコードカバレッジ対象外（受け入れ基準網羅率 100%）
- **エビデンス出力先**: `test-results/evidence/adopt-openspec-kit/apply-verification.log`

---

## 3. 保留指摘・技術的負債

| 指摘元（フェーズ/タスク/役割）        | 内容                                                                                                                                                                        | 保留理由                                                                                 | 推奨対応時期                         |
| ------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- | ------------------------------------ |
| Apply / Task 1.1〜2.2 / `pg-reviewer` | `AGENTS.md` 管理区間外（L11-18, L48）の `test-results/`・`openspec/` の説明が管理区間内（L65-75）と一部重複しているため、`src/`・`test/`・`doc/` 中心に簡素化する余地がある | 現状でも矛盾はなく可読性上の問題はないため、今回の変更スコープでは現状維持とした         | アプリケーション実装言語・構成確定時 |
| Apply / Task 1.1〜2.2 / `pg-reviewer` | `verify-gate.{ps1,sh}` の `Assert-ReviewsValid` は `reviews.md` 内の最後の `- **LGTM**:` 行のみを機械判定する仕様（上流 `AIUtils/openspec-kit` 仕様）                       | `Non-Goals` により上流キット自体の改修は対象外。最新セクションに最終判定を置く運用で整合 | 上流 `AIUtils/openspec-kit` 更新時   |

---

## 4. 将来の改善・リファクタ候補

- アプリケーションの実装言語・テストランナー確定時に、`openspec/config.yaml` の `context`（技術スタック欄）および `AGENTS.md` 管理区間外の「自動フォーマット」表に言語固有のフォーマッタ・リンター・テストコマンドを追記する。
- `AIUtils/openspec-kit` 側が更新された際は、`pwsh "D:\programming\repos\MyGitHubRepos\AIs\AIUtils\openspec-kit\install.ps1" -TargetPath . -BaseBranch develop -EvidenceDir test-results/ -Force -SkipConfig` で再同期する。

---

## 5. 運用上の注意事項

- **`AGENTS.md` の編集ルール**: `<!-- openspec-kit:begin -->` 〜 `<!-- openspec-kit:end -->` の区間は `install.ps1` による再同期で上書きされるため、プロジェクト固有のルールは必ずマーカー区間より上部（管理区間外）に記述すること。
- **ロールバック手順**: 本変更は設定・テンプレート・スクリプトの追加と更新のみであり、DB や外部システムへの副作用はない。万一問題が発生した場合はマージコミットの `git revert -m 1 <commit-hash>` で即座に巻き戻し可能。
