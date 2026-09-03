# Luna Chat Coder Design Rationale

This document is durable maintainer memory for Luna Chat Coder. It lives inside the skill directory so the reasoning survives template copies, repository migration, history resets, and replacement of top-level documentation.

It is **not runtime policy**. Normal repository work follows `../SKILL.md` and reads operational references only when needed. Read this document when modifying, reviewing, simplifying, porting, or redesigning Luna itself.

If this rationale and the current skill disagree, investigate and reconcile them deliberately. Historical explanation must not silently override current runtime policy.

## 1. Problem statement

Ordinary web chat can be a useful software-development surface, but its execution environment and continuity properties differ from a persistent developer workstation.

Luna exists because several failures recur in that environment:

- the chat sandbox may reset, disappear, hit resource limits, or lack direct network access;
- repository-required dependencies, runtimes, services, browsers, compilers, or generated inputs may not initially be available;
- the user's own computer should not become a required escape hatch;
- exact source cannot safely be reconstructed from conversational prose after context loss;
- connected GitHub operations are useful but can be awkward or unreliable for some payloads;
- GitHub Actions can provide networked execution and durable transport, but adds latency, quota, lifecycle, and sometimes monetary cost;
- multiple chats, agents, CI jobs, or humans may change remote state concurrently;
- Luna-created temporary workflows, payloads, logs, and artifacts may outlive the moment that created them, so their safety must not depend on a repository remaining private;
- models sometimes react badly to failed remote operations by guessing, editing unrelated source, or blindly retrying.

Luna's job is to make these constraints manageable without turning ordinary chat into another autonomous coding system or making the user operate extra infrastructure.

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

The normal user experience should still be: **give the chat a repository and a development task**.

The design principle is:

> **Discover early, activate late.**

The model should know Luna exists before repository work begins, but loading Luna must not itself trigger Actions, remote state, or visible ceremony.

## 3. What Luna is and is not

Luna is a continuity, capability-fallback, exact-transport, and evidence policy embedded in a repository template.

It is not:

- a separate autonomous coding agent;
- a framework that chooses a project's architecture, database, runtime, compiler, browser stack, or test framework;
- a reason to move ordinary coding into GitHub Actions;
- a promise that every Agent Skills host exposes the same capabilities;
- a mechanism that depends on direct access to the user's computer;
- a replacement for project-specific engineering or security instructions.

The repository defines the engineering method. Luna helps the current chat environment carry it out faithfully.

This boundary matters when Luna itself is being developed inside a repository that also uses Luna. Maintainers must distinguish **rules Luna needs for its own behavior** from **opinions about how the surrounding project should be run**. The former belongs here; the latter belongs in project instructions unless it is strictly necessary for Luna to operate safely.

### Canonical maintenance boundary

The canonical `Osteoporosis/luna-chat-coder` repository is both Luna's development repository and the source copied into downstream templates. Luna's normal repository-development policy therefore applies while maintaining the canonical repository itself; `AGENTS.md` intentionally serves the same discovery role in both contexts, and `SKILL.md` routes Luna maintenance to this rationale.

Maintainer guidance must not leak into downstream project policy. In particular, freedom to rewrite or refactor canonical Luna aggressively is a rule for maintaining Luna, not a rule for repositories that merely use the skill.

## 4. Why the sandbox is primary

The sandbox work container should be treated as a disposable development workstation, not merely a temporary text editor.

It is already attached to the conversation, gives the model direct access to the working tree and command output, and supports a natural edit/build/test/debug loop. Using it also avoids unnecessary Actions startup, transport, workflow, artifact, and cleanup overhead.

The policy therefore says **inventory before acquiring**. This does not assume that any particular runtime or tool is preinstalled; it means inspect what is already available before downloading, installing, or dispatching a remote mission.

## 5. Why the user's host computer is outside the model

A user's own computer may be a capable development environment, but ordinary web chat should not require access to it.

Making host access a dependency would weaken portability and recreate the tunnel/local-agent setup that Luna is specifically trying to avoid. Luna instead composes the chat sandbox, GitHub durable state, and bounded remote execution.

