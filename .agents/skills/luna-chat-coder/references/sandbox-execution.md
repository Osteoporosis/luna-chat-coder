# Sandbox Execution

Use this reference for invocation failures, timeouts, or work that may outlast a tool call. A failed interface does not establish sandbox-wide failure; a command's nonzero exit status alone does not establish an invocation failure.

## Recover from invocation failures

Record the exact error, output, and status available. An error such as `ClientError` without program output or exit status leaves command start, workspace survival, and cause unknown. Do not infer censorship, OOM, deletion, or a platform-wide incident from its name.

1. **Refresh the available routes.** In ChatGPT, `container.exec`, `python.exec`, and `python_user_visible.exec` are all usable sandbox execution routes; Python's standard-library `subprocess` can run commands. Select by the current host's exposed capabilities and tool contracts, not a fixed list, count, or preference order. Names and capabilities can change; use analogous execution routes on other LLM hosts. Distinct tools need not have independent backends or the same workspace.
2. **Make one fresh recovery pass.** Before declaring execution unavailable, try every currently available, permitted route once, including the route that just failed. Earlier failures do not count toward this pass. Use minimal, dependency-free probes with short timeouts: simple output first, then a tiny standard-library child command if an interpreter works. Keep probes independent of project/site startup; do not apply probe-only isolation to real project commands. Inspect actual status and outputs; interpreter success alone does not prove process execution or the required repository commands work.
3. **Continue as soon as a route can sustain the work.** Before replaying interrupted commands, inspect surviving task-owned processes, logs, and outputs, and verify workspace/source identity; use [recovery.md](recovery.md) if uncertain. Never duplicate unresolved jobs or side effects. Continue the sandbox engineering loop with unchanged verification, source binding, and terminal-status tracking; this is not degraded remote mode. Stop probing once a suitable route works. Changing interfaces does not guarantee a longer execution budget.

Declare sandbox execution unavailable for the required step only if every route in the fresh pass fails to sustain it. Bound the pass to these checks; do not start repeated cycles without new evidence or a later user-requested retry. Such a retry is a new check opportunity, not a guaranteed reset. Do not invent recovery probabilities, waiting intervals, or claims that a new message repairs the environment. An explicit safety/permission denial must not be bypassed through another route.

## Long-running commands

1. **Check what stopped.** Inspect timeout behavior, job state, and outputs. Distinguish a tool wait window from a command-killing wrapper or sandbox-wide limit; resolve surviving work or missing status before rerunning side effects.
2. **Choose supported supervision.** Prefer a persistent session or job handle. For call-scoped limits, consider a supervised detached job: on compatible Unix hosts, `setsid` with `nohup` and redirected input/output, or an equivalent process API. Verify the needed behavior with a small task-relevant check.
3. **Track completion.** Identify each attempt, bind it to stable source/inputs, retain logs and the actual command's terminal status, poll with short calls, and verify expected outputs. Give jobs a finite budget and supervision that can stop the workload and its children. Confirm current ownership before cleanup; restored PID files are historical state. Finish or stop owned jobs before handoff when possible; report unresolved work.

## Fallback and evidence

If no permitted sandbox route sustains the step after the applicable recovery and supervision checks, preserve observed recovery state and follow the project's authorized fallback. Use a bounded [Actions mission](actions-missions.md) where remote execution is permitted; otherwise stop execution attempts and report the blocker without silently changing environments.

Report attempted routes/commands, available execution time, output/status or exact errors, last verified source, and unresolved job/workspace state. Keep successful evidence separate from later failures and unknown start/survival state explicitly unknown. Neither a successful probe nor an older passing test proves a new product check passed.
