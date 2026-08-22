# Luna Chat Coder Design Rationale

This document is durable maintainer memory for Luna Chat Coder. It is intentionally stored inside the skill directory so that the reasoning survives repository-template copies, history resets, repository migration, and replacement of top-level README or AGENTS files.

It is **not runtime policy**. Normal repository-development tasks should follow `../SKILL.md` and read the operational references only when needed. Read this document when modifying, reviewing, simplifying, porting, or redesigning Luna itself.

If this rationale and the current skill disagree, investigate and deliberately reconcile them. Do not let historical explanation silently override current executable policy.

## 1. Problem statement

Ordinary web chat can be a useful software-development surface, but its execution environment and continuity properties differ from a persistent developer workstation.

The recurring problems Luna is intended to handle are:

- the chat sandbox may be disposable, reset, replaced, resource-limited, duration-limited, or temporarily unavailable;
- the sandbox may have useful runtimes and tools already installed, but may lack a repository-specific dependency, SDK, compiler, service, browser, native library, generated input, or external network access;
- the user’s own computer is normally outside the web chat execution boundary and should not become a required escape hatch;
- exact repository source cannot safely be reconstructed from conversational prose after context loss;
- connected GitHub APIs can be efficient for some writes but awkward, limited, or occasionally unreliable for large or complex changes;
- GitHub Actions can supply networked execution and durable outputs, but remote runs, artifacts, workflows, storage, and cleanup have latency, quota, management, and sometimes monetary cost;
- several chats, AI agents, CI jobs, or humans may work on the same repository concurrently;
- chat history, repository history, or the original template repository may later disappear, so critical design intent must not exist only in conversation or commit archaeology;
- models sometimes react badly to failed Actions runs: guessing from a red status, changing unrelated source without reading logs, or blindly re-running the same failing workflow.

Luna exists to make this constrained environment reliable without replacing ordinary chat with another autonomous coding system.

## 2. North star

The shortest durable model is:

```text
Chat
    intent and interaction

Sandbox work container
    primary disposable development workstation

GitHub
    exact durable repository state

GitHub Actions mission
    bounded remote capability, transport, or execution when the normal path is insufficient
```

The user should normally experience this as “give the chat a repository and a development task.” Luna should be almost invisible on the healthy path.

The design principle is:

> **Discover early, activate late.**

The model should know Luna exists before repository work begins, but loading Luna must not itself trigger Actions, remote state, or extra ceremony.

## 3. What Luna is and is not

Luna is a **continuity, capability-fallback, exact-transport, and evidence policy** embedded in a repository template.

It is not:

- a separate autonomous coding agent;
- a new application architecture or engineering methodology;
- a framework that chooses the project’s database, test framework, runtime, browser tooling, or compiler;
- a reason to move ordinary coding into GitHub Actions;
- a promise that every Agent Skills host has the same GitHub capabilities;
- a mechanism that depends on direct access to the user’s computer;
- a substitute for project-specific engineering instructions.

The repository itself defines what technologies and verification are required. Luna only helps the current chat environment satisfy those requirements faithfully.

## 4. Why the sandbox is primary

The sandbox work container should be treated as a disposable development workstation, not merely a temporary text editor.

Reasons to prefer it:

- it is already attached to the conversation;
- it avoids remote-run startup and transport round trips;
- the model can inspect the exact working tree and command output directly;
- useful runtimes and tools may already exist, so inventory can be cheaper than reacquisition;
- iterative edit/build/test/debug loops are much more natural in a local execution context than across one-shot remote jobs;
- using an already-available sandbox avoids unnecessary Actions minutes, artifacts, workflows, storage, cleanup, and possible cost.

The policy therefore says **inventory before acquiring**. This is not a guarantee that any particular tool is preinstalled. It is a rule to inspect what exists before downloading, installing, or dispatching a remote mission.

## 5. Why the user’s host computer is outside the model

The user’s local computer may contain the best development environment, but an ordinary web chat generally should not assume direct host access.

Making host access a dependency would weaken portability and can create pressure to reduce isolation merely to unblock development. Luna instead tries to compose capabilities from the sandbox, GitHub durable state, and bounded remote execution.

This is a design boundary, not a claim that every future product surface can never access a user-controlled machine.

