# Story Generation Plan — post-manager-system

> ユーザーの「進めてください」指示に基づき、Planning と Generation を連続実行。

## Approach

**Breakdown Method**: Epic-Based（JIRA エピック PMS-1〜PMS-7 に対応）

**Story Format**: 標準テンプレート
```
As a [ペルソナ], I want to [アクション], so that [ベネフィット].
```

**Acceptance Criteria Format**: Given/When/Then（簡略形式）

---

## Execution Checklist

### PART 1: Planning
- [x] Step 1: User Stories の必要性を評価（Assessment 完了）
- [x] Step 2: Story plan 作成
- [x] Step 3: 質問生成（要件が充分なため省略 — ユーザー承認済み）
- [x] Step 4: 必須成果物の確認（stories.md / personas.md）
- [x] Step 5: Epic-Based アプローチを選定
- [x] Step 6: 本ファイルに Plan を保存

### PART 2: Generation
- [x] Step 15: Story Plan 読み込み
- [x] Step 16: ペルソナ生成（personas.md）
- [x] Step 16: ユーザーストーリー生成（stories.md + ユースケース図）
- [x] Step 17: 進捗を aidlc-state.md に反映

---

## Stories to Generate

| ストーリー ID | タイトル | Epic | JIRA |
|---|---|---|---|
| US-01 | ログイン | FR-03 | PMS-14 |
| US-02 | 郵便物登録 | FR-01 | PMS-8 |
| US-03 | 郵便物一覧表示 | FR-01 | PMS-9 |
| US-04 | 郵便物編集（管理者） | FR-01 | PMS-10 |
| US-05 | 郵便物削除（管理者） | FR-01 | PMS-11 |
| US-06 | 受取確認（担当者） | FR-02 | PMS-13 |
| US-07 | ステータス管理 | FR-02 | PMS-12 |
| US-08 | Slack 通知送信 | FR-04 | PMS-17 |
| US-09 | Teams 通知送信 | FR-04 | PMS-18 |
| US-10 | Webhook URL 設定 | FR-04 | PMS-19 |
| US-11 | 基本検索 | FR-05 | PMS-20 |
| US-12 | ステータスフィルタ | FR-05 | PMS-21 |
| US-13 | 月次統計グラフ | FR-06 | PMS-22 |
| US-14 | 年次統計集計 | FR-06 | PMS-23 |
| US-15 | 担当者別・種別別サマリ | FR-06 | PMS-24 |
| US-16 | ユーザー管理 | FR-03 | PMS-16 |
| US-17 | ロール管理 | FR-03 | PMS-15 |
| US-18 | データアーカイブ | FR-07 | PMS-25 |
| US-19 | データ削除 | FR-07 | PMS-26 |
| US-20 | 保管期間設定 | FR-07 | PMS-27 |
