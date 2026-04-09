# 目次

* [本書について](README.md)
* [はじめに：なぜHarness Engineeringが必要なのか](chapters/00-introduction.md)

### 第一巻：信頼性あるAgent Programming（1→10x）

* [仕様：Agentとの意図アラインメント](chapters/02a-intent-alignment.md)
  * [意図アラインメント：Vibe Codingはなぜ失敗するのか](chapters/02a-intent-alignment.md)
  * [構造で意図を伝える：階層化と次元](chapters/02b-structured-intent.md)
  * [イテレーションで実行可能な仕様を練り上げる](chapters/02c-iterative-spec.md)
  * [実践：AILock-Step Feature Workflow](chapters/02d-case-study.md)
* [検証：コードが仕様に忠実であることを確保する](chapters/03-verification.md)
  * [検証のアンカーは規約である](chapters/03-verification.md)
  * [テスト基盤の先行構築：仕様を実行可能な制約に変える](chapters/03a-test-first.md)
  * [Code Review：テストでは捕捉できない意図ドリフトを補う](chapters/03b-code-review.md)
  * [実践：AILock-Stepの検証パイプライン](chapters/03c-practice.md)
* [進化：規約と検証の継続的イテレーション](chapters/evolution-v1.md)
* [第一巻の振り返り：Closed Loopから進化へ](chapters/v1-conclusion.md)

### 第二巻：Agent開発のスケーリング（10→100x）

* [Agentを自律稼働させる：分解、コンテキスト、メモリ](chapters/04-long-running.md)
  * [コンテキスト崩壊：長期タスクが制御不能になる構造的理由](chapters/04a-context-wall.md)
  * [タスク分解：実行ブロックの粒度を制御する](chapters/04b-task-decomposition.md)
  * [Context Engineering：Agentが何を見るかを決める](chapters/04c-context-engineering.md)
  * [クロスセッション永続化：メモリと引き継ぎ](chapters/04d-memory.md)
* [マルチAgent並列処理：分離と統合](chapters/05-multi-agent.md)
  * [分離：Agent間の並行競合を防ぐ](chapters/05a-isolation.md)
  * [統合：独立した成果物の一貫性を確保する](chapters/05b-integration.md)
  * [Platform Engineering：多層フィードバック基盤の構築](chapters/05c-platform.md)
* [進化：手動巡回から自動化ドリフト検知へ](chapters/evolution-v2.md)

### 第三巻：100倍組織のガバナンス

* [組織再構築：Agentが協働の前提を変えるとき](chapters/06-hybrid-team.md)
  * [既存構造が機能しなくなる理由](chapters/06a-why-old-structure-fails.md)
  * [ボトルネックの移動：コードから組織へ](chapters/06b-bottleneck-shift.md)
  * [プロセスをAgentの速度に合わせる](chapters/06c-process-speed.md)
* [役割の再定義：コードを書くことから検証体制の設計へ](chapters/07-role-redefinition.md)
  * [ガバナンスを軸に役割を再設計する](chapters/07a-new-roles.md)
  * [暗黙の調整を明示的な仕組みに置き換える](chapters/07b-coordination.md)
* [進化：組織資産と新たなモート](chapters/evolution-v3.md)

---

* [コントリビューター](contributors.md)