## 6. GitHub as durable truth

Conversation is good for intent, decisions, and explanation. It is not the authoritative source of exact repository bytes.

The recovery order is deliberately:

```text
commit / PR head
    > immutable Git or Actions artifact
    > surviving sandbox working tree
    > conversation reconstruction
```

Branch and tag names are coordination names, not immutable identity. Important publication and transport operations should bind to a resolved commit SHA.

Observed repository facts must remain distinct from material assumptions. If source, history, documentation, and conversation disagree, investigate rather than silently choosing whichever is convenient.

## 7. Concurrency is normal, not exceptional

Luna assumes that another actor may change remote state at any time:

- another web chat may work on the repository;
- another AI coding agent may create or update a branch;
- CI may create runs, artifacts, or generated state;
- a human may push, rebase, merge, rename, delete, or create branches and workflows;
- organization automation may alter permissions, retention, or branch state.

Therefore:

- resolve mutable names to current immutable identity before consequential writes, patch application, publication, or cleanup;
- do not assume a branch is “ours” because its name looks familiar;
- do not infer ownership from age or ancestry alone;
- preserve unfamiliar state until ownership is understood;
- use task-owned branch/ref/artifact namespaces when independent work can overlap;
- use immutable SHAs and payload checksums for identity, while names only coordinate ownership.

This concurrency rule belongs in core policy, not only in recovery documentation, because it should prevent conflicts before they happen.

## 8. Why Actions is modeled as a mission, not a bridge

Earlier terminology described GitHub Actions as a “bridge.” That metaphor suggested an always-available, bidirectional connection and encouraged conceptual sprawl: source bridge, capability bridge, build-input bridge, and similar terms.

The intended behavior is closer to an unmanned probe:

1. give it exact source identity and bounded inputs;
2. define a narrow purpose;
3. let it execute independently;
4. inspect logs, checks, artifacts, commits, or other durable results after it terminates;
5. clean up task-owned temporary state after its value ends.

The canonical term is therefore **Actions mission**.

Mission roles may include supply, exact transport, degraded remote execution, and occasional task-owned remote control such as cleanup, but these are roles of the same mechanism rather than a rigid taxonomy. Transport is intentionally bidirectional: a mission may bring exact source or required bytes into the sandbox, or carry verified bytes back to durable GitHub state, without moving the engineering loop itself out of the sandbox. Commands used inside supply or transport missions to acquire/build/package a payload, apply it, or verify its integrity/output are mission mechanics rather than degraded remote mode so long as Actions is not substituting for the sandbox repository engineering loop.

## 9. Supply missions

A supply mission is appropriate when the sandbox can perform the engineering work but cannot obtain a required external input.

Examples include a dependency/package cache, runtime, SDK, compiler, executable or application distribution, installer, native library, browser payload, generated data, vendor tree/archive, or other repository-required input.

Important properties:

- acquire only what the task requires;
- derive versions and requirements from repository declarations where possible;
- identify the sandbox target platform/ABI when native compatibility matters;
- record provenance, repository SHA, sandbox target, runner platform/architecture, relevant tool versions, and production commands;
- checksum the returned payload;
- treat runner-native or compiled outputs as platform-specific unless compatibility with the sandbox target is established;
- allow project-appropriate caches, package sets, vendor trees, portable install trees, or installers to be materialized and consumed offline in the sandbox without implying they belong in source control, while preserving required filesystem semantics inside the transported payload rather than trusting an outer artifact container to do so;
- return to the sandbox for the normal engineering loop after supply.

Luna should not turn this into a general-purpose environment-management methodology.

## 10. Exact transport is a first-class, bidirectional option

Transport must not be treated only as publication or as a punishment used after every connected GitHub mechanism has failed. A sandbox that cannot reach GitHub directly may still be perfectly capable of editing, building, testing, and debugging once exact source arrives. When repository size or payload shape makes per-file reconstruction materially riskier or more expensive, a bounded mission that exports an exact checkout, archive, Git bundle, or equivalent artifact can therefore be the correct source-acquisition path while leaving the engineering loop in the sandbox.

