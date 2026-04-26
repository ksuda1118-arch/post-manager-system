# CLAUDE.md — post-manager-system

このファイルは Claude Code が自動的に読み込む設定・規約ファイルです。

---

## GitHub 操作

### トークンの取得パターン（必須）

`.bashrc` の冒頭に非インタラクティブシェル向けの early return があるため、
`source ~/.bashrc` ではトークンが設定されない。
また Bash ツールはコマンド間でシェル変数が引き継がれない。
**トークン取得と git/gh 操作は必ず同一コマンドチェーンで実行すること。**

```bash
# git push
TOKEN=$(grep 'CLAUDE_GITHUB_TOKEN' ~/.bashrc | sed 's/.*="\(.*\)"/\1/') && \
git -c credential.helper="" push "https://x-access-token:${TOKEN}@github.com/ksuda1118-arch/post-manager-system.git"

# gh CLI（issue 作成・PR 作成など）
TOKEN=$(grep 'CLAUDE_GITHUB_TOKEN' ~/.bashrc | sed 's/.*="\(.*\)"/\1/') && \
GH_TOKEN="$TOKEN" gh <subcommand> ...
```

### NG パターン

```bash
source ~/.bashrc && git push          # NG: early return で変数未設定
GH_TOKEN="$CLAUDE_GITHUB_TOKEN" ...  # NG: CLAUDE_GITHUB_TOKEN が空
```

### リポジトリ情報

- **GitHub リポジトリ**: `https://github.com/ksuda1118-arch/post-manager-system`
- **GitHub アカウント**: `ksuda1118-arch`
- **デフォルトブランチ**: `master`

---

## JIRA / Confluence 連携

認証情報は `~/.bashrc` の非インタラクティブ guard より後ろに定義されているため、
直接 `grep` で取得すること。

```bash
# JIRA / Confluence API 呼び出しパターン
JIRA_EMAIL=$(grep 'JIRA_EMAIL' ~/.bashrc | sed 's/.*="\(.*\)"/\1/')
JIRA_TOKEN=$(grep 'JIRA_API_TOKEN' ~/.bashrc | sed 's/.*="\(.*\)"/\1/')
curl -u "$JIRA_EMAIL:$JIRA_TOKEN" -H "Accept: application/json" \
  "https://ksuda1118.atlassian.net/rest/api/3/..."
```

| サービス | 情報 |
|---|---|
| **JIRA インスタンス** | `https://ksuda1118.atlassian.net` |
| **JIRA プロジェクト** | `PMS`（post-manager-system） |
| **Confluence スペース** | `postmanage`（post-manager-system） |
| **Confluence ホームページ ID** | `5832872` |
| **Atlassian MCP サーバー** | `atlassian`（ユーザー設定に登録済み・次セッションから利用可） |

---

## AI-DLC ワークフロー

### ドキュメント出力先

このプロジェクトでは標準の `aidlc-docs/` の代わりに `docs/aidlc/` を使用する。

```
docs/aidlc/
├── aidlc-state.md          # フェーズ進捗管理
├── audit.md                # 全操作の監査ログ
├── inception.md            # Inception フェーズ索引
├── inception/
│   ├── plans/              # 計画ドキュメント
│   ├── requirements/       # 要件定義
│   └── user-stories/       # ユーザーストーリー・ペルソナ
└── construction/           # (今後作成)
```

### ルール参照先

- コアワークフロー: `~/aidlc-workflows/aidlc-rules/aws-aidlc-rules/core-workflow.md`
- ルール詳細: `~/aidlc-workflows/aidlc-rules/aws-aidlc-rule-details/`

### 現在のフェーズ状態

`docs/aidlc/aidlc-state.md` を参照すること。

### Extension 設定

| Extension | 状態 |
|---|---|
| Security Baseline | **無効**（学習目的） |
| Property-Based Testing | **有効**（PBT-01〜PBT-10 全ルール） |

---

## 環境情報

- **OS**: Linux (WSL2)
- **Shell**: bash（非インタラクティブ時は `.bashrc` の early return に注意）
- **Python**: 3.12（`python3` コマンド）
- **Node**: npm 利用可
