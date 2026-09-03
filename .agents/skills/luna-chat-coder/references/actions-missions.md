# Actions Missions

Use an Actions mission when the normal sandbox or connected GitHub path cannot safely or efficiently provide a required capability, exact transport, or execution step.

A useful mental model is an unmanned probe: dispatch it with an exact target, payload, and return contract; let it operate independently; then inspect its logs, artifacts, checks, or durable Git result after it terminates. Do not treat GitHub Actions as a live shell connected to the chat.

## Choose the smallest useful mission

Common mission roles are:

- **supply mission**: obtain or prepare a repository-required external input the sandbox cannot obtain directly;
- **transport mission**: carry exact repository source or an exact change payload between the sandbox and durable GitHub/Actions state when a byte-preserving transfer is safer or more efficient than direct operations;
- **degraded execution mission**: substitute bounded remote edit/build/test/debug or verification work for the sandbox engineering loop only while the sandbox itself is unavailable or cannot sustain that work.

These are roles, not an exhaustive taxonomy or separate infrastructure. Supply and transport missions may execute bounded acquisition, build/package, apply, integrity, or output-verification commands needed to produce or validate their payloads; those commands do not by themselves constitute degraded remote mode while the sandbox remains the primary engineering loop. A bounded mission may also perform task-owned remote control such as cleanup when the connected integration can establish ownership and terminal state but cannot perform the required operation directly. Keep each mission as small as practical.

## Mission contract

Before dispatch, define the smallest sufficient mission:

- **source identity**: repository plus expected commit or PR-head SHA;
- **purpose**: the capability, transport, or bounded execution need being handled;
- **inputs**: exact files, patch/bundle, lockfiles, versions, parameters, or other required state;
- **operations**: explicit commands or workflow steps;
- **outputs**: artifact, logs, checksum, generated input, test result, commit, or other durable result expected back;
- **integrity**: checksums and provenance when bytes cross the sandbox/runner boundary;
- **permissions**: minimum workflow and repository permissions required;
- **trust boundary**: which mission inputs are untrusted and whether the mission has secrets or a privileged token; do not execute untrusted code with those privileges;
- **terminal state**: what makes the mission complete and what temporary state can eventually be removed.

If the expected source SHA no longer matches, stop that mission path and deliberately recover/rebase rather than applying an exact payload to the wrong source.

After every mission terminates, including a reported success, inspect the conclusion and verify the expected outputs against the mission contract before consuming them or making follow-up decisions. A green workflow status alone does not prove that the intended artifact, commit, ref, checksum, source identity, or cleanup result is correct. Verify the outputs that matter for that mission; failures then require the additional diagnosis described below before retry or source modification.

## Supply mission

Use a supply mission when the sandbox can do the engineering work but cannot obtain a required external input.

A supply mission should:

1. check out the expected repository commit when repository context is required;
2. read the repository's lockfiles, runtime/toolchain declarations, and relevant configuration;
3. determine the sandbox target OS/architecture and any relevant ABI/runtime compatibility before acquiring native payloads;
4. obtain only the required dependency, runtime, SDK, compiler, executable/application distribution, installer/package, native input, generated data, package cache, vendor tree, archive, or similar input;
5. prefer the ecosystem's normal pinned/offline-compatible form;
6. record provenance including source, repository SHA, sandbox target, runner OS/architecture, relevant tool/runtime versions, and production commands;
7. checksum the returned payload;
8. upload only the required result with a bounded retention period;
9. verify provenance, checksum, and platform compatibility before consuming it in the sandbox.

Runner-native output is not presumed compatible with the sandbox. Prefer platform-independent packages when appropriate, or deliberately acquire/build for the sandbox target rather than for the Actions runner merely because the runner produced the payload.

A supplied dependency cache, package set, vendor tree, portable application tree, or installer may be materialized into the project-expected sandbox location and consumed offline. That does not imply the supplied bytes belong in source control; follow the repository's own policy for `vendor/`, caches, toolchains, generated inputs, and install roots. Preserve executable bits, symlinks, and other required filesystem semantics when the payload depends on them; if the outer artifact/container may normalize that metadata, wrap the payload in a format that preserves it and checksum the inner payload.

After supply, return to the sandbox work container for editing, building, testing, and debugging whenever possible.

## Exact transport mission

Transport may bring exact source into the sandbox or publish an exact sandbox result. Keep the sandbox as the engineering loop unless degraded remote mode is required.

### Select the representation

Choose the simplest reliable exact path using payload semantics, integration limits, round trips, and observed reliability. The following are defaults, not a hierarchy:

- use direct file writes for small, isolated text edits;
- use an exact plain-text Git patch for larger semantic-text changes when text transport is practical;
- prefer a Git object, bundle, archive, artifact, or file reference for opaque, binary, or filesystem-sensitive state.