The same principle applies in the other direction after verification. Connected file operations, native Git object operations, archives/artifacts, patches, and bundles are alternative transports. The correct choice depends on the observed payload and integration, not on a fixed hierarchy.

A byte-preserving transport is especially attractive when exact bytes already exist and model-mediated serialization adds no semantic value. Re-emitting a large source tree or verified multi-file change through model-authored complete-file/blob content creates extra opportunities for byte drift, partial updates, and round trips. Small intentional textual edits are different: direct file operations can remain the simplest reliable path. Luna should not encode a rigid file-count threshold because payload semantics, connector capabilities, and future model/tool fidelity can change.

The mission definition and the transported bytes are different layers. Putting a substantial patch or source payload into model-authored workflow YAML, heredocs, command literals, or large textual workflow inputs does not become more exact merely because Actions executes it. The fidelity benefit comes from carrying an already-existing exact file/object/artifact reference byte-for-byte and verifying identity at the receiving side. Artifact transport is not assumed to be symmetric: a workflow-produced artifact can be downloaded later, while sandbox-to-runner transfer requires an actual host capability or prior durable state. When that channel does not exist in a given direction, Luna should choose another exact mechanism rather than using the workflow as a text-shaped disguise for model serialization. An artifact wrapper is also not itself a fidelity guarantee: complete source or other filesystem-sensitive payloads should use an inner archive, Git bundle, or equivalent format that preserves hidden paths and required mode/symlink/case semantics, with the inner payload checksummed.

Git patches are a particularly useful example, but the exact boundary is explicit Git object state rather than an ambient working tree. First materialize and verify the intended result as a Git tree or commit, then generate a binary patch between the expected base tree and that result tree with text conversion/external diff helpers disabled. This makes staged-versus-unstaged state irrelevant to the transport itself, includes intended new files once they are deliberately represented in the result tree, and permits the receiving side to verify that patch application recreates the expected result tree. A bare no-argument `git diff --binary` is therefore not the canonical exact-transport recipe. Archives remain natural for complete source snapshots and supply payloads. A partial archive overlay is more situational because deletes, renames, modes, symlinks, and safe path replacement need an explicit contract; it should not displace patch/bundle merely for convenience.

A deterministic archive, patch, or bundle may be better when:

- direct Git/network access is unavailable but the sandbox can otherwise do the work;
- many files or a large exact payload make repeated complete-file operations inefficient or fragile;
- binary changes, renames, executable bits, mode changes, or Git object/history semantics matter;
- the connector has payload or operation limits;
- API errors persist after the returned errors have been inspected;
- one checksummed payload makes recovery or handoff materially simpler.

Conversely, a small textual change should not incur a remote mission merely because exact transport exists.

The choice should minimize overhead while preserving exactness and reliability. For transported payloads, source/base identity, checksums, post-transfer identity checks, and repository-defined verification are important integrity boundaries.

## 11. Mission results must be verified; failures must be diagnosed

A mission's reported conclusion is not the whole result. Even a green run can produce the wrong artifact, commit, ref, checksum, source identity, or cleanup effect if the mission contract or workflow is defective. Luna therefore verifies the outputs that matter before consuming a mission result or making the next consequential decision.

A separate repeated failure mode in web-chat development is reacting to a failed remote operation without examining evidence.

Bad patterns include:

- editing application source just because a workflow is red;
- assuming a dependency or code defect without reading logs;
- re-running an unchanged workflow repeatedly;
- losing useful partial artifacts or commits before inspecting them;
- treating permission, quota, stale SHA, workflow syntax, and product-test failures as the same class of problem.

Luna therefore requires output-contract verification for successful missions and diagnosis before retry or source modification for failed missions.

When possible, distinguish:

- repository/application/test failure;
- mission/workflow defect;
- permission or authentication failure;
- quota, storage, usage, duration, or platform limit;
- stale source identity/base drift;
- transient runner or service failure.

An unchanged retry is reasonable only when evidence supports a transient or flaky failure. Without new evidence, one unchanged retry is the default ceiling; another identical failure should change the diagnosis or path rather than create an infinite loop.

If logs are unavailable, say so and preserve uncertainty.

## 12. Degraded remote mode

