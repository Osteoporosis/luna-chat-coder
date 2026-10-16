# Recovery

Use this reference after a chat or sandbox reset, after conversational context loss, or whenever the exact relationship between the sandbox working tree and GitHub is unclear.

## Order of trust

```text
commit / PR head
    > immutable Git or Actions artifact
    > surviving sandbox working tree
    > conversation reconstruction
```

Use this order to establish exact source while preserving and reconciling newer task-owned work. Chat preserves intent and explanations; durable repository state preserves exact source bytes.

## Procedure

1. Inspect the surviving sandbox workspace before assuming it is empty or stale.
2. Identify the expected repository, task branch/PR if any, and exact commit SHA from GitHub, including the latest task checkpoint when one exists. Recover any checkpoint's pending work and verification status; an in-progress checkpoint is not a completion claim.
3. Inspect task-owned branches/refs, relevant workflow runs, and artifacts when a previous Actions mission may contain reusable or diagnostic state.
4. If the user supplies a previously downloaded sandbox workspace snapshot, treat it as supplementary working-state evidence and compare it with current durable GitHub identity before adopting or merging its contents. Do not let the snapshot outrank a current commit or PR head merely because Luna created it.
5. Recover exact source from a commit, PR head, Git bundle/archive, artifact, or repository read operation.
6. Compare recovered state with surviving sandbox changes before merge or replacement.
7. Preserve unfamiliar sandbox or remote changes until ownership is understood.
8. Re-read only the project instructions needed to restore the current task boundary; investigate stale documentation instead of letting it override current source or history.
9. Resume from the recovered durable state and continue frequent origin task-branch checkpoints under [`SKILL.md`](../SKILL.md#checkpoint-in-progress-repository-work).

## Drift

Branch and tag names can move. Verify the payload's source commit before applying it, and follow [Publish exact changes](../SKILL.md#publish-exact-changes) when updating its destination.

If the source or destination differs from the expected state, recover and deliberately rebase, merge, or recreate the payload.

## Parallel work and lost context

Use ownership already expressed by a task branch, PR, issue, commit, mission name, or task-owned artifact. Do not overwrite or delete unfamiliar state merely because it appears old or unrelated.

If chat context has been lost, reconstruct the task boundary from durable GitHub state before cleanup or continuation. A failed mission's logs, artifact, branch, or commit may be more reliable recovery evidence than reconstructed conversation text.

Before replaying an interrupted sandbox command or replacing its inputs, check for surviving work as described in [`sandbox-execution.md`](sandbox-execution.md).

## Restart when cheaper

If a failed attempt leaves no reusable commit, patch, diagnosis, environment artifact, or other durable result, clean up task-owned temporary state when safe and restart from the last known durable base.