This is a design boundary, not a claim about every possible future product surface.

## 6. GitHub as durable truth

Conversation is useful for intent and explanation. It is not authoritative for exact repository bytes.

Recovery therefore prefers:

```text
commit / PR head
    > immutable Git or Actions artifact
    > surviving sandbox working tree
    > conversation reconstruction
```

Branch and tag names are coordination names, not immutable identity. Consequential transport, publication, and cleanup should bind to resolved commits or equivalent immutable identity.

Observed repository facts must remain distinct from assumptions. When source, documentation, history, and conversation disagree, investigate rather than choosing whichever is convenient.

## 7. Concurrency is normal

Another actor may change repository state at any time: another chat, another agent, CI, a human, or organization automation.

Therefore Luna:

- resolves mutable names before consequential writes or cleanup;
- does not infer ownership from branch names, age, or ancestry alone;
- preserves unfamiliar state until ownership is understood;
- uses task-owned namespaces when independent work can overlap;
- uses commit SHAs and payload checksums for identity while names coordinate ownership.

This is core policy because preventing a collision is cheaper than recovering from one.

## 8. Why Actions is modeled as a mission

Earlier terminology described GitHub Actions as a “bridge.” That suggested an always-on connection and encouraged too many overlapping sub-concepts.

The intended model is closer to a bounded probe:

1. give it exact source identity and bounded inputs;
2. define a narrow purpose;
3. let it run independently;
4. inspect durable results after it terminates;
5. remove task-owned temporary state after its value ends.

The canonical term is **Actions mission**.

Supply, transport, degraded execution, and occasional cleanup/control are roles of the same mechanism, not a rigid taxonomy. The important boundary is whether Actions is servicing a sandbox-owned engineering loop or replacing it.

### Luna owns the state it creates

A previous draft generalized mission security into repository-wide security advice. That is outside Luna's role. A project may have its own security model, deployment rules, secret-handling conventions, and trusted workflows; Luna should not overwrite those policies merely because it is present.

Luna is responsible for **Luna-created or Luna-modified mission machinery and temporary state**: temporary workflow definitions, transport payloads, artifacts, logs, caches, and the credentials or permissions it chooses to request. Intended project output remains governed by project policy. Luna's own temporary state must be safe even if repository visibility later changes: do not embed secret values in it, use minimum permissions, and do not execute untrusted code with Luna-provided privileged credentials.

The durable rule is scope, not paternalism: Luna manages its own side effects well and leaves project-owned policy to the project.

## 9. Supply missions

A supply mission is appropriate when the sandbox can do the engineering work but cannot obtain a required external input.

Examples include a dependency/package cache, runtime, SDK, compiler, executable distribution, installer, native library, browser payload, generated data, or vendor archive.

The mission should acquire only what the task requires, derive versions from repository declarations when possible, account for target OS/architecture/ABI when native compatibility matters, record provenance, checksum the returned payload, and return the normal engineering loop to the sandbox.

Runner-native output is not automatically compatible with the sandbox. Filesystem-sensitive payloads should preserve executable bits, symlinks, and other required semantics inside an appropriate inner archive or equivalent format rather than trusting an artifact wrapper to preserve them.

Luna should not turn this into a general-purpose environment-management methodology.

## 10. Exact transport is a first-class option

Transport is bidirectional. Connected file operations, Git objects, archives, artifacts, patches, and bundles are alternatives. Choose the simplest reliable exact path from payload semantics, integration limits, round trips, and observed reliability. Representation-specific defaults guide that choice rather than forming a hierarchy: direct writes fit small isolated text edits, semantic-text patches fit larger textual changes, and byte-preserving paths fit opaque, binary, or filesystem-sensitive state.

Exactness and opacity are separate properties. **Exact bytes** means byte identity is preserved; it does not mean the representation is binary, encoded, or unreadable. A semantic-text patch can be exact. Conversely, an encoded payload can be opaque to the model even when it is exact.

Model-mediated reconstruction is generative rather than byte-preserving. Sampling can select different tokens according to decoding settings such as temperature; generation-time text-watermarking schemes, when present, intentionally bias token selection. Either can change bytes without changing the intended meaning. Exact transport therefore relies on transferred bytes plus integrity checks, not faithful-looking model regeneration.