If the sandbox itself is unavailable or cannot sustain the required repository engineering loop because of a hard platform constraint, Luna may temporarily continue that loop through bounded Actions missions. Supply/transport missions may still execute their own bounded acquisition, packaging, application, and output-verification mechanics while the sandbox is healthy; degraded mode begins when Actions substitutes for the sandbox engineering loop rather than merely servicing it. Missing direct GitHub network access, missing downloadable bytes, or an initially absent runtime/tool is not by itself evidence that the sandbox cannot sustain engineering work; when practical, transport or supply should first restore the sandbox path.

This is called **degraded remote mode** rather than “moving development to Actions.” The distinction matters:

- a runner is ephemeral and non-interactive from chat’s perspective;
- work should be split into bounded edit/build/test/verification steps;
- reusable progress should be persisted in commits, task branches, exact patches/bundles, or immutable artifacts;
- logs/results should be inspected before deciding the next mission;
- the assistant should tell the user that sandbox execution was unavailable or insufficient and that remote execution was used;
- if the sandbox becomes usable again, returning to it is preferred when practical.

The user-visible notice matters because the verification environment changed.

## 13. Durable handoff and context loss

Chat, sandbox, or conversational context can disappear unexpectedly. Important state must therefore survive independently of conversation memory.

Persist state when losing it would make recovery expensive or ambiguous. Appropriate durable carriers include:

```text
branch / PR / issue / commit / task-owned artifact
```

Cheap reasoning and easily reconstructed intermediate notes can remain in chat.

A failed attempt that leaves a useful diagnosis, exact payload, environment artifact, commit, or logs may still be worth preserving. A failed attempt with nothing reusable can be cleaned up and restarted from the last known durable base.

## 14. Remote-state growth must be actively bounded

Cleanup is not only aesthetic. Unbounded branches, workflows, runs, and artifacts can:

- make the repository difficult for humans to understand;
- obscure ownership and active work;
- consume Actions artifact storage or retention quota;
- increase management and recovery complexity;
- in some GitHub plans or configurations, contribute to billable usage or storage.

The design therefore has two layers of control.

### Before creation

- reuse a task branch when the same durable task state can safely continue there;
- avoid duplicate artifacts when an existing artifact is still the intended exact payload;
- prefer the smallest mission and output that satisfies the task;
- use bounded retention for temporary payloads;
- inspect remote-state growth during mission-heavy work;
- stop creating more temporary state when growth or ownership becomes surprising.

### After value ends

- remove task-owned temporary branches/refs, workflow definitions, mission-only files, obsolete artifacts, and other temporary state when ownership and terminal state are clear;
- retain runs and artifacts while they still have debugging, review, handoff, or recovery value;
- never delete the only recovery payload before its result has been consumed or replaced by stronger durable repository state;
- make cleanup idempotent;
- if context was lost, reconstruct ownership and terminal state from durable GitHub evidence before deleting unfamiliar objects.

Cleanup must be recovery-aware rather than aggressively eager. When the connected integration can establish ownership and terminal state but lacks a direct deletion/control operation, a bounded Actions cleanup mission is a valid fallback with minimum permissions and exact identity checks. That is remote control, not degraded remote execution.

## 15. Task-owned naming and collision reduction

Readable task-owned names help humans and agents distinguish temporary state. A short purpose plus a random suffix is preferable when work can overlap.

When Python is available, a convenient default is:

```bash
python -c "import secrets; print(secrets.token_hex(4))"
```

Four random bytes produce eight hex characters. The suffix is long enough to make accidental collisions unlikely for the intended scale while remaining readable in branch, artifact, and workflow names.

The suffix is only a collision-reduction aid. If Python or randomness is unavailable, another reasonable UUID/random mechanism or sufficiently unique task-derived suffix is acceptable. Mission execution must not block merely because the preferred suffix generator is unavailable.

Names are never immutable identity; SHAs and checksums remain authoritative.

## 16. Discovery and the role of AGENTS.md

The template uses `AGENTS.md` as a cheap, prominent discovery router into the embedded skill.

That file should stay small. It should point to the skill and preserve project-specific engineering instructions rather than duplicating the entire Luna protocol.

