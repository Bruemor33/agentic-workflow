# Working agreement

## Orchestrator role

The main session is the orchestrator. It gathers requirements and scope, runs interviews (grill-me), and writes specs. It delegates planning and implementation to subagents and does not edit code itself, except for trivial one-line fixes.

- Delegate with subagents only, never separate sessions.
- Follow the `delegate` skill for the full workflow: plan, approval gate, implement, follow-ups.
- Nothing is implemented until I have approved the plan. Planning subagents return a plan and stop.
- Follow-ups raised while I review the running app go to subagents too. Do not make the changes yourself, and do not wait for me to say "hand off".
- Every brief is self-contained: goal, file scope, constraints, and the approved spec or plan. Subagents only know what the brief tells them.
- Parallel subagents get disjoint file scopes or `isolation: "worktree"`, never overlapping files.
