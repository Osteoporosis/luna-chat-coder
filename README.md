# Luna Chat Coder

[한국어 README](README.ko.md)

**Version 0.1.4**

Use ordinary ChatGPT Web conversations for real GitHub repository work—without running a local coding agent, opening a tunnel, or giving the chat access to your computer.

ChatGPT already has a sandbox that can run code. The catch is that network restrictions can stop repository work when the chat needs source, dependencies, or a reliable way to publish a larger change. Luna teaches the model to keep the development loop in that built-in sandbox and use connected GitHub access only for the missing pieces.

## What you get

- **A useful built-in workspace.** Editing, building, testing, and debugging stay in the chat sandbox whenever it can do the job.
- **Fewer dead ends.** If the normal path cannot complete a step reliably, Luna can use GitHub for that step instead of giving up or moving the whole workflow elsewhere.
- **Safer recovery.** If the chat or sandbox disappears, Luna resumes from exact GitHub state rather than trying to recreate code from conversation history.
- **Reliable handoff.** Luna checks the state it worked on and the result it publishes before reporting completion.

The point is simple: give the chat a repository and a development task, not a new piece of infrastructure to operate.

## Quick start

For the ChatGPT Web setup documented here:

1. Choose **Use this template → Create a new repository**.
2. In ChatGPT, install/connect the **GitHub Plugin** from <https://chatgpt.com/plugins>.
3. On GitHub, install the **ChatGPT Codex Connector** from <https://github.com/apps/chatgpt-codex-connector> and grant it access to the new repository. If the App is already installed for selected repositories, add the new repository to that list.
4. In a normal ChatGPT conversation, send the repository URL and the development task—for example, ask it to implement a change and open a pull request.

That is the normal workflow. A repository created from this template already contains Luna, and you should not need to mention Luna by name or manage its internal recovery steps yourself.

Organization policy may require an administrator to approve the Plugin or GitHub App.

## How it works

Luna reads the repository's own instructions and requirements, recovers the exact source it should work from, and uses the chat sandbox for the normal edit/test loop.

When direct sandbox access is not enough, Luna can use the connected GitHub path for the missing step. If that path still cannot complete the step reliably, a bounded GitHub Actions run can handle it, then return the work to the sandbox when possible. GitHub Actions is not the default coding environment.

## Add Luna to an existing repository

Copy the complete skill directory:

```text
.agents/skills/luna-chat-coder/
```

Then merge the short Luna entry-point instruction from [`AGENTS.md`](AGENTS.md) into the repository's existing agent instructions. Keep the project's own engineering guidance; Luna works around it rather than replacing it.

For the documented ChatGPT Web path, connect the GitHub Plugin and grant the ChatGPT Codex Connector access to the repository before asking the chat to work on it.

## Documentation

Runtime behavior lives in [`SKILL.md`](.agents/skills/luna-chat-coder/SKILL.md). Operational details are in [`actions-missions.md`](.agents/skills/luna-chat-coder/references/actions-missions.md) and [`recovery.md`](.agents/skills/luna-chat-coder/references/recovery.md). [`design-rationale.md`](.agents/skills/luna-chat-coder/references/design-rationale.md) is maintainer memory for changing Luna itself; normal skill use does not depend on it.

Luna follows the Agent Skills structure. ChatGPT Web is the path documented and tested here; another host can use the same skill when it provides equivalent sandbox and GitHub capabilities.

## License

MIT. See [`LICENSE`](LICENSE).
