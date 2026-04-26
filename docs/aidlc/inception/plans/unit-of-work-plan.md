# Unit of Work Plan — post-manager-system

> AI-DLC Inception Phase | Units Generation — Part 1 (Planning)  
> 生成日時: 2026-04-27

---

## 前提情報

- **アーキテクチャ**: DDD 4層モノレポ（frontend / backend）
- **デプロイモデル**: 単一サービス（モノリス）
- **バウンデッドコンテキスト**: Mail（中核）/ User / Notification / Statistics
- **ユーザーストーリー数**: 20（US-01〜US-20）

---

## Step 1: 計画ステップ

- [ ] Step A — ユニット分割方針の確定
- [ ] Step B — `unit-of-work.md` 生成
- [ ] Step C — `unit-of-work-dependency.md` 生成
- [ ] Step D — `unit-of-work-story-map.md` 生成
- [ ] Step E — 境界検証・全ストーリーの割り当て確認

---

## Step 2: 質問事項

> [Answer]: タグにご回答ください。空欄のまま送信いただくと次のステップへ進みます。

---

### Q1. ユニット分割粒度（Story Grouping）

モノレポ・モノリス構成において、バックエンドのユニット分割をどの粒度で行いますか？

**選択肢**:

- **A** — バウンデッドコンテキスト単位（Mail / User / Notification / Statistics の 4 ユニット + Frontend 1 ユニット = 計 5 ユニット）
- **B** — フロントエンド／バックエンド の 2 ユニット（バックエンドは DDD モジュールで内部分割）
- **C** — 機能グループ単位（認証管理 / 郵便物管理 / 通知・設定 / 統計・レポート の 4 グループ + Frontend = 計 5 ユニット）

[Answer]: 

---

### Q2. Notification ユニットの扱い（Business Domain）

Notification は汎用サブドメインで実装量が比較的少ない（Webhook アダプター 2 つ + NotificationConfig のみ）です。どう扱いますか？

- **A** — 独立ユニットとして扱う（拡張性を重視）
- **B** — Mail Management ユニットに内包する（実装の凝集度を重視）

[Answer]: 

---

### Q3. フロントエンドの分割方針（Code Organization）

SPA フロントエンドをどう分割しますか？

- **A** — 1 ユニット（frontend 全体を単一ユニットとして扱う）
- **B** — バックエンドのユニットに対応して機能単位で分割（例：auth-ui / mail-ui / stats-ui）

[Answer]: 

---

### Q4. 開発優先順位（Dependencies）

ユニット間の実装順序について優先方針を教えてください。

- **A** — 依存関係ベース（User → Auth 基盤 → Mail → Notification → Statistics の順）
- **B** — ビジネス優先度ベース（コア機能 Mail を先行、統計・設定は後回し）
- **C** — 並列可能な部分は同時進行（学習目的のため 1 人開発だが構成は並列を意識）

[Answer]: 

---

### Q5. コード構成方針（Code Organization — Greenfield）

モノレポ内のディレクトリ構成について確認です。Application Design で下記を定義していますが、ユニット単位で追加の構造変更はありますか？

現行案：
```
backend/src/
  presentation/  domain/  application/  infrastructure/
frontend/src/
  pages/  features/  api/  store/
```

- **A** — そのままで問題ない（DDD レイヤー軸のまま進める）
- **B** — ドメイン内をユニット（BC）ごとにサブディレクトリで明確に分けたい（例: `domain/mail/`, `domain/user/` など ← Application Design ではすでに定義済み）
- **C** — その他（具体的に記載）

[Answer]: 

---

### Q6. チーム構成・担当割り当て（Team Alignment）

学習目的の 1 人開発ですが、将来のチーム分担を意識したユニット設計にしますか？

- **A** — 1 人開発として最適化（シンプルな分割で OK）
- **B** — 将来の担当分離を意識（BC 単位でオーナーシップが明確になる設計）

[Answer]: 

---

## Step 3: 計画ファイル情報

- **保存先**: `docs/aidlc/inception/plans/unit-of-work-plan.md`（このファイル）
- **生成予定成果物**:
  - `docs/aidlc/inception/application-design/unit-of-work.md`
  - `docs/aidlc/inception/application-design/unit-of-work-dependency.md`
  - `docs/aidlc/inception/application-design/unit-of-work-story-map.md`

---

## 回答記録（AI が記入）

> ユーザーの回答を受領後に以下を記入する。

| 質問 | 回答 | 決定内容 |
|---|---|---|
| Q1 | — | — |
| Q2 | — | — |
| Q3 | — | — |
| Q4 | — | — |
| Q5 | — | — |
| Q6 | — | — |