### Model-visible transport and safety classifiers

Use **safety classifier** for the deployment safeguard discussed here. Both OpenAI and Anthropic use that term; Anthropic also uses **Constitutional Classifiers** for a specific classifier design.

During Luna maintenance, model-visible Base64 and similar opaque transport encodings have been observed to coincide with long safety-classifier waits. Luna treats that observed behavior as operational evidence: do not create opaque transport text when a practical exact alternative exists. This policy does not require knowing a host's internal routing or claiming that every encoded string deterministically triggers a classifier.

This is a transport rule, not a content rule. Opaque-looking repository or task content must still be preserved and used faithfully. Tool-internal encoding that happens after the model boundary is not the target. Non-model byte channels are outside this concern and may use any exact, efficient representation the host supports in either direction, including Actions-to-sandbox transfer.

Safety systems are a durable constraint. Luna should not depend on future hosts relaxing safety review; reduce unnecessary classifier surface instead. Transport capabilities may improve, so exact route selection remains capability-driven.

A Git patch is suitable for model-visible transport when its changed content is semantic text. `GIT binary patch` is a known opaque case because Git encodes its binary body with Base85, but the absence of that marker does not by itself make a patch semantic.

Public references establish terminology and implementation examples; the observed latency above is maintainer evidence:

- OpenAI, `Introducing gpt-oss-safeguard`: <https://openai.com/index/introducing-gpt-oss-safeguard/>
- Anthropic, `Constitutional Classifiers`: <https://www.anthropic.com/news/constitutional-classifiers>
- Git binary-patch Base85 implementation: <https://github.com/git/git/blob/master/base85.c>
- OpenAI API temperature semantics: <https://developers.openai.com/api/reference/cli/resources/responses/methods/create>
- Kirchenbauer et al., `A Watermark for Large Language Models`: <https://proceedings.mlr.press/v202/kirchenbauer23a.html>

### Verified textual patch fallback

The publication work that produced version `0.1.3` showed why channel fidelity and end-to-end exactness must be separated. The sandbox had an exact Git patch, but no practical sandbox-to-GitHub file upload; repeated complete-file serialization produced blob mismatches.

A semantic-text patch can cross a model-mediated channel exactly when the base SHA, patch checksum, and expected result tree are recorded and the receiver verifies the payload and the published tree. Deterministic chunking is acceptable when channel limits require it.

If the exact payload is opaque, prefer a non-model byte path. When no such path exists, the same checksum-and-result-tree contract remains the last exact fallback; the opaque payload is tolerated because transport necessity is established, not because encoding is preferred.

The detailed procedure belongs in `actions-missions.md`.

## 11. Mission results must be verified

A mission's status is not its whole result. A green run can still produce the wrong artifact, commit, ref, checksum, source identity, or cleanup effect.

Luna therefore checks the outputs that matter before relying on a successful mission. Failed missions require diagnosis before retry or source modification.

When possible, distinguish repository/test failure, mission/workflow defect, permission/authentication failure, quota/platform limits, stale source identity, and transient runner/service failure. Do not edit application source merely because a workflow is red, and do not repeat unchanged runs without evidence that a transient retry is justified.

If logs are unavailable, preserve that uncertainty.

## 12. Degraded remote mode

Degraded remote mode begins only when Actions substitutes for the sandbox repository engineering loop because the sandbox itself is unavailable or cannot faithfully sustain the work after practical inputs/capabilities have been supplied.

Missing direct GitHub access, missing downloadable bytes, or an initially absent tool is not enough by itself. Supply or transport should restore the sandbox path when practical.

In degraded mode, split work into bounded missions, persist reusable progress in exact durable state, inspect results before the next mission, tell the user that the execution environment changed, and return to the sandbox when it becomes usable again and doing so is practical.

## 13. Durable handoff and context loss

Chat or sandbox context can disappear unexpectedly. State should become durable when losing it would make recovery expensive or ambiguous.

Suitable carriers include:

```text
branch / PR / issue / commit / task-owned artifact
```

