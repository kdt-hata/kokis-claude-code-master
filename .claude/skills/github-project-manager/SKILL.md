---
name: github-project-manager
description: GitHub Project「All AI Project」(kodato-dev/ai-master-project) の管理スキル。イシュー作成、親子関係（sub-issue）の設定・変更、AC（受け入れ条件）の追加編集、プロジェクト階層のツリー可視化、状況サマリー表示。「イシュー作成」「サブイシュー紐付け」「AC追加」「プロジェクト状況」「イシューツリー」などのリクエスト時に使用。
---

# GitHub Project Manager

kodato-dev/ai-master-project リポジトリの「All AI Project」を管理する。

## リポジトリ情報

- **リポジトリ**: `kodato-dev/ai-master-project`
- **プロジェクト**: All AI Project
- **プロジェクト番号**: 8
- **プロジェクトURL**: https://github.com/orgs/kodato-dev/projects/8/views/4

## 重要なルール

**イシュー作成時は必ずプロジェクトに紐付けること**

イシューを作成したら、必ず以下のコマンドでプロジェクト(#8)に追加する：

```bash
gh project item-add 8 --owner kodato-dev --url https://github.com/kodato-dev/ai-master-project/issues/イシュー番号
```

## タスク一覧

### 1. イシューツリー可視化

全イシューの親子関係をツリー形式で表示する。

```bash
gh api graphql -f query='
query {
  repository(owner: "kodato-dev", name: "ai-master-project") {
    issues(first: 100, states: OPEN, orderBy: {field: CREATED_AT, direction: DESC}) {
      nodes {
        number
        title
        parent { number title }
        subIssues(first: 20) { nodes { number title } }
      }
    }
  }
}'
```

出力形式:
```
📁 #3 親イシュータイトル
├── #43 子イシュー1
├── #55 子イシュー2
└── #110 子イシュー3
```

### 2. イシュー作成

親イシュー、担当者、ACを指定してイシューを作成する。

```bash
gh issue create --repo kodato-dev/ai-master-project \
  --title "イシュータイトル" \
  --body "## 概要
説明

## AC
- [ ] 条件1
- [ ] 条件2

## 親イシュー
#親番号" \
  --assignee ユーザー名1,ユーザー名2
```

担当者の例: `kdt-hata`, `kaito-ujiie`

### 3. 親子関係（Sub-issue）設定

イシュー間のsub-issue紐付けを設定する。

**Step 1: イシューIDを取得**
```bash
gh issue view 番号 --repo kodato-dev/ai-master-project --json id -q '.id'
```

**Step 2: Sub-issueとして紐付け**
```bash
gh api graphql -f query='
mutation {
  addSubIssue(input: {
    issueId: "親イシューID",
    subIssueId: "子イシューID"
  }) {
    issue { number title }
    subIssue { number title }
  }
}'
```

**既存の親から外す場合**
```bash
gh api graphql -f query='
mutation {
  removeSubIssue(input: {
    issueId: "現在の親イシューID",
    subIssueId: "子イシューID"
  }) {
    issue { number }
  }
}'
```

### 4. AC（受け入れ条件）追加・編集

イシューのbodyを更新してACを追加する。

```bash
gh issue edit 番号 --repo kodato-dev/ai-master-project --body "## 概要
説明

## AC
- [ ] 条件1
- [ ] 条件2

## 補足
補足情報

## 親イシュー
#親番号"
```

### 5. プロジェクト状況サマリー

```bash
gh api graphql -f query='
query {
  repository(owner: "kodato-dev", name: "ai-master-project") {
    issues(first: 100, states: OPEN) {
      totalCount
      nodes {
        number
        title
        state
        assignees(first: 3) { nodes { login } }
        subIssues(first: 20) {
          totalCount
          nodes { number state }
        }
      }
    }
  }
}'
```

出力例:
```
## プロジェクト状況

📊 全体: オープン XX件
👥 担当者別: kdt-hata XX件 / kaito-ujiie XX件

📁 主要イシュー進捗
#3 タイトル... [3/10]
#106 タイトル... [1/4]
```

### 6. イシュー詳細取得

```bash
gh issue view 番号 --repo kodato-dev/ai-master-project --json number,title,body,state,assignees,url
```
