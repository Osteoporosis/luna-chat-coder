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

Transport is bidirectional. It may bring exact repository source into the sandbox or carry an already-verified sandbox change back to durable GitHub state. It is not reserved for complete API failure, and it should not force the engineering loop itself into Actions.

Prefer a byte-preserving transport when exact bytes already exist and reconstructing them through model-authored per-file/blob content would add meaningful serialization, partial-update, or round-trip risk. Small intentional textual edits can still use direct file operations when that is simpler and sufficiently reliable. Avoid rigid file-count thresholds; choose based on payload fidelity, semantics, and observed integration behavior.

Treat the workflow definition as control-plane text, not as the transport payload. Do not embed a substantial source tree, patch, archive, or generated replacement content inside model-authored workflow YAML, shell heredocs, command literals, or large textual workflow inputs and then describe that path as byte-preserving transport. Prefer the mission to retrieve an already-existing exact artifact/archive/patch/bundle/commit or a host-provided file reference, and verify its checksum before use. If the host can transfer a sandbox file to the mission byte-for-byte, use that capability; if it cannot, choose another reliable exact path rather than disguising model reserialization as an Actions transport.

Do not assume artifacts form a symmetric sandbox↔runner channel. A workflow can produce an artifact for later download, but sending an existing sandbox payload into a mission requires an actual host-provided file-transfer path, prior durable Git/Actions state, or another exact mechanism. Discover that capability instead of inventing it.

An Actions artifact is a carrier, not by itself a fidelity guarantee. When exact source or another payload depends on hidden paths, executable modes, symlinks, case sensitivity, or similar filesystem semantics, package an inner archive, Git bundle, or other format that preserves the required state and checksum that inner payload. Do not upload a raw directory with default artifact behavior and call the wrapper exact; for example, a complete Luna source payload must not silently omit `.agents/`.

### Bringing exact source into the sandbox

If the sandbox cannot reach GitHub directly and connected repository reads are impractical for the repository size or shape, a transport mission may export the expected commit or PR-head as an archive, Git bundle, or other deterministic artifact. The mission should:

1. resolve and verify the expected immutable source SHA;
2. check out exactly that state;
3. produce the smallest suitable byte-preserving inner payload, using a Git bundle when history/objects matter or an archive that includes hidden paths and preserves required filesystem semantics when a complete source tree is sufficient;
4. record source SHA, production command, relevant platform/tool versions, and a checksum;
5. upload the payload with bounded retention;
6. let the sandbox verify checksum and source/tree identity before editing.

After transfer, return to the sandbox for normal source inspection, editing, build, test, and debugging. Lack of direct GitHub network access is a transport constraint, not by itself a reason for degraded remote execution.

### Carrying verified changes out of the sandbox

A patch or bundle can be better than repeated independent writes when:

- a verified change spans enough state that complete-file writes create unnecessary round trips or partial-update risk;
- connected write operations return repeated or structurally relevant errors after those errors have been inspected;
- payload or operation limits make direct publication brittle;
- binary changes, renames, executable-bit/mode changes, or Git object/history semantics should be preserved exactly;
- a single checksummed payload materially simplifies recovery or handoff.

Do not switch transports merely because one API call failed. Inspect the returned error first. Retry an unchanged operation only when the evidence supports a transient failure and a retry is safe. If the same path remains unreliable, switch deliberately rather than repeating it blindly.

A Git patch can be an exact change payload for Git-tracked content and tracked modes when it is generated between two explicit Git tree states. Do not use the bare ambient `git diff --binary` form as the exact-transport recipe: without explicit tree arguments it represents working-tree versus index state, can omit staged or untracked intended changes, and `git diff` may invoke repository-configured text conversion that is unsuitable for an applyable patch. First materialize and verify the intended result as a Git tree or commit, then diff the expected base tree against that result with text conversion and external diff helpers disabled.

For example, after deliberately staging exactly the intended result:

```bash
expected_base=<expected-commit-sha>
result_tree=$(git write-tree)
git diff --binary --no-textconv --no-ext-diff \
  "${expected_base}^{tree}" "$result_tree" -- > change.patch
sha256sum change.patch
```

The staged/index state above is only a way to materialize the explicit result tree; unrelated ambient staged, unstaged, or untracked state is not part of the transport contract. Preserve the expected `result_tree` identity with the payload. A result commit may be used instead when that is the simpler durable state.

Bind the payload to the expected base SHA and result tree. From a clean checkout of the expected base, the remote side should:

```text
verify remote HEAD == expected base SHA
verify patch checksum
git apply --check --index change.patch
git apply --index change.patch
verify git write-tree == expected result tree
git diff --cached --check
run only mission-local integrity/publication checks required by the transport contract
commit the staged result to the task branch
verify committed tree == expected result tree
```

Substantive repository edit/build/test/debug iteration remains sandbox-owned unless degraded remote mode is active. A transport mission may perform bounded checks that validate transport or publication output without becoming the repository engineering loop.

Use a Git bundle when preserving Git objects or history is more useful than a patch. A complete source archive is also natural for inbound source or supply payloads. For ordinary repository changes, do not default to blindly extracting a partial archive over the working tree: without an explicit manifest it can underspecify deletions, renames, modes, symlinks, or path safety. If a file-set archive is intentionally the best payload, stage it, validate its path set/checksums and any required replacement/deletion semantics, then apply it deliberately.

Before choosing artifact transport, consider current payload-size, upload/download, and retention limits exposed by the integration or platform. Do not recreate a substantial change from prose or ad-hoc string replacements when an exact payload exists.

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

## Security

- Keep credentials, signing material, and unrelated runner state out of artifacts.
- Use minimum workflow permissions.
- Verify the provenance of downloaded executables and native inputs; an Actions artifact is transport, not automatic trust.
- Keep large caches, SDKs, and toolchains out of normal source history unless the repository explicitly adopts them there.
- Do not expose or weaken the user's host computer to avoid using a mission.
