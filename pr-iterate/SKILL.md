---
name: pr-iterate
description: Drive a PR to healthy — satisfaction 5/5 and green CI — by healing gate findings, closing out actionable review threads, pausing on blockers.
disable-model-invocation: true
---

**Iterate** an open pull request until it is **healthy**: **gate** satisfaction **5/5** on the current head SHA and pipeline **green**. **Heal** findings between gates. **Close out** every actionable review thread. **Pause** on `human-needed`, **non-code** blockers, or a **stall**.

Done when the Healthy report below is accurate for the current head SHA.

## Target

Resolve the PR from the user's argument (URL, number, or "current branch"). If none, ask once.

```bash
gh pr view <n> --json number,title,url,headRefOid,labels
gh pr checks <n>
```

Push commits so the gated SHA and the branch tip are the same commit.

## Loop

### 1. Observe

**Done when:** checks for the current `headRefOid` are fetched; you have a **pr-gate** result for that same SHA (latest matching review on the PR, or a fresh run of [`pr-gate`](../pr-gate/SKILL.md)); and unresolved review threads are listed (thread id, path, authors, bodies).

Thread fetch: [`CLOSE_OUT.md`](CLOSE_OUT.md).

### 2. Pause check

**Done when:** no pause condition applies, or you have given a pause report (blocker, evidence, question) and stopped for the user.

**Pause** on:

- `human-needed` — label present, or latest gate applied / warranted it
- **non-code** — secrets/auth, permissions, flaky infra, external outage, branch-protection policy, product/domain decision, or a finding that needs anything outside the repo
- **stall** — a heal → re-gate cycle left satisfaction unchanged

Resume only after the user answers.

### 3. Heal

**Done when:** every **actionable** unresolved thread is **closed out**, every accepted change is on the remote head (committed and pushed), and every skipped non-actionable thread is named in one local line why.

**Actionable** = unresolved thread that requests a code change, reports a defect, or asks a blocking question. Skip acknowledgements, pure FYI, and noise with no concrete ask.

Process Critical → Major → Minor → Nice to have (gate grades), then any remaining actionable threads:

1. **Reflect** — is the suggestion valid for this PR and codebase? Valid → act; invalid → decline. Partial credit is still a decline of the bad part plus a valid fix for the rest.
2. **Act** — apply suggestion blocks when they apply cleanly; otherwise implement. Declines need no code change.
3. **Close out** — reply on the thread, then resolve it. Reply either way (implemented or declined). Mechanics and reply shape: [`CLOSE_OUT.md`](CLOSE_OUT.md).

If already **5/5** and only CI is red, still **close out** any leftover actionable threads, then skip to Pipeline.

### 4. Pipeline

**Done when:** `gh pr checks` is all success on the current head SHA, or you have paused on **non-code**.

Diagnose each failure. Fix in-repo causes; push; re-watch. If failures look unrelated and the branch is behind its base, merge the base once and re-check. If only CI config or out-of-scope packages would clear a check, pause.

### 5. Re-gate

**Done when:** [`pr-gate`](../pr-gate/SKILL.md) has finished on the current `headRefOid` with a satisfaction score and `human-needed` verdict.

**5/5** and green → healthy. Else → Pause check.

## Healthy report

```markdown
## Healthy

**PR:** <url> — <title>
**Satisfaction:** 5/5
**Pipeline:** green
**Iterations:** <n>
**Threads closed out:** <n> (implemented / declined)
**Paused during run:** no | yes (<blocker → how resolved>)
```