However, `AGENTS.md` should **not be treated as a hard runtime dependency**. A downstream repository may later replace its README and AGENTS files with project-specific versions. If the host supports repository-local Agent Skill discovery and `.agents/skills/luna-chat-coder/` remains present, the skill should still be usable.

Because automatic repo-local skill discovery is host-dependent, the robust recommendation is:

- keep the skill directory intact;
- merge the short Luna entry-point instruction into the project’s own `AGENTS.md` when practical;
- do not require downstream projects to preserve Luna’s original top-level README or AGENTS contents verbatim.

This is **best-effort discovery resilience**, not a promise that every host will find an unreferenced repo-local skill automatically.

## 17. Why maintainer rationale lives inside the skill directory

The original development conversation, private history, or pre-publication repository may not survive the public release. Requiring future maintainers or models to read every historical commit would also be inefficient and brittle.

Therefore the important reasoning is stored here, next to the skill, but outside runtime policy.

The separation is intentional:

```text
SKILL.md
    current executable policy

references/actions-missions.md
references/recovery.md
    operational details loaded when needed

references/design-rationale.md
    maintainer memory loaded only when Luna itself is being changed
```

This document should be self-contained enough that a fresh model can understand why the current policy looks the way it does without access to prior chat transcripts, deleted repositories, or old PRs.

It may be longer than runtime documentation. Losing critical rationale is more harmful than paying a few extra tokens during the relatively rare task of maintaining Luna itself.

## 18. ChatGPT Web integration boundary

The fully specified integration path documented by this repository is ChatGPT Web with two distinct GitHub layers:

1. **GitHub Plugin** — ChatGPT-side workflow/tool capability.
2. **ChatGPT Codex Connector GitHub App** — GitHub-side installation and target-repository authorization.

They are separate prerequisites for the validated path and should not be casually collapsed into one concept.

This repository keeps the core skill host-neutral because Agent Skills can be portable, and other web AI products may also provide sandboxed execution. But format compatibility does not imply equal capabilities. A host must not invent repository writes, Actions control, logs, artifacts, or credentials it does not actually expose.

## 19. Why README is user-first

Most users will not read a long conceptual explanation before trying the template.

The README therefore front-loads:

- `Use this template`;
- GitHub Plugin installation/connection;
- ChatGPT Codex Connector installation/authorization;
- the final normal interaction: give ChatGPT the repository URL and development task.

The design model is intentionally later and optional. A user should not need to understand the probe metaphor, recovery order, or transport strategy to benefit from Luna.

## 20. Terminology decisions

Canonical runtime terminology is deliberately narrow:

- `sandbox work container`
- `durable repository state`
- `Actions mission`
- `degraded remote mode`

Avoid `local container` because users often interpret “local” as their own computer.

Avoid `bridge` because it implies a persistent communication path rather than bounded remote execution.

In human-facing Korean documentation, transliterating common engineering terms such as 런타임, 샌드박스, 리포지토리, 프로젝트 is often clearer than overly literal translation. Workflow-specific terms such as GitHub Actions, Actions mission, edit/build/test, diff, fallback, commit, PR, and artifact may remain in English when translation would become longer or ambiguous.

## 21. Stable decisions worth preserving

Unless new evidence provides a strong reason to change them, these are intentionally stable:

- ordinary chat remains the development surface;
- sandbox-first execution;
- GitHub exact state outranks conversation reconstruction;
- user host access is not a dependency;
- project requirements define the engineering method;
- Actions is bounded fallback/transport/execution rather than the default workstation;
- exact byte-preserving transport is a first-class, bidirectional option rather than merely a publication fallback;
- exact state should avoid unnecessary model-mediated reserialization when that path would add meaningful fidelity or partial-update risk and a practical byte-preserving alternative exists;
- every mission result must be checked against its expected outputs, and remote failure must be diagnosed from evidence before blind retry or source modification;
- concurrent actors are assumed;
- temporary remote state is task-owned, bounded, recovery-aware, and growth-controlled, with bounded cleanup missions available when direct control is insufficient;
- completion claims are evidence-bounded;
- core skill remains host-neutral while ChatGPT Web is the fully specified integration path;
- runtime policy and maintainer rationale remain separate;
- top-level discovery files may evolve downstream, so the skill directory should carry its own durable rationale.

## 22. Rejected or corrected approaches