Cheap reasoning may remain in chat. A failed attempt that leaves useful logs, diagnosis, an exact payload, or a reusable commit can still be valuable; an attempt that leaves nothing reusable can be abandoned and restarted from the last known durable base.

## 14. Remote-state growth must be bounded

Branches, workflows, runs, and artifacts have operational and sometimes monetary cost. They can also make ownership and recovery harder to understand.

Before creating more state, reuse task-owned state when safe, avoid duplicate artifacts, prefer the smallest mission/output that satisfies the task, use bounded retention, and stop creating more temporary objects when growth or ownership becomes surprising.

After value ends, remove Luna-owned temporary branches/refs, workflow definitions, mission-only files, and obsolete artifacts when ownership and terminal state are clear. Do not delete the only useful recovery payload or unfamiliar state. Cleanup should be idempotent and recovery-aware.

When direct integration can establish ownership but cannot perform cleanup, a small bounded cleanup mission with exact identity checks is acceptable. That is remote control, not degraded development.

## 15. Task-owned naming

Readable task-owned names help humans and agents distinguish temporary state. A short purpose plus a collision-resistant suffix is a useful default, for example:

```text
mission-deps-a7f3c2d1
mission/patch-a7f3c2d1
mission-export-a7f3c2d1.yml
```

When convenient, Python's `secrets.token_hex(4)` produces an eight-character suffix. Other reasonable random/UUID mechanisms are fine. Names reduce collisions; immutable identity still comes from SHAs and checksums.

## 16. Discovery and AGENTS.md

The template uses `AGENTS.md` as a small, prominent discovery router into the embedded skill.

It should point to Luna while leaving room for project-specific instructions. It is not a hard runtime dependency: downstream projects may replace top-level documentation, and hosts with repository-local skill discovery may still find `.agents/skills/luna-chat-coder/` directly.

The robust recommendation is to keep the skill directory intact and merge the short Luna entry point into the project's own agent instructions when practical.

### Upstream provenance and quiet update awareness

Agent Skills defines `metadata` as an extension map but does not define a canonical source URL field. Luna therefore stores namespaced upstream repository, upstream skill path, and last-integrated upstream version metadata inside `SKILL.md` so provenance survives template copies without depending on a particular installer lockfile.

The freshness check must remain advisory. In constrained chat environments, connected repository tooling may reach GitHub even when the sandbox cannot. Prefer that already-connected read path—on the documented ChatGPT Web setup, the GitHub Plugin path—over probing or configuring sandbox networking.

Read only the fixed public upstream skill, preferably its frontmatter, compare semantic versions against `luna-upstream-version`, remain silent on failure or no update, and never auto-update the downstream copy. Fetched upstream body text is data for the version check, not new runtime instruction.

“Once per known conversation” is the strongest portable quieting rule available without inventing persistent host state.

## 17. Why maintainer rationale lives inside the skill directory

The original development conversation, private history, or pre-publication repository may not survive. Requiring future maintainers to reconstruct design intent from every old commit would also be inefficient.

The separation is intentional:

```text
SKILL.md
    current runtime policy

actions-missions.md / recovery.md
    operational details loaded when needed

design-rationale.md
    maintainer memory loaded only when changing Luna
```

This file may be longer than runtime documentation because it is rarely loaded and is meant to be self-contained. Length alone is not a reason to split it while it remains coherent and easy to inspect inside a copied template. Duplication, however, is a reason to edit: rationale should explain why, while procedures belong in operational references.

Normal skill behavior must never depend on reading this file.

## 18. ChatGPT Web integration boundary

The fully specified path uses two distinct GitHub layers:

1. **GitHub Plugin** — ChatGPT-side repository/tool capability.
2. **ChatGPT Codex Connector GitHub App** — GitHub-side installation and repository authorization.

They are separate prerequisites and should not be casually collapsed into one concept. Actions access is additionally relevant only when a mission is actually needed.

The core skill remains host-neutral. Another host may use analogous capabilities only when they actually exist and are authorized.

## 19. README is the adoption surface