Treat transport capabilities as directional. A repository read or downloadable artifact does not imply a sandbox-to-runner or upload path; discover the actual capability rather than inventing one. Inspect a failed connected operation before retrying or switching transports.

Do not encode existing bytes as Base64 or another opaque text representation merely to move them through model-visible text. This rule concerns Luna-introduced transport representation, not project content: opaque-looking repository or task data must still be preserved and handled faithfully. Encoding performed inside a tool after the model supplies semantic text or a file/reference is outside this rule.

This constraint ends at the model boundary. Non-model byte channels may use any exact, efficient representation supported by the host in either direction, including Actions-to-sandbox transfers.

Keep payload data separate from workflow control text. When an exact file or object already exists, transport or reference it instead of embedding its contents in workflow YAML, heredocs, or command literals. An artifact is only a carrier; use an inner archive, bundle, or equivalent when modes, symlinks, hidden paths, or other filesystem semantics matter.

### Bring exact source into the sandbox

If ordinary Git access is unavailable and connected repository reads are impractical, a transport mission may export the expected commit or PR head as a Git bundle or archive.

1. verify the immutable source SHA;
2. produce a payload that preserves the required Git/filesystem semantics;
3. record provenance and checksum the payload;
4. upload it with bounded retention;
5. verify checksum and source identity in the sandbox before editing.

Lack of direct GitHub network access is a transport constraint, not by itself a reason for degraded remote execution.

### Publish a verified sandbox result

First materialize the intended result as an explicit Git tree or commit. Do not generate the transport patch from ambient working-tree state.

For example, after deliberately staging exactly the intended result:

```bash
expected_base=<expected-commit-sha>
result_tree=$(git write-tree)
git diff --binary --no-textconv --no-ext-diff \
  "${expected_base}^{tree}" "$result_tree" -- > change.patch
sha256sum change.patch
```

Use a model-visible patch when its changed content is semantic text. `GIT binary patch` is a known opaque case because its body is encoded, but its absence does not by itself make a patch semantic. Prefer a file/object/artifact path for opaque changed content when practical.

When a semantic-text patch must cross a model-visible channel:

1. record the expected base SHA, patch checksum, and expected result tree;
2. transfer the patch as task-owned data, using deterministic chunks only when required by limits;
3. verify the reassembled checksum and confirm remote `HEAD` still equals the expected base;
4. run `git apply --check --index` and `git apply --index`, then verify `git write-tree` equals the expected result tree;
5. publish the clean result and verify the committed tree still equals the expected result tree.

A checksum, result-tree, or publication mismatch is a transport failure until evidence shows otherwise. Preserve the verified sandbox result; do not repair transport drift with ad-hoc text edits or source changes. If no exact non-model route exists for an opaque payload, the same end-to-end verification contract may be used as a last transport fallback.

Use a Git bundle when Git objects or history matter. Use an archive for a complete source or supply payload when history does not matter. Do not reconstruct a substantial verified result from prose when an exact payload exists.

## Degraded remote mode

Enter degraded remote mode only when the sandbox work container itself is unavailable or cannot sustain the requested execution because of a hard platform constraint such as usage, duration, resource, or execution limits. Missing direct GitHub network access, missing downloadable bytes, or an initially absent tool/runtime should first be treated as something transport or supply may restore to the sandbox when practical; those conditions alone do not establish degraded remote mode.

In this mode, continue through a sequence of bounded missions rather than pretending the runner is a persistent interactive workstation:

1. establish or recover the exact durable repository base;
2. dispatch a mission for the next bounded edit/build/test/verification step;
3. persist reusable progress as an exact commit, task branch, patch, bundle, or immutable artifact;
4. inspect the returned logs/results before deciding the next mission;
5. repeat only while the sandbox remains unavailable and the task still benefits from remote execution;
6. return to the sandbox path if it becomes available and doing so is cheaper or clearer.

Tell the user that degraded remote mode was used because the sandbox execution environment was unavailable or insufficient. Report the actual remote checks performed. Do not claim interactive sandbox verification when only Actions verification occurred.

## Diagnose failures before retrying

A failed mission is evidence to inspect, not a prompt to guess.

Before changing source or re-running the mission:

1. inspect the run conclusion and the jobs/steps that actually failed;
2. read the available error output and job logs around the first relevant failure;
3. inspect any produced artifacts, commits, refs, or partial results so a retry does not overwrite useful state;
4. distinguish at least these classes when possible: repository/test failure, mission/workflow defect, permission/authentication failure, quota/platform limit, stale source identity, and transient runner/service failure;
5. state uncertainty explicitly when logs or results are unavailable.

Do not modify application source merely because an Actions run is red. Do not re-run an unchanged failed mission unless the evidence supports a transient or flaky failure. Without new evidence, one unchanged retry is the maximum; another identical failure should trigger diagnosis, a changed mission, a different transport, or an explicit blocker report.

Keep a failed mission's logs and task-owned state while they still have debugging or recovery value.