### A rigid publication hierarchy

Rejected: always prefer file writes, then native Git objects, then patch mission only as a final fallback.

Reason: large multi-file changes, binary/mode/history semantics, transport limits, and persistent API instability can make one exact patch/bundle objectively safer or cheaper earlier.

### A rigid gap taxonomy as the core decision model

Rejected.

Reason: acquisition, transport, control, and execution needs can overlap in one task, and host capabilities will evolve. Requiring every situation to fit a fixed category can create new false boundaries without improving the durable decision. The more stable rule is that the normal engineering loop stays in a usable sandbox; bounded missions may supply, transport, or control remote state as needed; remote edit/build/test execution is reserved for the case where the sandbox itself cannot faithfully sustain it.

### Model-mediated per-file reconstruction as the default exact transport

Rejected when exact state already exists, model-mediated reconstruction adds meaningful fidelity or partial-update risk, and a practical byte-preserving path exists.

Reason: if the source tree or verified payload already exists as exact bytes, routing it through model-authored complete-file/blob content adds serialization and partial-update risk without adding useful reasoning. This is a preference, not a permanent ban: small intentional text edits and constrained hosts may still make direct content operations the best path.

### Embedding the payload in workflow text and calling it exact transport

Rejected for substantial exact payloads when a practical byte-preserving channel exists.

Reason: model-authored YAML, heredocs, command literals, or large textual workflow inputs still pass the payload through model serialization. Actions can execute that text, but it does not recover the fidelity benefit of carrying an already-existing exact patch/archive/bundle/artifact/file reference.

### Actions as the normal coding environment

Rejected.

Reason: unnecessary remote latency, metered execution, artifact/workflow management, weaker interactive feedback, and remote-state growth.

### Bridge terminology

Rejected in canonical policy.

Reason: encouraged an inaccurate persistent-connection mental model and too many overlapping sub-concepts.

### Reconstructing exact changes from prose

Rejected whenever exact source bytes, patch, bundle, commit, or artifact are available.

Reason: silently loses fidelity and makes recovery ambiguous.

### Blind Actions retries

Rejected.

Reason: hides root causes, wastes compute, can amplify bad source modifications, and creates redundant runs/artifacts.

### Aggressive immediate cleanup

Rejected.

Reason: a failed run or artifact may be the only useful diagnosis or recovery payload if the chat or sandbox disappears.

### Keeping all rationale inside SKILL.md

Rejected.

Reason: runtime policy should stay concise and unambiguous. Historical reasoning belongs in maintainer-only progressive-disclosure documentation.

### Depending on Git history as maintainer memory

Rejected.

Reason: public release may reset history, repositories can migrate, and asking a fresh model to inspect every commit is inefficient and non-portable.

## 23. Questions future maintainers should ask before adding rules

Before expanding Luna, ask:

1. Does this rule prevent a repeatable failure in constrained chat-based repository development?
2. Is it a Luna concern, or should the project’s own engineering instructions decide it?
3. Can the rule be an operational reference rather than always-loaded runtime policy?
4. Does it preserve exact state and observable evidence?
5. Does it remain safe with concurrent chats, agents, CI, and humans?
6. What happens if the chat and sandbox disappear immediately after this step?
7. Can it create unbounded remote state, quota, storage, or cost?
8. Does it assume a host capability that may not exist?
9. Will a normal user need to understand this concept, or can Luna keep it invisible?
10. Does this rule encode today's connector/tool surface too literally, or will its invariant still make sense as hosts and models improve?
11. Is there a simpler rule that preserves the same reliability?

## 24. Maintenance discipline

When Luna itself changes:

- update `SKILL.md` only for current runtime policy;
- update operational references when procedure changes;
- update this document when a stable rationale, rejected alternative, known failure mode, or important boundary changes;
- keep README English/Korean structure aligned when user-facing behavior changes;
- do not require historical Git or old conversations to understand the current design;
- prefer deleting obsolete concepts over carrying synonyms indefinitely;
- keep versioning deliberate; pre-publication work may remain at `0.1.0` until the first public release is intentionally cut.

The aim is a small runtime protocol with enough durable design memory that future maintainers can simplify or extend it without rediscovering the same failures from scratch.
