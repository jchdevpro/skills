---
name: pr-gate
description: Score an open PR 1–5, post graded review comments (with suggestions), APPROVE or REQUEST_CHANGES, apply `human-needed` when blast radius warrants it.
disable-model-invocation: true
---

**Gate** an open pull request: read the full change, judge every axis below, post one GitHub review whose comments are **graded** and titled (with a **suggestion** when a concrete fix fits), set the review event to **APPROVE** or **REQUEST_CHANGES**, emit a **satisfaction** score (1–5), and apply `human-needed` when **blast radius** warrants a human.

Done when every axis has a verdict, every finding is posted as a graded comment (or the review body states there were none), the review event matches the table below, the score matches the scale, `human-needed` is applied via `gh` or explicitly ruled out, and the local report mirrors what was posted.

## Target

Resolve the PR from the user's argument (URL, number, or "current branch"). If none, ask once.

```bash
gh pr view <n> --json number,title,url,baseRefName,headRefName,headRefOid,files,additions,deletions,labels
gh pr diff <n>
```

Review the **diff that merges**, not local uncommitted work.

## Axes

Judge each axis. Every finding that moves the score becomes a graded PR comment with a path/line when it maps to a hunk.

### Quality

Readable, consistent with surrounding code, honest names, error paths handled where the change introduces risk. Skip what lint/CI already enforces.

### Maintainability

A future reader can change this without archaeology. Locality beats cleverness; the edit belongs where similar edits already live.

### Simplicity

Favour the smallest design that meets the change. Flag **speculative generality** — abstractions, knobs, layers, or "flexibility" the PR does not need yet. Prefer delete/inline over a framework for one call site.

### Coverage

Behaviour the diff introduces or alters has tests (or an explicit, credible reason they are impossible here). Missing coverage on non-trivial logic pulls the score down.

### Meaningful tests

Tests assert outcomes a user/caller would care about — behaviour and contracts — not implementation trivia (private call sequences, snapshot noise, tautologies). A green suite that would still pass if the bug shipped is a finding.

### Blast radius

How far a mistake could reach: auth/permissions, money/data integrity, migrations, shared libraries, public APIs, wide fan-out, risky rollback. High blast radius does not automatically fail the gate; it triggers **human-needed**.

## Grades

Every finding carries exactly one grade — the discriminator in the comment title:

| Grade | Use when | Score ceiling |
| --- | --- | --- |
| **Critical** | Wrong/unsafe behaviour, security, data loss, broken contract in production paths | **1** |
| **Major** | Material quality, simplicity, coverage, or meaningful-test gap that should block confidence | **2** |
| **Minor** | Clear defect or smell; fix before merge preferred, not catastrophic | **3** |
| **Nice to have** | Optional polish; ship remains reasonable without it | no cap |

The **worst grade** on the PR sets the satisfaction ceiling (table above). **Nice to have** alone can still be a **5**. No findings → score from residual blast-radius judgement only (usually **5** or **4**).

## Review event

Post **one** review on the head SHA. Pick `event` from the worst grade (not from the numeric score alone):

| Worst grade | `event` |
| --- | --- |
| **Critical** or **Major** | `REQUEST_CHANGES` |
| **Minor** or **Nice to have** only, or no findings | `APPROVE` |

If GitHub rejects `APPROVE` or `REQUEST_CHANGES` (own PR, branch protection, missing write access), retry once with `COMMENT`, keep the same body and inline comments, and say which event was intended and why it was demoted. Never use `COMMENT` as the first choice when `APPROVE` or `REQUEST_CHANGES` applies.

Summary in the review body; one inline comment per finding that maps to a line; non-anchorable findings listed only in the review body under the same title/grade shape.

## Comments

### Comment shape

```markdown
### [Grade] Short title

<why this matters — one short paragraph>

```suggestion
<replacement lines when a concrete fix is clear>
```
```

Omit the `suggestion` block when the fix is structural, cross-file, or not a drop-in replacement — say what to change instead. Suggestions must apply cleanly to the commented lines on the RIGHT (head) side.

Title rules: `[Critical|Major|Minor|Nice to have]` then a specific noun phrase (`[Major] Unvalidated workspace id on delete`). No ungraded comments.

### Submit

```bash
# owner/repo from gh; HEAD = headRefOid from gh pr view
# event: APPROVE | REQUEST_CHANGES (see Review event)
gh api --method POST repos/{owner}/{repo}/pulls/<n>/reviews --input - <<'EOF'
{
  "commit_id": "<headRefOid>",
  "event": "REQUEST_CHANGES",
  "body": "<report body — Satisfaction, axis table, human-needed, weakest axis>",
  "comments": [
    {
      "path": "path/to/file.ts",
      "line": 42,
      "side": "RIGHT",
      "body": "### [Major] Title\n\nWhy…\n\n```suggestion\nfixed\n```"
    }
  ]
}
EOF
```

Use multi-line comments (`start_line` + `line`) when the suggestion spans a range. If the API rejects a line (diff drift), retry once with a corrected line or demote that finding to the review body — still graded and titled.

## Satisfaction

Pick one integer. Worst **grade** sets the ceiling; within that band, the **weakest axis** decides the exact number.

| Score | Meaning |
| --- | --- |
| **5** | Safe to ship. No Critical/Major/Minor; only optional nits or none. |
| **4** | Ship-ready; at most **Nice to have**, or trivial residual risk. |
| **3** | Mixed — **Minor** findings (or justified test gaps). Address before calling it production-confident. |
| **2** | Weak — at least one **Major**. |
| **1** | Unsafe — at least one **Critical**, or unreviewably poor. |

A **Coverage** / **Meaningful tests** gap on non-trivial logic is at least **Major** unless the report justifies why tests are impossible here.

## human-needed

Apply when **blast radius** is high *or* confidence in production safety is low (ambiguous domain rules, security-sensitive paths, irreversible migrations, or review blocked on missing context).

```bash
gh label create human-needed --description "Needs human review before merge" --color B60205 2>/dev/null || true
gh pr edit <n> --add-label human-needed
```

If the label already exists, leave it and say so. If not warranted, state why — do not remove an existing `human-needed` unless the user asks.

## Report

Mirror the posted review body locally:

```markdown
## Satisfaction: N/5

**PR:** <url> — <title>
**Review event:** APPROVE | REQUEST_CHANGES | COMMENT (demoted)
**human-needed:** applied | not warranted (<one reason>)
**Comments:** <k> inline

| Axis | Verdict | Evidence |
| --- | --- | --- |
| Quality | … | … |
| Maintainability | … | … |
| Simplicity | … | … |
| Coverage | … | … |
| Meaningful tests | … | … |
| Blast radius | low / medium / high | … |

### Findings
- `[Grade] Title` → `path:line` (or review body)

### Weakest axis
…
```
