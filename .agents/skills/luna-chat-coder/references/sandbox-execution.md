# Long-running Sandbox Commands

Use this reference when a command times out or is likely to outlast a tool call.

## Check what stopped

Inspect the current call's timeout behavior, job state, and outputs. Distinguish the tool's wait window from a command-killing wrapper or a sandbox-wide limit, and check for surviving work before retrying.

## Choose a workable execution path

Prefer a supported persistent session or job handle. For a call-scoped limit, also consider a supervised detached job. On compatible Unix hosts, `setsid` with `nohup` and redirected input/output, or an equivalent process API, can provide detachment. Choose from the current host's capabilities and verify the needed behavior with a small task-relevant check.

## Follow the job through completion

Keep each attempt identifiable, bind it to stable source/inputs, and retain logs plus the actual command's terminal status. Poll with short calls and verify the expected outputs before reporting completion. Resolve missing status or surviving work before rerunning side effects.

Give the job a finite execution budget and supervision that can stop the workload, including its child processes. Before cleanup, confirm current ownership; PID files restored from a snapshot are historical state. Finish or stop owned jobs before handoff when possible, and report anything unresolved.

When no permitted sandbox execution path can sustain the step, use a bounded [Actions mission](actions-missions.md).