## Task ownership and collision-resistant names

Temporary remote state must be task-owned and bounded. Give independent missions distinct names when they can overlap. Prefer a short readable purpose plus a collision-resistant suffix, for example:

```text
mission-deps-a7f3c2d1
mission/patch-a7f3c2d1
mission-export-a7f3c2d1.yml
```

When a sandbox with Python is available, a cheap preferred attempt is:

```bash
python -c "import secrets; print(secrets.token_hex(4))"
```

If Python or randomness is unavailable, use another reasonable UUID/random mechanism or a sufficiently unique task-derived suffix. Suffix generation is a collision-reduction aid, not a reason to block the task.

Names coordinate ownership; immutable identity still comes from commit SHAs and payload checksums. Keep unrelated tasks out of shared scratch branches, artifact names, workflow payloads, and mutable transport files.

## Durable lifecycle and cleanup

Cleanup must remain safe even if the chat, sandbox, or conversational context disappears unexpectedly. Do not rely on conversation memory as the only record of remote-state ownership.

A task branch or other mission-owned object should remain while it still has active publication, PR review, debugging, handoff, or recovery value. After merge or deliberate abandonment, remove it when task ownership and terminal state are clear. Do not delete an unfamiliar branch or mission object merely because it is old, and do not use ancestry alone as proof that cleanup is safe.

After successful transfer, publication, deliberate abandonment, or replacement, inspect the task-owned temporary state:

```text
temporary branch or ref
mission workflow definition
transport or supply artifact
mission-only repository file
workflow run retained for diagnostics
```

Artifacts should be as small and short-lived as practical, but do not remove the only exact recovery payload before its result has been consumed or replaced by durable repository state. Workflow runs may retain useful diagnostics; bounded growth matters more than an exact run count. Treat workflow definitions, historical runs, branches/refs, and artifacts as separate lifecycle objects.

Control growth before it becomes a cleanup emergency. Prefer one task branch over a new branch for every retry when the same branch can safely carry the durable task state. Avoid duplicate transport/supply artifacts when an existing artifact is still the intended exact payload; when a newer durable result supersedes an older temporary payload, shorten retention or remove the obsolete copy when safe. During mission-heavy work, periodically inspect the count, size, age, and ownership of task branches, temporary workflows, recent runs, and artifacts. If growth is surprising or ownership is unclear, stop creating more temporary state until the existing state is understood. Respect repository/organization retention, storage, quota, and budget controls when they are observable.

Keep a failed mission while it has debugging or recovery value. When a better durable path supersedes it, remove its task-owned temporary objects when safe. During mission-heavy work, occasionally audit task branches/refs, temporary workflow definitions, recent mission runs, and artifact storage so remote state tracks active work rather than forgotten attempts.

If context is lost, reconstruct ownership and terminal state from durable GitHub evidence before cleanup. Preserve anything unfamiliar until that reconstruction is sufficient.

Prefer existing trusted reusable workflows when they express the mission safely. If a temporary workflow is necessary, use narrow triggers, minimum permissions, task-owned names, isolated temporary state, and remove the definition from final source unless the project deliberately adopts it as maintained infrastructure. If concurrency controls are used, derive their group from task identity so unrelated missions cannot cancel or overwrite one another.

If the connected integration can verify ownership and terminal state but cannot delete or otherwise retire a task-owned remote object, a small cleanup mission may use the repository's available GitHub CLI/API capabilities with minimum permissions to perform that operation. Bind destructive operations to exact identities where possible, re-check mutable refs immediately before deletion, and keep unfamiliar state untouched. Using Actions for bounded cleanup/control does not imply degraded remote mode.

Cleanup should be idempotent: an object that is already absent is already clean.

## Security for Luna-created missions

Luna is responsible for the workflow definitions, payloads, artifacts, logs, and other remote state it creates or modifies for a mission. It should behave as though that temporary state may later be visible more broadly, rather than relying on the repository's current visibility. This is a boundary on Luna's own behavior, not a replacement for the project's security policy.

- Do not place secret values in Luna-authored workflow text, mission payloads, logs, artifacts, caches, patches, bundles, or other Luna-created durable output.
- If a Luna mission genuinely needs a credential, use the host's approved secret mechanism with the smallest practical scope and workflow permissions.
- Do not execute untrusted code or artifacts in a Luna-created job that has secrets or a write-capable token.
- Treat attacker-controlled issue/PR text, refs, labels, commit metadata, and workflow inputs as data in Luna-authored privileged jobs; do not interpolate them directly into executable shell or generated code.
- Verify the provenance of downloaded executables and native inputs; an Actions artifact is transport, not automatic trust.
- Do not expose or weaken the user's host computer to avoid using a mission.

If Luna has reason to believe a credential it handled was exposed, stop using it for the mission and report the exposure. Do not attempt broader project credential remediation unless the user or project instructions authorize it.
