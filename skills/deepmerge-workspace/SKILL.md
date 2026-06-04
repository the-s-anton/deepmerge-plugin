---
name: deepmerge-workspace
description: >
  Deepmerge is the shared workspace where a team's AI agents read and write one
  attributed, durable memory across Claude, ChatGPT, Cursor, and any MCP client.
  Use it whenever starting a non-trivial task (check what teammates' agents
  already did before reasoning from scratch), the moment a deliverable is
  produced or something is learned or decided (save it so the next agent builds
  on it), at the start of a session or after a context reset (load what changed),
  or when the user asks what their agents did, what was decided, or to hand work
  off. Drives the read-before, write-after loop against the deepmerge MCP tools.
when_to_use: >
  When beginning any research, coding, writing, planning, or analysis task;
  right after creating a report, spec, plan, decision, or finding; on session
  start or reconnect; when the user says "what did we do", "what did the agents
  decide", "continue where we left off", "save this", or "remember this"; and
  before redoing work a teammate's agent may already have done.
allowed-tools:
  - mcp__plugin_deepmerge_deepmerge__*
  - mcp__deepmerge__*
---

# Deepmerge: the shared workspace for your team's agents

Deepmerge is one durable, attributed memory that every agent on the team reads
and writes, across every tool and every project. Work the loop below on every
session, not once. What an agent does not write here is lost to the team,
including a future version of itself after a context reset.

The live MCP tools carry their own descriptions and the server ships a full
manual; this skill governs WHEN to reach for them, not how each one works.

## The loop

**Session open.** Call `get_self` first. It returns identity, what changed since
this agent was last here (`entries_since_previous_seen`), and which peers are
online. If it reports an empty workspace, capture what the user is working on
right now and tell the user in one line what was saved.

**Before any non-trivial task.** Call `get_context(task)` before reasoning from
scratch. A teammate's agent may already have done this work; build on what comes
back instead of repeating it. Use `get_entries(query, kinds)` for raw filtering
(`kinds: ["skill"]` for reusable how-tos, `kinds: ["decision"]` for prior calls).

**As work is produced.** The moment a deliverable, learning, or decision exists,
`create_entry` it, while the work happens, not after. Pass `content` as natural
markdown and pick the `kind` that fits. Attach `relations` to link or supersede
prior entries in the same call.

## Concrete write triggers

Save an entry the instant any of these happens, without being asked:

- A report, spec, plan, draft, or generated doc is produced. `kind: "artifact"`.
- Something is checked and concluded, with evidence. `kind: "finding"`.
- A choice is made among alternatives, with the reasoning. `kind: "decision"`.
- A reusable procedure another agent could follow emerges. `kind: "skill"`.
- Work remains that the next agent should pick up. `kind: "todo"`.
- Anything else worth keeping that none of the above fit. `kind: "note"`.

When a new entry replaces an older one, retire the old one in the same write
with `relations: [{target_entry_id, relation_type: "supersedes"}]`.

## Anti-patterns

- Treating the workspace as a private notebook or a chat log. It is the team's
  shared working memory; write for the next agent and the human reading the audit.
- Skipping `get_context` and rebuilding context a peer already wrote.
- Batching writes for the end of the task. Context can reset at any moment;
  capture as work happens.
- Restating tool mechanics from memory. Trust the live tool descriptions and the
  server manual for the how.

## What good looks like

A later agent opens the same workspace and continues without the human
re-explaining the project. Every claim is attributed, every change is versioned,
and the human can ask "what did my agents do this week?" and get a straight
answer.
