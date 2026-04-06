# 目次

* [まえがき](preface.md)
* [AIの構造的特性とエンジニアリング課題](chapters/01-framework.md)
  * [Agentの仕組み](chapters/01a-how-agents-work.md)
  * [AIの構造的特性](chapters/01b-characteristics.md)
  * [AI変革が行き詰まる理由](chapters/01c-why-transformation-fails.md)

### Part I: 信頼性あるAgent Programming（1→10倍）

* [信頼性あるデリバリーの出発点: 仕様](chapters/02-specification.md)
  * [入力が出力を決める: Agent時代のボトルネックが仕様である理由](chapters/02a-why-spec.md)
  * [機械のための操作マニュアル: 機械可読な仕様とは何か](chapters/02b-machine-readable-spec.md)
  * [書いた後にどう検証するか: コーディング前にDoc Testingでギャップを発見する](chapters/02c-doc-testing.md)
  * [仕様は明確なのに出力が抜け落ちる: 暗黙知の外在化](chapters/02d-tacit-knowledge.md)
  * [複数のソースが矛盾する: Trust Ringsと裁定ルール](chapters/02e-trust-rings.md)
  * [仕様も腐る: 障害モードと継続的メンテナンス](chapters/02f-spec-decay.md)
* [信頼性あるデリバリーのバックボーン: 自動品質管理](chapters/03-verification.md)
  * [ウィッシュリストから実行可能な制約へ: Test First](chapters/03a-test-first.md)
  * [AIがコードもテストも書く: 共謀問題](chapters/03b-collusion.md)
  * [どのテストが最もごまかしにくいか: Trophy Testingモデル](chapters/03c-trophy-testing.md)
  * [終わってから確認ではなく、やりながら確認: 実行中の継続的フィードバック](chapters/03d-continuous-feedback.md)
  * [出力が速すぎてレビューが追いつかない: AgentでAgentを検証する](chapters/03e-adversarial-verification.md)
  * [では何をすべきか: コードレビューから検証システム設計へ](chapters/03f-new-role.md)

### Part II: Agent開発のスケーリング（10→100倍）

* [Agentを自律稼働させる: 分解、コンテキスト、メモリ](chapters/04-long-running.md)
  * [手を離したら何が起きるか: Context Wall](chapters/04a-context-wall.md)
  * [大きなタスクをAgentサイズに切る: タスク分解](chapters/04b-task-decomposition.md)
  * [コンテキストは多ければ良いわけではない: Context Engineering](chapters/04c-context-engineering.md)
  * [セッション終了、知識はどこへ: クロスセッションメモリエンジニアリング](chapters/04d-memory.md)
  * [タスクが完了しなかったとき: タスクレベルの引き継ぎドキュメント](chapters/task-handoff/04d1-task-handoff.md)
  * [セッションはチャットログではない: 実行単位のエンジニアリング](chapters/04e-session.md)
  * [手取り足取りから自律へ: 実行モードの転換](chapters/04f-letting-go.md)
* [マルチAgent並列処理: 分離と統合](chapters/05-multi-agent.md)
  * [2つのAgentを同時に: 競合と分離](chapters/05a-isolation.md)
  * [個別には正しいが、合わせると爆発する: 契約と統合](chapters/05b-integration.md)
  * [飛行機を飛ばす前に滑走路を作る: Platform Engineering First](chapters/05c-platform.md)
    * [ミリ秒フィードバック: 静的解析とコード標準](chapters/05c1-linters.md)
    * [秒〜分フィードバック: フィードバックチャネルとしてのCI/CD](chapters/05c2-cicd.md)
    * [分〜日フィードバック: Observability](chapters/05c3-observability.md)
    * [ビジネス品質の測定: Benchmark駆動フィードバックループ](chapters/05c4-benchmark.md)
    * [Environment as Code: 信頼性あるフィードバックの基盤としての再現性](chapters/05c5-iac.md)
    * [シグナルが赤になったとき: Agentのトラブルシューティング能力](chapters/05c6-troubleshooting.md)
  * [何体まで管理できるか: Span of Control](chapters/05d-span.md)

### Part III: 100倍組織のガバナンス

* [複数人の協働: ハイブリッドチームの組織設計](chapters/06-hybrid-team.md)
  * [既存のチーム構造がもう機能しない理由](chapters/06a-why-old-structure-fails.md)
  * [ボトルネックの移動: コードから組織へ](chapters/06b-bottleneck-shift.md)
  * [実行ではなくガバナンスを軸に役割を再設計する](chapters/06c-new-roles.md)
  * [プロセスをAgentの速度に合わせる](chapters/06d-process-speed.md)
  * [Agentを投入する前に境界を定義する: Conway's Lawは今も適用される](chapters/06e-conway.md)
  * [銀の弾丸はないが、原則はある](chapters/06f-principles.md)
* [新時代の組織資産](chapters/07-beyond.md)

---

* [コントリビューター](contributors.md)
