# アプリケーション設計書 — post-manager-system

> AI-DLC Inception Phase | Application Design  
> 生成日時: 2026-04-26

---

## 設計方針

| 項目 | 決定内容 |
|---|---|
| **アーキテクチャ** | DDD（Domain-Driven Design）4層アーキテクチャ |
| **フロントエンド** | SPA（Single Page Application） |
| **バックエンド API** | RESTful API |
| **認証方式** | JWT トークンベース（ステートレス） |
| **通知実装** | バックエンド内組み込み・同期処理 |
| **統計集計** | オンデマンド集計（リクエスト時に SQL 集計） |
| **リポジトリ構成** | モノレポ（frontend / backend ディレクトリ分割） |

---

## バウンデッドコンテキスト

```mermaid
flowchart LR
    subgraph CORE["中核ドメイン"]
        MAIL["📬 Mail Management\n郵便物の登録・ワークフロー\nステータス管理"]
    end

    subgraph SUPPORT["支援サブドメイン"]
        USER["👥 User Management\nユーザー・ロール管理\nJWT 認証"]
        STATS["📊 Statistics\n月次・年次集計\n担当者別サマリ"]
    end

    subgraph GENERIC["汎用サブドメイン"]
        NOTIF["🔔 Notification\nSlack/Teams\nWebhook 送信"]
    end

    MAIL -->|"受取確認 → recipient"| USER
    MAIL -->|"登録トリガー"| NOTIF
    MAIL -->|"集計データ"| STATS
```

---

## レイヤーアーキテクチャ

```mermaid
flowchart TB
    subgraph PRES["🌐 プレゼンテーション層\nHTTP エンドポイント・ミドルウェア"]
        direction LR
        MC["MailController"] 
        AC["AuthController"]
        UC_C["UserController"]
        SC["StatisticsController"]
        SEC["SettingsController"]
        JWT_MW["JWTAuthMiddleware"]
        RBAC_MW["RBACMiddleware"]
    end

    subgraph APP["⚙️ アプリケーション層\nUse Cases（オーケストレーション）"]
        direction LR
        REG["RegisterMail"]
        CONF["ConfirmReceipt"]
        SRCH["SearchMails"]
        LOGIN["Login"]
        STATS_UC["GetStatistics"]
        USERS_UC["ManageUsers"]
        ARCH["ArchiveMails"]
        NOTIF_UC["ConfigureNotification"]
    end

    subgraph DOM["🏛️ ドメイン層\nビジネスルール・不変条件"]
        direction LR
        subgraph MAIL_BC["Mail BC"]
            M["Mail Entity"]
            MVO["Sender / MailStatus\nMailType Value Objects"]
            MWS["MailWorkflowService"]
            IML["IMailRepository"]
        end
        subgraph USER_BC["User BC"]
            U["User Entity"]
            UVO["Email / Role\nValue Objects"]
            IUL["IUserRepository"]
        end
        subgraph NOTIF_BC["Notification BC"]
            NC["NotificationConfig"]
            INP["INotificationPort"]
        end
        subgraph STATS_BC["Statistics BC"]
            SDS["StatisticsDomainService"]
        end
    end

    subgraph INFRA["🔧 インフラストラクチャ層\n外部システム・永続化"]
        direction LR
        MRI["MailRepositoryImpl"]
        URI["UserRepositoryImpl"]
        SLACK["SlackWebhookAdapter"]
        TEAMS["TeamsWebhookAdapter"]
        JWT_S["JWTService"]
        DB[("RDBMS")]
    end

    PRES --> APP
    APP --> DOM
    INFRA -.->|"implements interfaces"| DOM
    MRI & URI --> DB

    style PRES fill:#BBDEFB,stroke:#1565C0,stroke-width:2px
    style APP fill:#C8E6C9,stroke:#2E7D32,stroke-width:2px
    style DOM fill:#FFF9C4,stroke:#F57F17,stroke-width:2px
    style INFRA fill:#FCE4EC,stroke:#AD1457,stroke-width:2px
```

---

## ディレクトリ構成（モノレポ）

