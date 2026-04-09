# 実践：AILock-Step の検証パイプライン

前の2節で原則とメカニズムを確立しました。この節では 02d の OKR システム事例を引き続き使い、これらのメカニズムが AILock-Step フレームワークでどのように実装されるかを示します。02d では spec が一文から実行可能なドキュメントへと発展する全過程を示しました。ここではその続きとして、spec 完成後に検証基盤をどう構築し、どう実行し、コード統合前に意図の逸脱をどう検知するかを見ていきます。

3つのステップを順に展開します。アーキテクチャドキュメントで API contract を定義する（検証の骨格）、spec から結合テストを生成する（実行可能な制約）、独立した Agent で spec 一致性レビューを行う（テストの死角を補完する）。

## 第1ステップ：API Contract の定義

02d の spec にはすでに API contract の表が含まれており、6つのインターフェースの Method、URL、Parameters が列挙されています。この表はフロントエンドとバックエンドの通信境界を定義していますが、まだ具体性が足りず、テストの構築に直接使える段階ではありません。

テストに必要な情報は「このインターフェースが存在する」ことだけではありません。何を受け取り、何を返し、どのような場合にエラーになるかも知る必要があります。フレームワークはアーキテクチャドキュメントの段階で contract をテスト可能なインターフェース仕様に拡張します。OKR システムの「目標の新規作成」インターフェースを例に取ります。

```yaml
POST /system/okrObjective:
  request:
    body:
      employeeId: { type: integer, required: true }
      quarter: { type: string, required: true, pattern: "YYYY-QN" }
      title: { type: string, required: true, maxLength: 200 }
  response:
    success: { code: 200, body: { msg: "新增成功", data: { objectiveId: integer } } }
    errors:
      - { condition: "title 为空", code: 500, msg: "目标标题不能为空" }
      - { condition: "quarter 格式不合法", code: 500, msg: "季度格式错误" }
```

この contract は3つのことを行っています。各フィールドの型と制約を定義し（title は最大200文字、spec の Key Rule に対応）、成功レスポンスの構造を定義し（RuoYi の AjaxResult 形式と一致、project-context.md の規約に対応）、エラーシナリオと対応するエラーメッセージを定義しています。

この contract があれば、フロントエンド開発は mock server を構築してバックエンドインターフェースをシミュレートでき、バックエンド開発は contract の入力パラメータを使って直接インターフェースをテストできます。両側が独立して開発を進め、それぞれ contract に基づいて検証を行います。

## 第2ステップ：Spec から結合テストを生成する

Spec が完成し、contract が定義されたら、次はテストの生成です。フレームワークの `/generate-tests` skill は2つのソースからテストケースを抽出します。spec の受け入れ基準とビジネスルール、そして API contract のインターフェース仕様です。

受け入れ基準から生成されるテストは、ユーザーが知覚できる振る舞いをカバーします。OKR システムの場合、13件の acceptance criteria と5件の business rules の合計18件のルールが7つの E2E テストケースにマッピングされます。マッピング関係は明示的です。

```markdown
| テストケース | 対応する Spec 項目 | 検証内容 |
|---------|----------------|---------|
| test-01 | AC-1, BR-004 | 目標の新規作成：必須フィールドの検証 + 四半期のデフォルト値 |
| test-02 | AC-2 | 目標リスト：四半期と従業員による絞り込み |
| test-03 | AC-3 | 目標の編集：タイトルとステータスの変更 |
| test-04 | AC-4, BR-001 | 目標の削除：関連 KR の連鎖削除 |
| test-05 | AC-5 | 目標リスト：関連 KR 数の表示 |
| test-06 | BR-002, BR-003 | KR の制約：数量上限5件 + 達成率の範囲 0-100 |
| test-07 | BR-005 | 四半期概要：読み取り専用、編集ボタンなし |
```

このマッピング表自体がトレーサビリティドキュメントです。どの受け入れ基準にも対応するテストケースが見つかり、どのテストケースもそれが検証する spec 項目まで遡れます。後日 spec が変更された場合（例えば BR-005 を削除した場合）、マッピング表がどのテストを同期して調整すべきかを即座に示してくれます。

API contract から生成されるテストはインターフェースレベルの振る舞いをカバーします。「目標の新規作成」インターフェースの場合、contract が定義した3つのフィールドの制約と2つのエラーシナリオが、直接インターフェーステストに変換されます。

```typescript
describe('POST /system/okrObjective', () => {
  test('成功创建目标', async () => {
    const res = await request.post('/system/okrObjective', {
      employeeId: 1,
      quarter: '2026-Q2',
      title: '提升客户满意度'
    });
    expect(res.body.code).toBe(200);
    expect(res.body.data).toHaveProperty('objectiveId');
  });

  test('title 为空时返回错误', async () => {
    const res = await request.post('/system/okrObjective', {
      employeeId: 1,
      quarter: '2026-Q2',
      title: ''
    });
    expect(res.body.code).toBe(500);
  });

  test('title 超过 200 字符时返回错误', async () => {
    const res = await request.post('/system/okrObjective', {
      employeeId: 1,
      quarter: '2026-Q2',
      title: 'x'.repeat(201)
    });
    expect(res.body.code).toBe(500);
  });
});
```