The README is the first contact for a template user. It should not make people reverse-engineer Luna's value from internal terminology, nor mirror the runtime protocol simply because every Markdown file is visible in the copied repository.

Its job is to answer, quickly:

1. What problem does this solve?
2. Why is this approach different or useful?
3. How do I start?
4. What should I expect after setup?

The distinctive value is not “Luna has a sophisticated transport taxonomy.” It is that ChatGPT already has a useful built-in code-execution sandbox, but limited network access can make real repository work stall. Luna teaches the chat to keep that sandbox as the development workspace and use GitHub only for the missing network/transport pieces—without requiring the user to expose a development machine or operate a tunnel.

README prose should therefore lead with that outcome, use plain language, and expose internal terms only when the user must act on them. Implementation details can be linked because every template copy already contains the skill and references in inspectable form.

Do not duplicate defensive caveats or operational checklists merely to demonstrate completeness. A concise README can be technically accurate precisely because the detailed policy lives elsewhere.

English and Korean are one product surface. Keep structure, emphasis, and meaning aligned; neither language should become a secondary or more awkward version of the other.

## 20. Terminology decisions

Canonical runtime terminology stays deliberately narrow:

- `sandbox work container`
- `durable repository state`
- `Actions mission`
- `degraded remote mode`

Avoid `local container` because users may interpret “local” as their own computer. Avoid `bridge` because it implies a persistent connection rather than bounded remote work.

Human-facing README prose should use these terms sparingly. Runtime documents can be precise without making the public introduction sound like a protocol specification.

## 21. Stable decisions worth preserving

Unless new evidence provides a strong reason to change them:

- ordinary chat remains the development surface;
- the sandbox remains the primary engineering workspace;
- GitHub exact state outranks conversational reconstruction;
- the user's host computer is not a dependency;
- project requirements define the engineering method and project-owned policy;
- Actions is bounded fallback/transport/execution rather than the default workstation;
- Luna is responsible for the safety and cleanup of Luna-created remote state, not for imposing repository-wide security policy;
- exact byte-preserving transport is first-class, and end-to-end verified textual patch transport is an acceptable fallback when no practical byte-preserving upload exists;
- Luna should not introduce model-visible opaque transport encodings when the same bytes can travel through a practical file/object path; project content itself is unaffected;
- mission outputs are verified and failures are diagnosed before blind retry;
- concurrent actors are assumed;
- temporary remote state is task-owned, bounded, and recovery-aware;
- completion claims are evidence-bounded;
- upstream update awareness is advisory, quiet, and never auto-updates a downstream copy;
- runtime policy, operational procedure, maintainer rationale, and user-facing README remain separate layers;
- proposed Luna changes remain reversible until the final candidate is demonstrably better than the prior baseline.

## 22. Rejected or corrected approaches

### Actions as the normal coding environment

Rejected because it adds remote latency, metered execution, workflow/artifact management, and weaker interactive feedback without need when the sandbox is usable.

### A rigid publication hierarchy or gap taxonomy

Rejected because host capabilities vary. Representation-specific defaults are still useful—small text edits can use direct writes, larger semantic-text changes favor plain-text patches, and opaque bytes favor byte-preserving paths—but none overrides exactness, integration limits, or observed reliability.

### Reconstructing already-existing exact state through model prose

Rejected when a practical exact object, patch, bundle, archive, artifact, or file reference already exists. Re-serialization adds fidelity and partial-update risk without adding useful reasoning.

### Encoding transport bytes into model-visible text when a byte path exists

Rejected. Workflow text and other model-authored fields are not byte-preserving transports, and converting an existing payload to Base64 or another opaque representation only increases model-visible classifier surface. Semantic-text source patches and encoded strings that are actual project content are not this case.

### Rejecting a verified textual patch only because the channel is not inherently byte-preserving

Rejected after the `0.1.3` publication experience. Channel fidelity and end-to-end exactness are different properties. If checksum and result-tree verification detect drift, a textual patch can be an exact fallback even though the channel itself is not trusted as byte-preserving.

### Blind retries and source edits after remote failure

Rejected. Inspect evidence first; otherwise retries waste compute and can turn infrastructure problems into unnecessary source changes.