```
post-manager-system/
├── frontend/                    # SPA フロントエンド
│   └── src/
│       ├── pages/               # 画面コンポーネント
│       ├── features/            # 機能別モジュール
│       ├── api/                 # API クライアント
│       └── store/               # 状態管理
│
├── backend/                     # バックエンド API
│   └── src/
│       ├── presentation/        # Controllers, Middleware, DTOs
│       ├── application/         # Use Cases
│       ├── domain/              # Entities, Value Objects, Domain Services, Interfaces
│       │   ├── mail/
│       │   ├── user/
│       │   ├── notification/
│       │   └── statistics/
│       └── infrastructure/      # Repository Impl, Adapters, JWTService
│
└── docs/
    └── aidlc/                   # AI-DLC ドキュメント
```

---

## API エンドポイント一覧

| Method | Path | 責務 | 必要ロール |
|---|---|---|---|
| `POST` | `/api/auth/login` | ログイン・JWT発行 | — |
| `POST` | `/api/auth/logout` | ログアウト | 全員 |
| `POST` | `/api/auth/refresh` | トークンリフレッシュ | 全員 |
| `GET` | `/api/mails` | 郵便物一覧 | 全員 |
| `GET` | `/api/mails/search` | 郵便物検索 | 全員 |
| `POST` | `/api/mails` | 郵便物登録 | Admin |
| `PUT` | `/api/mails/:id` | 郵便物編集 | Admin |
| `DELETE` | `/api/mails/:id` | 郵便物削除 | Admin |
| `POST` | `/api/mails/:id/confirm` | 受取確認 | Handler |
| `GET` | `/api/mails/archived` | アーカイブ一覧 | Admin |
| `POST` | `/api/mails/archive` | アーカイブ実行 | Admin |
| `DELETE` | `/api/mails/archived` | アーカイブ削除 | Admin |
| `GET` | `/api/users` | ユーザー一覧 | Admin |
| `POST` | `/api/users` | ユーザー作成 | Admin |
| `PUT` | `/api/users/:id` | ユーザー更新 | Admin |
| `GET` | `/api/statistics/monthly` | 月次統計 | Admin / Viewer |
| `GET` | `/api/statistics/yearly` | 年次統計 | Admin / Viewer |
| `GET` | `/api/statistics/summary` | サマリ統計 | Admin / Viewer |
| `GET` | `/api/settings/notifications` | Webhook 設定取得 | Admin |
| `PUT` | `/api/settings/notifications` | Webhook 設定更新 | Admin |
| `POST` | `/api/settings/notifications/test` | テスト通知送信 | Admin |

---

## 主要フロー: 郵便物登録〜通知〜受取確認

```mermaid
sequenceDiagram
    actor ADM as 管理者（田中）
    actor HDL as 担当者（山田）
    participant FE as Frontend（SPA）
    participant MW as Middleware
    participant CTRL as MailController
    participant UC as RegisterMailUseCase
    participant DOM as MailWorkflowService
    participant DB as RDBMS
    participant NP as NotificationAdapter

    ADM->>FE: 郵便物登録フォーム入力
    FE->>MW: POST /api/mails + JWT
    MW->>MW: JWT 検証 / RBAC（Admin）
    MW->>CTRL: 認証済みリクエスト
    CTRL->>UC: RegisterMailCommand
    UC->>DB: Mail 保存（status: UNNOTIFIED）
    UC->>NP: Slack/Teams 通知送信
    NP-->>UC: 送信成功
    UC->>DOM: notifyMail（status → NOTIFIED）
    UC->>DB: Mail 更新（status: NOTIFIED）
    UC-->>CTRL: RegisterMailResult
    CTRL-->>FE: 201 Created
    FE-->>ADM: 登録完了

    Note over HDL: Slack 通知受信
    HDL->>FE: ログイン → 郵便物一覧
    FE->>CTRL: POST /api/mails/:id/confirm + JWT
    CTRL->>UC: ConfirmReceiptCommand
    UC->>DOM: confirmReceipt（status → CONFIRMED）
    UC->>DB: Mail 更新（status: CONFIRMED）
    UC-->>CTRL: void
    CTRL-->>FE: 200 OK
    FE-->>HDL: 受取確認完了
```

---

## 参照ドキュメント

| ドキュメント | 内容 |
|---|---|
| [components.md](components.md) | コンポーネント定義・責務 |
| [component-methods.md](component-methods.md) | メソッドシグネチャ |
| [services.md](services.md) | サービス間オーケストレーション・フロー |
| [component-dependency.md](component-dependency.md) | 依存関係マトリクス・データフロー |
| [../requirements/requirements.md](../requirements/requirements.md) | 要件定義書 |
| [../user-stories/stories.md](../user-stories/stories.md) | ユーザーストーリー |
