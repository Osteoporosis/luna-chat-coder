# Recovery

Use this reference after a chat or sandbox reset, after conversational context loss, or whenever the exact relationship between the sandbox working tree and GitHub is unclear.

## Order of trust

```text
commit / PR head
    > immutable Git or Actions artifact
    > surviving sandbox working tree
    > conversation reconstruction
```

Chat preserves intent and explanations. Durable repository state preserves exact source bytes.

## Procedure

1. Inspect the surviving sandbox workspace before assuming it is empty or stale.
2. Identify the expected repository, task/PR if any, and commit SHA from GitHub.
3. Inspect task-owned branches/refs, relevant workflow runs, and artifacts when a previous Actions mission may contain reusable or diagnostic state.
4. If the user supplies a previously downloaded sandbox workspace snapshot, treat it as supplementary working-state evidence and compare it with current durable GitHub identity before adopting or merging its contents. Do not let the snapshot outrank a current commit or PR head merely because Luna created it.
5. Recover exact source from a commit, PR head, Git bundle/archive, artifact, or repository read operation.
6. Compare recovered state with surviving sandbox changes before merge or replacement.
7. Preserve unfamiliar sandbox or remote changes until ownership is understood.
8. Re-read only the project instructions needed to restore the current task boundary; investigate stale documentation instead of letting it override current source or history.
9. Resume from the recovered durable state.

## Drift

Branch and tag names are coordination names and can move. Before applying a payload, publishing a commit, or performing a destructive operation, compare the expected immutable identity with the actual base.

If the base moved, recover the new state and deliberately rebase, merge, or recreate the payload.

## Parallel work and lost context

Use ownership already expressed by a task branch, PR, issue, commit, mission name, or task-owned artifact. Do not overwrite or delete unfamiliar state merely because it appears old or unrelated.

If chat context has been lost, reconstruct the task boundary from durable GitHub state before cleanup or continuation. A failed mission's logs, artifact, branch, or commit may be more reliable recovery evidence than reconstructed conversation text.

## Restart when cheaper

If a failed attempt leaves no reusable commit, patch, diagnosis, environment artifact, or other durable result, clean up task-owned temporary state when safe and restart from the last known durable base.
