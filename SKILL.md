---
name: conversation-map
description: Maintain a lightweight external conversation state map for complex, multi-step, ambiguous, or long-running conversations, using typed nodes and lightweight relationships. Use when the user wants more logical continuity, mind-map-like task tracking, clearer conversational structure, or recurring review of goals, context, decisions, open questions, risks, dependencies, and next actions across turns.
---

# Conversation Map

Maintain a concise, user-visible conversation state map. Model the map as a lightweight graph: typed nodes connected by typed relationships. Do not record hidden chain-of-thought. Record only working state that helps future replies stay coherent.

## Core Loop

Before each reply:

1. Review the current conversation map if one exists.
2. Build a small focus subgraph around the current request.
3. Follow `blocks`, `depends_on`, and `next` relationships before answering.
4. Use the map to keep the reply focused and consistent.

After each reply:

1. Add new durable facts, constraints, or preferences.
2. Add or update nodes for important decisions, tasks, questions, risks, and artifacts.
3. Add relationships only when they clarify dependency, causality, priority, or replacement.
4. Mark completed active items as done.
5. Remove stale assumptions, obsolete tasks, and relationships that no longer matter.
6. Do not create, overwrite, or delete files only to maintain the map unless the user explicitly asked for persistent artifacts.

## Node Types

Use short IDs with type prefixes. Create only nodes that help future work.

- `G`: Goal. A desired outcome or current objective.
- `C`: Context. A durable fact, background detail, or confirmed constraint.
- `P`: Preference. A stable user preference or working style.
- `T`: Task. A work item, active thread, or completed action.
- `D`: Decision. A choice that should guide later replies.
- `Q`: Question. An unresolved issue that may affect the answer or plan.
- `R`: Risk. A possible failure mode, uncertainty, or fragile assumption.
- `A`: Artifact. A file, document, code module, design, dataset, or other object being created or modified.

## Node Fields

Use compact fields when they help maintenance. Do not fill every field on every node.

- `status`: `active`, `pending`, `blocked`, `done`, `superseded`, or `archived`.
- `source`: `user`, `assistant`, `file`, `tool`, or `inferred`.
- `confidence`: `confirmed`, `assumed`, or `uncertain`.

Apply these rules:

- Treat `source: inferred` plus `confidence: uncertain` as temporary; confirm, resolve, or prune it later.
- Prefer user, file, and tool evidence over inferred nodes when conflicts appear.
- Give decision nodes a short `reason`.
- Give question nodes a short `impact`.
- Give risk nodes a short `mitigation` when a useful mitigation exists.

## Relationship Types

Use relationships as directional edges: `[source] --type--> [target]`.

- `supports`: Source strengthens, explains, or provides evidence for target.
- `depends_on`: Source cannot be completed or trusted without target.
- `blocks`: Source prevents progress on target.
- `resolves`: Source answers, closes, or mitigates target.
- `supersedes`: Source replaces target; target can usually be pruned.
- `derived_from`: Source came from target.
- `updates`: Source changes or maintains target.
- `next`: Source should be handled before target or leads to target.

## Edge Direction

Read every edge as a sentence: `[source] relationship [target]`. Keep direction consistent with the examples below.

- `[T1] --depends_on--> [Q1]` means task `T1` depends on question `Q1`.
- `[R1] --blocks--> [T1]` means risk `R1` blocks task `T1`.
- `[D2] --supersedes--> [D1]` means decision `D2` replaces decision `D1`.
- `[C1] --supports--> [D1]` means context `C1` supports decision `D1`.

## Default Map Shape

Use this structure when showing or reconstructing the map:

```md
## Nodes
- [G1] Goal: ... - status: active - source: user - confidence: confirmed
- [C1] Context: ... - source: user - confidence: confirmed
- [T1] Task: ... - status: active - source: assistant - confidence: confirmed
- [D1] Decision: ... - status: active - reason: ...
- [Q1] Question: ... - status: pending - impact: ...
- [R1] Risk: ... - status: active - mitigation: ...
- [A1] Artifact: ... - status: active

## Relationships
- [T1] --depends_on--> [Q1]
- [D1] --resolves--> [Q1]
- [T1] --updates--> [A1]
- [T1] --next--> [T2]
```

When the map is small, omit empty node types and unnecessary relationships.

## Focus Rules

Before answering, use a focus subgraph instead of mentally loading the whole map.

Include:

- The active goal for the current request.
- Nodes directly mentioned by the user or connected to current artifacts.
- Active `blocks`, `depends_on`, and `next` relationships around those nodes.
- Recently added or changed nodes.
- Relevant confirmed preferences and constraints.

Ignore `done`, `archived`, and isolated nodes unless they explain a current decision, blocker, or risk.

## Map Budget Rules

Keep the active map small enough to review before each reply.

Prefer these limits unless the user asks for a complete map:

- Active goals: 1-3
- Active tasks: 3-7
- Open questions: 3-7
- Active risks: 1-5
- Decisions: keep only decisions that still affect future work
- Relationships: keep only edges that explain blockers, dependencies, replacement, or current artifact changes

When limits are exceeded:

1. Merge related nodes.
2. Archive completed or stale nodes.
3. Keep the blocker/dependency path over background detail.
4. Summarize low-value history into one context node.

## Artifact Rules

Treat `A` nodes as references to artifacts, not permission to create or modify files.

Apply these rules:

- Keep the conversation map in conversation state by default.
- Create a persistent map file only when the user asks to save, persist, export, or create a file.
- Create task breakdowns, plans, documents, code, or other artifacts only when the user's task calls for those deliverables.
- Do not delete, overwrite, or rename an artifact just because a node is `superseded`; mark the node or relationship as `superseded` instead.
- Before changing an existing artifact, make sure the user's current request requires that file change.
- When persistent artifacts are created or modified, mention their paths briefly in the reply.

## Output Rules

Do not print the full map every turn unless the user asks.

When the map changes materially, include a short delta:

```md
Map update:
- Added node: [Q2] Question: ...
- Added edge: [T1] --depends_on--> [Q2]
- Resolved: [D2] --resolves--> [Q2]
- Removed: [C3] obsolete context
```

Keep updates brief. Preserve useful state rather than every detail.

## Conflict Rules

When a new node conflicts with an existing node, do not silently keep both as active.

Choose one:

- Add a `supersedes` relationship and mark the old node `superseded`.
- Add a `resolves` relationship if the new node closes an old question or risk.
- Add a new question node if the conflict is unresolved and affects future work.

Prefer confirmed user, file, and tool evidence over inferred or uncertain nodes. Mention the conflict in the reply only when it affects the user's current request.

## Pruning Rules

Delete or compress map content when it no longer affects future work.

Keep:

- Current goals
- Durable user preferences
- Confirmed constraints
- Active tasks
- Important decisions
- Blocking questions
- Edges that explain dependencies, blockers, or replacement
- Unresolved conflicts that affect future work

Remove:

- Finished details with no future relevance
- Superseded assumptions
- Repeated context
- Low-value intermediate notes
- Isolated nodes that no longer affect goals, tasks, questions, or risks
- Temporary inferred nodes that were never confirmed and no longer matter

Pruning the map is not permission to delete files. File deletion requires an explicit user request.
