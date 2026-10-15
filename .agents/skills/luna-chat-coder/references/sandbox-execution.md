# Sandbox Execution

Use this reference when an execution-tool invocation fails, a command times out, or work is likely to outlast a tool call. An interface failure and a sandbox-wide execution failure are different observations.

## Invocation errors: try a permitted alternative first

If a call returns an invocation error such as `ClientError` without program output or an exit status, record that exact result. It does not establish whether the command started, whether the workspace survived, or why the call failed. Do not infer censorship, OOM, deletion, or a platform-wide incident from the error name.

Before declaring sandbox execution unavailable, make a small, dependency-free check through another already available and permitted sandbox execution interface. In ChatGPT, a failure of `container.exec` may leave `python.exec` usable; Python's standard-library `subprocess` can then invoke the required commands in the sandbox. A user-visible Python tool such as `python_user_visible.exec` is another candidate only when its tool contract permits that use and the code/output is appropriate to show the user. Prefer a non-user-visible interface for internal diagnostics. These names are examples, not guaranteed capabilities or evidence of independent backends. On another chat host, look for the equivalent permitted code interpreter or process interface rather than assuming ChatGPT tool names exist.

Start with simple output. If the interpreter works, test process execution with a tiny standard-library child command, for example:

```python
import subprocess
import sys

result = subprocess.run(
    # Isolate this probe from site startup, not the project's real commands.
    [sys.executable, "-I", "-S", "-c", "print('sandbox-subprocess-ok')"],
    capture_output=True,
    text=True,
    timeout=5,
    check=False,
)
print("exit:", result.returncode)
print("stdout:", result.stdout)
print("stderr:", result.stderr)
```

Inspect the actual status and outputs; interpreter success alone does not prove subprocess or repository-required execution works. Before replaying interrupted work, inspect surviving task-owned processes, logs and outputs, and verify the expected repository/source identity through the working interface. A different interface may expose a different or reset workspace; use [recovery.md](recovery.md) if identity is uncertain. Do not launch a duplicate job or repeat side effects merely because the prior call returned no status.

If the alternative sustains the required commands, continue the normal edit/build/test loop there with the same verification requirements, source binding and terminal-status tracking. This is still sandbox execution, not degraded remote mode. Do not keep retrying the broken interface while a suitable one works. Long-running subprocess work still needs the supervision below; changing interfaces does not guarantee a longer execution budget.

## Bound the diagnosis

For the same failure episode, use only the smallest discriminating probes: one minimal check of the failing route when needed and one suitable alternative route, with a child-process check only if that interpreter succeeds. Do not enumerate every tool alias or loop equivalent calls. An explicit safety/permission denial is not a transient invocation error to bypass through another interface.

If no permitted sandbox path works, preserve concise observed recovery state and follow the project's authorized fallback. Where remote execution is prohibited, stop execution attempts and report the blocker. A later user-requested retry is a reasonable new check opportunity, not a guaranteed reset; recheck minimally and continue if it works. Do not invent a recovery probability, fixed waiting interval, or claim that a new message itself repairs the environment.

Report the attempted interface and command, available time, output/status or exact error, last verified source and any unresolved job/workspace state. Retain successful evidence separately from later failures. Unknown survival or start state stays unknown; neither a successful print nor an older passing test proves a new product check passed.

## Long-running commands

### Check what stopped

Inspect the current call's timeout behavior, job state, and outputs. Distinguish the tool's wait window from a command-killing wrapper or a sandbox-wide limit, and check for surviving work before retrying.

### Choose a workable execution path

Prefer a supported persistent session or job handle. For a call-scoped limit, also consider a supervised detached job. On compatible Unix hosts, `setsid` with `nohup` and redirected input/output, or an equivalent process API, can provide detachment. Choose from the current host's capabilities and verify the needed behavior with a small task-relevant check.

### Follow the job through completion

Keep each attempt identifiable, bind it to stable source/inputs, and retain logs plus the actual command's terminal status. Poll with short calls and verify the expected outputs before reporting completion. Resolve missing status or surviving work before rerunning side effects.

Give the job a finite execution budget and supervision that can stop the workload, including its child processes. Before cleanup, confirm current ownership; PID files restored from a snapshot are historical state. Finish or stop owned jobs before handoff when possible, and report anything unresolved.

When no permitted sandbox execution path can sustain the step and the project permits remote execution, use a bounded [Actions mission](actions-missions.md). Otherwise report the blocker without silently changing the execution environment.
