# Conversation Map Skill

A lightweight Codex skill for maintaining a user-visible conversation state map during complex, multi-step, ambiguous, or long-running conversations.

The skill tracks goals, context, preferences, tasks, decisions, open questions, risks, artifacts, and their relationships without recording hidden chain-of-thought.

## What It Does

- Maintains a compact external conversation map.
- Uses typed nodes such as `G`, `C`, `P`, `T`, `D`, `Q`, `R`, and `A`.
- Uses typed relationships such as `supports`, `depends_on`, `blocks`, `resolves`, `supersedes`, `derived_from`, `updates`, and `next`.
- Keeps the active map small with map budget rules.
- Avoids creating, overwriting, or deleting files just to maintain the map.
- Shows full maps only when requested.

## Installation

Copy this folder into your Codex skills directory:

```powershell
Copy-Item -Recurse -Force .\conversation-map C:\Users\<you>\.codex\skills\conversation-map
```

Or copy just the published folder contents so the final structure is:

```text
C:\Users\<you>\.codex\skills\conversation-map\
  SKILL.md
  agents\
    openai.yaml
```

Restart Codex or open a new thread if the skill is not discovered immediately.

## Usage

Explicit invocation is the most reliable:

```text
Use $conversation-map to keep a concise state map while we work through this multi-step task.
```

The skill may also trigger implicitly when a request asks for logical continuity, mind-map-like task tracking, or recurring review of goals, decisions, blockers, risks, dependencies, and next actions.

## Notes

This skill is guidance for Codex, not a hard runtime policy. For mechanical enforcement of file operations or other strict constraints, use hooks, scripts, or repository policy in addition to the skill.

## License

MIT