これらのテストは Agent がコーディングを始める前から存在しています。実行結果はすべて赤です。インターフェースがまだ実装されていないからです。Agent がコーディングを開始し、インターフェースを1つ完成させるたびにテストを実行します。赤が緑に変わるのが進捗であり、赤のままであるのが乖離です。

注意点として、E2E テストとインターフェーステストは検証するレイヤーが異なります。E2E テストはユーザー操作の観点から完全な振る舞いパスを検証し、インターフェーステストは contract の観点から個々のインターフェースの入出力を検証します。両者は相互補完の関係にあり、E2E テストが受け入れ検証の主力で、インターフェーステストはより細かい粒度での特定を提供します。

## 第3ステップ：Code Review Against Spec

Agent がコーディングを完了すると、フレームワークの `/review-against-spec` skill が独立した review セッションを起動します。Review agent は新しい context で実行され、コーディング過程のいかなる情報も持っていません。入力は Agent のコード変更と元の spec であり、出力は spec 一致性レポートです。

OKR システムの場合、Agent が第1ラウンドのコーディングを完了した後の review agent のレポート構造は以下の通りです。

```markdown
## Spec Consistency Report

### Acceptance Criteria
| # | Criterion | Status | Evidence |
|---|-----------|--------|----------|
| AC-1 | 新增目标必须选择员工、季度、标题 | PASS | OkrObjectiveController.add() validates required fields |
| AC-4 | 删除目标同时删除关联 KR | PASS | @Transactional + keyResultMapper.deleteByObjectiveId() |
| ... | | | |

### Business Rules
| # | Rule | Status | Finding |
|---|------|--------|---------|
| BR-001 | 级联删除 | PASS | — |
| BR-002 | KR数量限制 max 5 | PARTIAL | 前端有 Element UI 提示，后端 ServiceImpl 缺少数量校验。绕过前端直接调 API 可突破限制。 |
| BR-003 | 完成率范围 0-100 | PASS | — |
| BR-004 | 默认季度 | FAIL | handleAdd() 未设置 form.quarter 默认值。新增对话框打开时季度下拉框为空。 |
| BR-005 | 总览只读 | PASS | — |

### Scope Check
- 检测到 login.vue 被修改（+12 行），但 spec 中未涉及登录模块。标记为 OUT_OF_SCOPE 变更。

### Summary
- AC: 13/13 PASS
- BR: 3/5 PASS, 1 PARTIAL, 1 FAIL
- Scope: 1 out-of-scope change detected
- Recommendation: 修复 BR-002 和 BR-004 后重新提交
```

このレポートは、code review がテストの死角を補完する具体的な方法を示しています。BR-002 の問題（バックエンドに数量バリデーションがない）は、テストではフロントエンドの振る舞いによって隠蔽される可能性があります。E2E テストはフロントエンド操作を通じて行われるため、フロントエンドのプロンプトが6番目の KR の追加を阻止し、テストは通過しますが、バックエンドの脆弱性は残ったままです。Review agent は spec とコードを1項目ずつ照合することで、実装が半分しかカバーしていないことを発見しました。

Scope check もテストではカバーできない領域です。Agent が login.vue を変更しても、テストは OKR 関連のページのみをカバーするため、ログインモジュールはトリガーされません。しかし review agent は diff をスキャンした結果、この変更が spec の範囲外であることを検知し、人間の判断に委ねるためにフラグを立てます。

## 検証の完全なチェーン

3つのステップが完了しました。contract が境界を定義し、テストが境界上に実行可能な制約を構築し、code review がテストではカバーできない意図の漂流を補完しました。人間が見るべきものはコードではなく、2つのレポートです。テスト結果（振る舞いが正しいか）と spec 一致性レポート（意図が整合しているか）です。両方のレポートが spec の構造に直接対応し、逸脱と欠落に焦点を当てています。

ここまでで、規約（第2章）と検証（本章）が単一タスクのクローズドループを構成しました。1人の人間と Agent で、1つの feature を spec からコード、そしてマージまでの全プロセスを確実に完了できます。

このクローズドループは、以降のすべての内容の基盤です。複数の Agent を同時に走らせて異なる feature を並行開発し始めるとき（第2巻のテーマ）、本章で構築した contract、テスト基盤、review メカニズムは、良いエンジニアリングプラクティスから生存の前提条件へと変わります。Contract がなければ、複数の Agent の成果物を統合した時点で衝突が起きます。テスト基盤がなければ、統合後の問題を特定のモジュールまで切り分けられません。独立した review がなければ、人間のレビュー能力が複数 Agent の産出速度に追いつけません。

---

*Harness Engineering Playbook · [AgentsZone](https://agentszone.ai) Community*
