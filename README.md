# Luna Chat Coder

[한국어 README](README.ko.md)

**Version 0.1.2**

> AI agents working from this repository should read [`AGENTS.md`](AGENTS.md) first.

Luna Chat Coder is a repository template with an embedded fallback skill for reliable software development from ordinary chat.

After the one-time setup, the intended user experience is simple: give ChatGPT the repository and the development task. Luna is discovered from the repository, uses the chat sandbox for normal work, and reaches for GitHub Actions only when that normal path is insufficient.

## Quick start

For the ChatGPT Web path documented by this repository:

1. On this repository, choose **Use this template → Create a new repository**.
2. In ChatGPT, open the Plugins Directory at <https://chatgpt.com/plugins> and install/connect the **GitHub Plugin**.
3. On GitHub, install the **ChatGPT Codex Connector** from <https://github.com/apps/chatgpt-codex-connector> and grant it access to the new repository. If it is already installed with selected repositories, add the new repository to its access list.
4. In an ordinary ChatGPT conversation, provide the repository URL and ask for the development work you want. You should not need to mention Luna Chat Coder; the repository's `AGENTS.md` points the model to the embedded skill.

A repository created from this template already contains Luna; no separate Luna installation is required.

Steps 1–3 are one-time user/admin setup in the validated ChatGPT Web workflow. Repository creation, plugin connection, GitHub App installation, and repository authorization should be completed before expecting the chat to operate that repository. Organization policy may require administrator approval.

