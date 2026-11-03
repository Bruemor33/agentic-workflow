---
name: delegate
description: Hand a spec, task, or follow-up to subagents for planning and implementation, with an approval gate between the two. Use after a grill-me/to-spec session, when the user says "hand off", "delegate", "plan this", or "have a subagent do it", and for follow-up changes raised while reviewing the running app.
---

The session agent orchestrates. Subagents do the work. Use subagents only, never separate sessions.

## 1. Frame the work

Confirm you have the spec or task summary and the file scope. If either is missing, get it from the conversation or ask once. Split the work into independent pieces only where file scopes don't overlap; otherwise use one subagent.

## 2. Plan phase

Spawn one planning subagent per piece, in a single message so they run in parallel. Each brief contains:

- Goal and the relevant part of the spec, pasted in full
- File scope (what to read, what may later be changed)
- Constraints, conventions, and anything ruled out
- Instruction: **return a plan and make no edits.** The plan lists files to change, the approach, risks, and how to verify.

## 3. Approval gate

Present each plan to the user: summary first, full plan on request. Wait for approve, edit, or reject. Do not start implementation before an explicit approval. Apply the user's edits to the plan text yourself.

## 4. Implement phase

Send the approved plan to the planning subagent with `SendMessage` if it still has the context. Otherwise spawn a fresh subagent with the approved plan pasted into the brief. Rules:

- Pieces that could touch the same files run one at a time, or each gets `isolation: "worktree"`.
- Tell the subagent to follow the plan, stay within scope, run the project's tests or checks, and report what changed and anything it deviated on.
- Check the reported result against the diff before telling the user it's done.

## 5. Follow-ups

When the user comes back with changes after trying the app, treat each as a new delegation. Don't edit code in the main session and don't wait to be told to hand off.

- Small, clear change: brief the subagent that already knows the area (`SendMessage`), or a fresh one, and implement directly.
- Large or ambiguous change: run a short plan phase and gate first.
- Brief contents: the user's feedback quoted, the files involved, what was already built, and what must not change.