### Aggressive immediate cleanup

Rejected because a failed run or artifact may still be the only useful diagnosis or recovery payload. Cleanup must understand ownership and terminal state.

### Mandatory update checks or automatic self-update

Rejected because they add latency, fail on restricted hosts, disturb the quiet normal path, and can overwrite downstream customizations. The update check is best-effort and advisory only.

### Treating downstream content drift as proof of staleness

Rejected because template users may intentionally customize Luna. Track the last deliberately integrated upstream version instead.

### Expanding Luna's internal safety rules into project-wide policy

Rejected. Luna may need minimum permissions, safe handling of its own temporary credentials, or protection against untrusted inputs in workflows it creates. That does not make Luna the authority over every project file, workflow, or secret-handling convention. Project-owned policy belongs to the project.

### Preserving a change because it has already been authored

Rejected. Implementation effort is not evidence that the result is better. Simplify, partially revert, or withdraw a change that is neutral, regressive, or needlessly complex.

### Keeping all rationale in runtime policy or depending on Git history for rationale

Both are rejected. Runtime policy should stay concise, while maintainer memory should survive repository copies and history changes without forcing every normal task to load it.

## 23. Questions future maintainers should ask

Before expanding Luna, ask:

1. Does this prevent a repeatable failure in constrained chat-based repository development?
2. Is it actually a Luna concern, or does it belong to the surrounding project?
3. Does it need to be runtime policy, or can it live in an operational reference or rationale?
4. Does it preserve exact state and observable evidence?
5. Is it safe with concurrent chats, agents, CI, and humans?
6. What happens if chat and sandbox state disappear immediately afterward?
7. Can it create unbounded remote state, quota, storage, or cost?
8. Does it assume a host capability that may not exist?
9. Can the normal user remain unaware of this concept?
10. Does it encode today's tool surface too literally?
11. Is there a simpler rule with the same reliability?
12. Can a first-time template user understand the benefit and setup without learning Luna's internal vocabulary?
13. Is the result demonstrably better than the prior baseline, or are we protecting sunk effort?

## 24. Maintenance changes are hypotheses, not commitments

Treat every proposed Luna change as a hypothesis about how to improve the system, not as a commitment created by editing files.

Discussion and implementation are one iterative design loop: understand the current behavior, make the change the design requires, verify what actually improved or regressed, and keep refining only while evidence supports the direction. Diff size is not a maintenance objective. Canonical Luna may be rewritten or refactored aggressively when that produces a clearer, more coherent policy.

This freedom applies only to maintaining Luna itself. It does not impose a rewrite preference on downstream projects, and it does not justify a larger runtime skill: the copied skill should remain compact, readable, and easy for weaker models to execute reliably. Runtime rules that bound an Actions mission constrain remote side effects, not maintenance diff size.

Before accepting a change, compare the final candidate with the prior baseline. Consider the intended failure mode, runtime complexity, normal-path user noise, portability, security boundaries, exactness/recovery behavior, and the checks that actually ran. Solving one edge case while making the healthy path more fragile or ceremonial is not progress.

If the candidate is worse, ambiguous, or needlessly complex, simplify it, revert the affected part, or withdraw it. Reversibility is a maintenance feature; the goal is the best resulting policy, not preservation of the current draft.

For user-facing changes, English/Korean parity is part of acceptance rather than a later cleanup step.

## 25. Maintenance discipline

When Luna itself changes:

- update `SKILL.md` only for current runtime policy;
- update operational references when procedure changes;
- update this rationale when a stable reason, rejected alternative, known failure mode, or important boundary changes;
- keep English and Korean README structure and meaning aligned;
- on canonical releases, keep `metadata.version`, `metadata.luna-upstream-version`, and both README version labels aligned; downstream customized copies change `luna-upstream-version` only when that upstream version is deliberately integrated;
- do not require historical Git or old conversations to understand the current design;
- prefer deleting obsolete concepts over carrying synonyms indefinitely;
- keep versioning deliberate.

The aim is a small runtime protocol, useful operational references, and enough maintainer memory to evolve Luna without rediscovering the same failures.