For an existing repository, see [Add to an existing repository](#add-to-an-existing-repository).

## What Luna does automatically

On repository work, the model should quietly:

1. read `AGENTS.md` and the embedded `SKILL.md`;
2. identify the exact repository/PR/commit state and inspect any surviving sandbox work;
3. materialize the exact target commit or PR-head source as a complete sandbox working tree and verify its base identity before editing;
4. inventory the sandbox before installing or acquiring anything;
5. perform normal edit/build/test/debug work in the sandbox work container;
6. use a bounded Actions mission only when supply, exact transport, or genuinely degraded execution makes it useful;
7. verify executable behavior with the checks required by the repository and task;
8. publish the exact verified change through the simplest reliable GitHub path;
9. report only what actually ran, and mention degraded remote execution when it was necessary.

A healthy task should not make the user operate Luna or watch its internal checklist.

## When Actions missions are useful

GitHub Actions is a bounded remote mechanism, not the default development environment.

Three common mission roles are:

- **Supply** — the sandbox can do the engineering work but cannot obtain a required dependency/package set, runtime, SDK, compiler, executable/application distribution, installer, native library, generated input, or similar external input.
- **Exact transport** — carry exact repository source into the sandbox when direct Git access is unavailable or impractical, or carry a verified change back to GitHub when a byte-preserving archive/patch/bundle is safer or more efficient than repeated content writes.
- **Degraded remote execution** — the sandbox itself is unavailable or cannot sustain the task because of a hard usage, duration, resource, or execution limit.

Transport is bidirectional and is a choice, not a last-resort punishment. Small intentional textual edits may use direct file operations; when exact source or verified changes already exist and reconstruction would add meaningful fidelity or partial-update risk, prefer byte-preserving archives, checksummed artifact payloads, Git objects, patches, or bundles when practical. Luna chooses the lowest-overhead path that remains exact and reliable for the observed payload.

Every mission result, including a reported success, should be checked against the expected output contract before it is relied on. A green run does not by itself prove that the intended artifact, commit, ref, checksum, or source identity is correct. Failures additionally require diagnosis from the returned error, failing step, logs, and partial results before source changes or retries; repeated blind retries are specifically discouraged.

Detailed mission rules live in [`actions-missions.md`](.agents/skills/luna-chat-coder/references/actions-missions.md).

## Why Luna exists

Chat-based development already has a useful execution environment. Luna exists to make better use of it without pretending it is a persistent developer workstation:

- the sandbox may reset or disappear;
- network, storage, resource, duration, or usage limits can block required work;
- repositories may require tools or external inputs not initially present;
- conversation text preserves intent well but is a poor source of exact bytes;
- GitHub Actions is remote, metered execution with workflow, startup, artifact, and cleanup overhead.

The policy is therefore **sandbox first; use bounded remote help for supply or transport, and move engineering execution remote only when the sandbox itself cannot sustain it**. Inventory what already exists before acquiring more.

The repository defines its own engineering method. Luna does not choose a database, test framework, runtime, or substitute technology merely because it is easier to run. It makes the repository's declared requirements executable as faithfully as practical.

The user's own computer is intentionally outside the workflow. Normal repository development should not require direct host access or weaker host isolation.

## Exact publication and recovery

GitHub is the durable source of exact repository state. For recovery, prefer:

```text
commit / PR head
    > immutable Git or Actions artifact
    > surviving sandbox working tree
    > conversation reconstruction
```

For source acquisition and publication, choose among connected repository operations, native Git object operations, or archive/checksummed-artifact/patch/bundle transport according to the actual payload and observed integration reliability. A substantial change should be bound to an expected base SHA. If the base moved, recover and deliberately rebase, merge, or recreate the payload.

Do not recreate an exact source tree or verified multi-file/binary payload from prose or model-authored complete-file/blob content when practical byte-preserving transport can carry the existing bytes.

Temporary mission state is task-owned and bounded, but cleanup is recovery-aware. Failed runs, branches, artifacts, or logs should remain while they still have debugging, review, handoff, or recovery value. If conversational context is lost, reconstruct ownership and terminal state from durable GitHub evidence before deleting unfamiliar remote objects.

See [`recovery.md`](.agents/skills/luna-chat-coder/references/recovery.md) and [`actions-missions.md`](.agents/skills/luna-chat-coder/references/actions-missions.md) for the detailed rules.

## Design model (optional)

Users do not need this model to use Luna, but it explains the design:

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

An Actions mission is closer to an unmanned deep-sea or space probe than to a live remote terminal: give it an exact source identity, inputs, purpose, and return contract; let it run independently; then inspect the durable result after it terminates.

## Why the ChatGPT Web path has two GitHub connections

The workflow documented here requires two separate layers:

1. **GitHub Plugin** — the ChatGPT-side workflow/tool capability.
2. **ChatGPT Codex Connector GitHub App** — the GitHub-side installation that is granted access to the target repository.

Both must be available and authorized for the target repository. They are not interchangeable. Actions/workflow/log/artifact access is additionally needed only when an Actions mission is actually required.

UI names can change; the capability and authorization boundary is what matters.

## Repository discovery and layout

The template deliberately separates discovery, runtime policy, operational details, and maintainer memory:

```text
AGENTS.md
    -> small repository entry point
    -> tells the model to read the skill before chat-based development

.agents/skills/luna-chat-coder/
  SKILL.md
      -> canonical machine policy
  references/
    actions-missions.md
      -> mission, failure-diagnosis, transport, lifecycle, and cleanup rules
    recovery.md
      -> recovery after sandbox/chat/context loss or source ambiguity
    design-rationale.md
      -> maintainer-only design memory for changing Luna itself
```

The intended pattern is **discover early, activate late**. Loading Luna does not mean Actions should run. On the healthy path the skill should be mostly invisible.

`AGENTS.md` is a discovery accelerator, not a hard runtime dependency. Downstream projects may replace their top-level README or AGENTS content. Keep the skill directory intact, and when practical merge the short Luna entry-point instruction into the project's own `AGENTS.md` rather than preserving Luna's original file verbatim. Hosts that support repository-local skill discovery may still find the skill without that pointer, but Luna does not assume every host will.

The maintainer rationale intentionally lives inside the skill directory so it survives template copies, top-level file replacement, repository migration, and a public-history reset. Normal development tasks do not need to read it.

## Portability

The embedded skill follows the Agent Skills structure and keeps the core policy host-neutral. Agent Skills is an open cross-platform format: <https://agentskills.io/>.

This repository documents one fully specified integration path: **ChatGPT Web + GitHub Plugin + ChatGPT Codex Connector**. Other Agent Skills hosts can use the same sandbox-first, durable-state, exact-transport, and bounded-mission policy when they actually provide equivalent code-execution and GitHub capabilities.

Format compatibility alone is not a promise of full operational support. A host must not invent repository write, Actions, logs, artifacts, or credentials it does not have.

## Add to an existing repository

Copy the complete skill directory:

```text
.agents/skills/luna-chat-coder/
```

Then add the short Luna entry-point instructions from this repository's `AGENTS.md` to the target repository's existing agent instructions. Keep the project's own engineering guidance; Luna is a continuity/fallback layer around it, not a replacement.

For the validated ChatGPT Web path, connect the GitHub Plugin and grant the ChatGPT Codex Connector access to that repository before asking the chat to work on it.

## Scope

Luna Chat Coder covers:

- early repository-policy discovery for chat-based development;
- exact recovery across sandbox or conversational context loss;
- sandbox-first execution and capability inventory;
- faithful acquisition of repository-required missing inputs;
- exact source/change transport in either direction when it is the better byte-preserving path;
- bounded GitHub Actions missions and failure diagnosis;
- degraded remote execution when the sandbox itself is unavailable or cannot sustain the required execution;
- evidence-based completion reporting;
- recovery-aware cleanup of temporary mission-owned remote state.

Keep the protocol small. Add rules only when they prevent a repeatable failure in constrained chat-based repository development.
