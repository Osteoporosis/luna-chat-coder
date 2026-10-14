# Luna Chat Coder Design Rationale

This is maintainer memory: the problems, choices, and trade-offs behind Luna. It travels with the skill so future maintainers can understand the design without old conversations or Git history.

Read it when changing Luna. Current behavior belongs in [`../SKILL.md`](../SKILL.md), with procedures in operational references; reconcile disagreements rather than treating this file as runtime policy.

## Purpose and scope

Ordinary chat already provides a useful development surface, but its sandbox and context are disposable, capabilities vary, and repository state can change concurrently. Luna supplies continuity, exact transport, and bounded fallback so the user can give the chat a repository and a task without operating extra infrastructure.

The project defines its engineering method and security policy. Luna helps the chat carry them out; it does not choose substitute technologies or make access to the user's computer a dependency. That boundary preserves both project intent and the sandbox's isolation.

The canonical repository is also the template source. Luna's normal repository-work policy applies while maintaining Luna itself, but maintainer freedom to redesign the skill does not become a rewrite preference for downstream projects.

## Why the sandbox stays primary

The sandbox work container supports direct inspection and a natural edit/build/test loop. Keeping viable work there avoids remote startup, transport, and lifecycle overhead. Inventory before acquiring follows from this: use what is available, then supply only what the project needs.

**Discover early, activate late** keeps fallback available without making the healthy path ceremonial. A missing dependency or network route usually calls for restoring the sandbox's capability; degraded remote mode is for replacing an engineering loop the sandbox cannot sustain.

### Call-scoped timeouts

A command-call deadline can be mistaken for a limit on all sandbox work. Past timeout incidents motivated considering persistent sessions or supervised detached jobs so a usable sandbox is not abandoned prematurely.

## Exact state and transport

GitHub commits and verified immutable artifacts preserve source identity across chat and sandbox loss. Conversation preserves intent; a surviving workspace preserves potentially valuable edits. This explains both the recovery preference for durable state and the need to reconcile, rather than overwrite, surviving work.

Materializing exact source before editing gives inspection, execution, and publication a common base. Mutable refs coordinate work; resolved SHAs identify what was actually read, tested, or changed.

Transport choices depend on the payload and the capabilities available in each direction. Small semantic edits fit file operations, larger textual changes can travel as patches, and opaque or filesystem-sensitive state benefits from object/file transfers. These are useful defaults rather than a fixed hierarchy. Native payloads must fit the sandbox, and their carrier must preserve the filesystem semantics the project needs.

Exactness is byte identity, not a particular encoding. Past full-file regeneration produced blob mismatches; end-to-end verification made textual patch transport usable despite a generative channel. Payload checksums and the expected result tree keep a transfer problem from being mistaken for a source problem. The same verification contract remains a fallback for opaque payloads when no practical byte path exists.

Earlier model-visible opaque encodings also coincided with long safety-classifier waits. That experience favors avoiding unnecessary encoded transport text while adapting to current capabilities, rather than depending on relaxed safety review. The distinction is between Luna's transport representation and project content: existing content stays intact, and non-model byte channels may use any exact, efficient representation.

## Why Actions is a bounded mission

An Actions mission has a purpose, exact inputs, inspectable outputs, and an end. That model makes remote work recoverable without turning a runner into an interactive workstation. Supply, transport, execution, and cleanup are roles of the same mechanism, not separate infrastructure.

The meaningful distinction is whether Actions supports the sandbox loop or replaces it. Supply and transport can perform work needed to prepare or verify their payloads; degraded remote mode substitutes for the engineering loop itself. This keeps useful remote capabilities available without making them the default development environment.

Acceptance comes from outputs tied to the requested source, not merely a run's status. Diagnosis before retrying keeps infrastructure and transport failures from provoking unrelated source edits. Logs and reusable results can retain value even when a mission fails.

## Ownership, safety, and lifecycle

Concurrent actors make identity and ownership essential. Task-owned names reduce collisions; exact refs and recorded ownership make publication and cleanup safe. Age, naming, or ancestry alone cannot establish that another actor's work is disposable.

Luna is responsible for the mission machinery and temporary state it creates. Minimum privilege, secret-free durable output, and separation of untrusted code from privileged execution protect that state even if repository visibility changes. This responsibility stays scoped to Luna's side effects rather than imposing a security model on the surrounding project.

Temporary remote state has cost and complicates recovery. Reuse and bounded retention keep it proportional to active work; retirement follows the end of review, diagnosis, or recovery value. A small cleanup mission is useful when ownership is known but the connected path lacks the operation. It remains remote control, not degraded development.

## Handoff and recovery

Durable handoff is worthwhile when losing work would make recovery expensive or ambiguous. Cheap intermediate reasoning can stay in chat; useful failed attempts deserve preservation, while attempts with nothing reusable can be restarted from the known base.

A private downloadable sandbox snapshot fills a different gap: user possession of substantial, unpublished repository work. Its trigger follows accumulated work and reproduction cost, including documentation, rather than importance or a fixed line count. Environment churn alone is not repository work.

Once triggered, the snapshot captures the actual project root, including Git metadata and untracked or ignored state. Filtering it down to publishable files would defeat that purpose. Capture stays within the root and the archive stays outside it. The snapshot supplements durable source identity; it does not replace publication or verification.

This is a convenience on hosts that already support direct private file handoff. Best-effort creation keeps a failed snapshot from blocking publication or spawning remote infrastructure solely to imitate a missing download path.

## Discovery, portability, and adoption

`AGENTS.md` is a small discovery entry point that coexists with project instructions. Keeping the whole skill directory together lets policy and maintainer context survive template copies even when top-level documentation is replaced. Normal use loads the runtime policy and only the operational references needed for the task.

The core policy is host-neutral; the documented ChatGPT Web path distinguishes the chat-side GitHub Plugin from the GitHub-side ChatGPT Codex Connector App and its repository authorization. Other hosts must supply equivalent capabilities rather than merely parse the skill. Actions access matters when a mission needs it.

Canonical terms separate the sandbox work container from the user's computer and an Actions mission from a persistent connection. README prose should instead lead with the benefit, setup, and expected experience. English and Korean are one adoption surface, with aligned structure and meaning rather than duplicated internal checklists.

Upstream metadata records provenance and the last deliberately integrated version because downstream copies can legitimately diverge. An advisory check through an already-available read path preserves update awareness without auto-updating customizations or making network setup part of every task.

## Maintaining the design

Treat a change as a hypothesis: understand the failure, implement the idea, compare the result with the prior baseline, and simplify or withdraw it when the improvement is unclear. Canonical Luna can be refactored substantially; diff size and effort already spent are not reasons to keep a design. Judge the result by clarity, reliable execution, recovery, portability, and the cost it adds to the healthy path.

Preserve the issue, intent, and consequential trade-off once, at the appropriate layer. Runtime policy states what matters; operational references help carry it out; rationale explains why. Session measurements and experiment logs belong in review evidence. This leaves future agents room to choose better mechanisms while retaining the reasons that matter.

Prefer a concise decision rule over enumerating possible future exceptions. Keep concrete detail when removing it would change a meaningful boundary or make the intended action unclear; brevity alone is not an improvement. Verify both the intended benefit and the important behavior that must remain unchanged.

Keep English/Korean README meaning aligned. On canonical releases, align `metadata.version`, `metadata.luna-upstream-version`, and both README version labels; downstream copies advance the upstream marker only when deliberately integrating that version.
