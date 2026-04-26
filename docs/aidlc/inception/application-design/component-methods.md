# コンポーネントメソッド定義 — post-manager-system

> 詳細なビジネスロジックは CONSTRUCTION フェーズの Functional Design で定義する。
> ここではメソッドシグネチャと責務の概要を示す。

---

## ドメイン層

### Entity: `Mail`

```typescript
class Mail {
  readonly id: MailId
  readonly receivedAt: Date
  readonly sender: Sender
  readonly recipientId: UserId
  readonly type: MailType
  readonly memo: string
  status: MailStatus
  notifiedAt?: Date
  confirmedAt?: Date
  isArchived: boolean

  // ファクトリメソッド
  static create(params: CreateMailParams): Mail

  // ドメインメソッド（ビジネスルールはMailWorkflowServiceが管理）
  markAsNotified(notifiedAt: Date): void
  confirmReceipt(confirmedAt: Date): void
  archive(): void
}
```

### Entity: `User`

```typescript
class User {
  readonly id: UserId
  readonly email: Email
  passwordHash: string
  role: Role
  isActive: boolean

  static create(params: CreateUserParams): User

  deactivate(): void
  changeRole(newRole: Role): void
  hasPermission(action: Action): boolean
}
```

### Entity: `NotificationConfig`

```typescript
class NotificationConfig {
  slackWebhookUrl?: WebhookUrl
  teamsWebhookUrl?: WebhookUrl

  static create(params: CreateNotificationConfigParams): NotificationConfig
  updateSlackUrl(url: WebhookUrl | null): void
  updateTeamsUrl(url: WebhookUrl | null): void
  hasAnyConfig(): boolean
}
```

### Domain Service: `MailWorkflowService`

```typescript
class MailWorkflowService {
  // ステータス遷移の妥当性を検証し遷移を実行
  notifyMail(mail: Mail, notifiedAt: Date): void
  confirmReceipt(mail: Mail, userId: UserId, confirmedAt: Date): void
  // 遷移可能かどうかを確認
  canConfirm(mail: Mail, userId: UserId): boolean
}
```

### Domain Service: `StatisticsDomainService`

```typescript
class StatisticsDomainService {
  calcMonthlyStats(mails: Mail[], year: number): MonthlyStats[]
  calcYearlyStats(mails: Mail[]): YearlyStats[]
  calcSummaryByRecipient(mails: Mail[], users: User[]): RecipientSummary[]
  calcSummaryByType(mails: Mail[]): TypeSummary[]
}
```

### Repository Interface: `IMailRepository`

```typescript
interface IMailRepository {
  findById(id: MailId): Promise<Mail | null>
  findAll(criteria: MailSearchCriteria): Promise<PaginatedResult<Mail>>
  save(mail: Mail): Promise<void>
  delete(id: MailId): Promise<void>
  findForStatistics(year: number): Promise<Mail[]>
  archiveBefore(date: Date): Promise<number>       // 件数返却
  deleteArchived(): Promise<number>
}
```

### Repository Interface: `IUserRepository`

```typescript
interface IUserRepository {
  findById(id: UserId): Promise<User | null>
  findByEmail(email: Email): Promise<User | null>
  findAll(): Promise<User[]>
  save(user: User): Promise<void>
}
```

### Port Interface: `INotificationPort`

```typescript
interface INotificationPort {
  send(payload: NotificationPayload): Promise<NotificationResult>
}

interface NotificationPayload {
  recipientName: string
  receivedAt: Date
  senderName: string
  mailType: MailType
}

interface NotificationResult {
  success: boolean
  error?: string
}
```

---

## アプリケーション層（Use Cases）

### `RegisterMailUseCase`

```typescript
class RegisterMailUseCase {
  execute(command: RegisterMailCommand): Promise<RegisterMailResult>
}

interface RegisterMailCommand {
  receivedAt: Date
  senderName: string
  senderAddress: string
  recipientId: string
  type: string
  memo: string
}

interface RegisterMailResult {
  mailId: string
  notificationSent: boolean
}
```

### `GetMailListUseCase`

