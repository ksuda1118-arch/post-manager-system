# Application Design Plan — post-manager-system

> AI-DLC Inception Phase | Application Design

---

## 設計計画チェックリスト

- [x] Step 1: 要件・ユーザーストーリーの分析
- [x] Step 2: 設計計画の作成
- [x] Step 3: 設計質問の生成（本ファイル）
- [x] Step 4: ユーザー回答の収集（Q1 → DDD、Q2〜Q7 → 推奨値で確定）
- [x] Step 5: 回答の曖昧さ分析（曖昧さなし）
- [x] Step 6: 設計成果物の生成
  - [ ] components.md
  - [ ] component-methods.md
  - [ ] services.md
  - [ ] component-dependency.md
  - [ ] application-design.md（統合ドキュメント）

---

## 分析サマリ

要件・ユーザーストーリーから以下の設計上の決定点を特定しました。

**コンポーネント候補（暫定）**:
- フロントエンド（SPA/MPA）: 画面・状態管理
- バックエンド API: REST エンドポイント群
- 認証・認可: RBAC ミドルウェア
- 郵便物管理: CRUD + ワークフロー
- 通知サービス: Slack/Teams Webhook 送信
- 統計サービス: 月次・年次集計
- データアクセス層: DB 操作

---

## 設計質問

> 各 `[Answer]:` に回答を記入してください。

---

### Q1. バックエンドのアーキテクチャスタイル

バックエンドのレイヤー構成をどのパターンで実装しますか？

A) **Controller → Service → Repository**（3層レイヤード）— 最も一般的。責務分離が明確
B) **MVC**（Model-View-Controller）— フレームワーク組み込みの構成に従う
C) **まだ決めていない** — NFR Requirements ステージで技術スタックと一緒に決める
D) **DDD（Domain-Driven Design）** — Presentation / Application / Domain / Infrastructure の4層。ドメインモデル中心設計

[Answer]: D — DDD ベース（ユーザー指定）

---

### Q2. フロントエンドのレンダリング方式

フロントエンドの構成はどちらを想定しますか？

A) **SPA**（Single Page Application）— React / Vue 等。API を呼んでクライアントで描画
B) **SSR / MPA**（Server Side Rendering）— サーバーで HTML を生成してブラウザへ返す
C) **まだ決めていない** — NFR Requirements ステージで決める

[Answer]: A

---

### Q3. 通知サービスの実装方式

Slack/Teams への Webhook 通知をどのように実装しますか？

A) **バックエンド内に組み込み・同期処理** — 郵便物登録 API が直接 Webhook を呼ぶ（シンプル）
B) **バックエンド内に組み込み・非同期処理** — 登録後にキューへ積んでバックグラウンド送信
C) **独立した通知マイクロサービス** — 通知専用のサービスを別に用意する

[Answer]: A

---

### Q4. 認証の実装アプローチ

認証・セッション管理の実装方針を選択してください。

A) **セッションベース**（サーバー側セッション + Cookie）— シンプル。学習目的に適切
B) **JWT トークンベース**（ステートレス）— SPA との相性が良い。スケーラブル
C) **まだ決めていない** — NFR Requirements ステージで決める

[Answer]: B

---

### Q5. フロントエンド・バックエンドのリポジトリ構成

コードをどのように管理しますか？

A) **モノレポ**（同一リポジトリに frontend / backend ディレクトリを配置）
B) **別リポジトリ**（frontend と backend で GitHub リポジトリを分ける）

[Answer]: A

---

### Q6. API 設計スタイル

バックエンドの API 設計はどちらにしますか？

A) **RESTful API**（HTTP メソッド + リソース URL 設計）— 標準的
B) **GraphQL** — フロントエンドの柔軟なデータ取得が可能
C) **まだ決めていない**

[Answer]: A

---

### Q7. 統計ダッシュボードのデータ集計タイミング

ダッシュボードの統計データはどのように集計しますか？

A) **オンデマンド集計**（画面表示時に SQL で集計）— シンプルだがデータ量が増えると遅くなる
B) **定期バッチ集計**（夜間バッチ等で集計テーブルを事前作成）— 高速だが複雑
C) **まだ決めていない**（データ量が小さいので A で十分な見込み）

[Answer]: A
