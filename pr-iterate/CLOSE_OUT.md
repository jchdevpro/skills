# Close out — thread reply + resolve

Disclosed reference for [`pr-iterate`](SKILL.md). Load when **closing out** an actionable review thread.

## List unresolved threads

```bash
gh api graphql -f query='
query($owner:String!, $repo:String!, $number:Int!) {
  repository(owner:$owner, name:$repo) {
    pullRequest(number:$number) {
      reviewThreads(first:100) {
        nodes {
          id
          isResolved
          isOutdated
          path
          comments(first:20) {
            nodes {
              databaseId
              author { login }
              body
              createdAt
            }
          }
        }
      }
    }
  }
}' -F owner='OWNER' -F repo='REPO' -F number=N
```

Keep only `isResolved: false`. Read comment body, author, path, and thread `id` — skip the rest of the payload.

## Reply

Reply on the thread's first comment (`databaseId`), then resolve.

**Bot / automated** — compact:

```markdown
**Valid.** Implemented: <one line>.
```

```markdown
**Declined.** <one line why>.
```

**Human reviewer** — same shape, one or two short sentences of rationale or what changed:

```bash
gh api repos/OWNER/REPO/pulls/PR/comments/COMMENT_ID/replies \
  -f body='**Valid.** Implemented: …'
```

## Resolve

```bash
gh api graphql -f query='
mutation($id:ID!) {
  resolveReviewThread(input: { threadId: $id }) {
    thread { isResolved }
  }
}' -f id='THREAD_NODE_ID'
```

**Close out** = reply posted **and** `isResolved: true`. Never resolve without a reply. Never leave an actionable thread open after acting.