```typescript
class GetMailListUseCase {
  execute(query: GetMailListQuery): Promise<PaginatedMailList>
}

interface GetMailListQuery {
  page: number
  limit: number
  sortBy: 'receivedAt' | 'status'
  sortOrder: 'asc' | 'desc'
  requesterId: string   // RBAC フィルタ用
}
```

### `SearchMailsUseCase`

```typescript
class SearchMailsUseCase {
  execute(query: SearchMailsQuery): Promise<PaginatedMailList>
}

interface SearchMailsQuery {
  fromDate?: Date
  toDate?: Date
  senderName?: string
  recipientId?: string
  status?: string[]
  page: number
  limit: number
}
```

### `ConfirmReceiptUseCase`

```typescript
class ConfirmReceiptUseCase {
  execute(command: ConfirmReceiptCommand): Promise<void>
}

interface ConfirmReceiptCommand {
  mailId: string
  requesterId: string   // 担当者本人確認
}
```

### `LoginUseCase`

```typescript
class LoginUseCase {
  execute(command: LoginCommand): Promise<LoginResult>
}

interface LoginCommand {
  email: string
  password: string
}

interface LoginResult {
  accessToken: string
  refreshToken: string
  user: UserDTO
}
```

### `GetStatisticsUseCase`

```typescript
class GetStatisticsUseCase {
  getMonthly(query: MonthlyStatsQuery): Promise<MonthlyStats[]>
  getYearly(): Promise<YearlyStats[]>
  getSummary(query: SummaryQuery): Promise<StatisticsSummary>
}
```

### `ManageUsersUseCase`

```typescript
class ManageUsersUseCase {
  createUser(command: CreateUserCommand): Promise<UserDTO>
  updateRole(command: UpdateRoleCommand): Promise<void>
  deactivateUser(command: DeactivateUserCommand): Promise<void>
  listUsers(): Promise<UserDTO[]>
}
```

### `ConfigureNotificationUseCase`

```typescript
class ConfigureNotificationUseCase {
  getConfig(): Promise<NotificationConfigDTO>
  updateConfig(command: UpdateNotificationConfigCommand): Promise<void>
  testSend(command: TestNotificationCommand): Promise<TestNotificationResult>
}
```

### `ArchiveMailsUseCase`

```typescript
class ArchiveMailsUseCase {
  preview(command: ArchivePreviewCommand): Promise<ArchivePreview>  // 対象件数確認
  execute(command: ArchiveCommand): Promise<ArchiveResult>
}

interface ArchiveCommand {
  beforeDate: Date
}
```

---

## プレゼンテーション層

### `MailController`

```typescript
class MailController {
  // GET /api/mails
  list(req: Request): Promise<Response>
  // GET /api/mails/search
  search(req: Request): Promise<Response>
  // POST /api/mails
  create(req: Request): Promise<Response>          // Admin only
  // PUT /api/mails/:id
  update(req: Request): Promise<Response>          // Admin only
  // DELETE /api/mails/:id
  delete(req: Request): Promise<Response>          // Admin only
  // POST /api/mails/:id/confirm
  confirmReceipt(req: Request): Promise<Response>  // Handler only
}
```

### `JWTAuthMiddleware`

```typescript
class JWTAuthMiddleware {
  handle(req: Request, next: NextFunction): Promise<void>
  // JWT を検証し req.user に User を付与、無効なら 401
}
```

### `RBACMiddleware`

```typescript
class RBACMiddleware {
  // 必要ロールを受け取りミドルウェアを生成するファクトリ
  require(roles: Role[]): MiddlewareFunction
}
```

---

## インフラストラクチャ層

### `SlackWebhookAdapter` / `TeamsWebhookAdapter`

```typescript
class SlackWebhookAdapter implements INotificationPort {
  send(payload: NotificationPayload): Promise<NotificationResult>
  // POST https://hooks.slack.com/... にメッセージを送信
}

class TeamsWebhookAdapter implements INotificationPort {
  send(payload: NotificationPayload): Promise<NotificationResult>
  // POST https://outlook.office.com/webhook/... にメッセージを送信
}
```

### `JWTService`

```typescript
class JWTService {
  generateAccessToken(user: User): string
  generateRefreshToken(user: User): string
  verifyAccessToken(token: string): JWTPayload
  verifyRefreshToken(token: string): JWTPayload
}
```
