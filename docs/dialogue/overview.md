# Dialogue overview

A dialogue database is a set of **NPCs**, each owning a set of **nodes**. A node
contains text and, optionally, choices. A choice can jump to another node, set a
flag, or end the conversation.

The runtime is intentionally small:

- `DialogueDatabase` parses and validates the JSON.
- `DialogueRunner` tracks one active conversation, flags, variables, responses
  and seen nodes.
- `DialogueView` is an optional uGUI presenter.

## The conversation model

```text
Start(npcId)
    │
    ▼
Entry node (default_start or first matching conditional_start)
    │
    ├── choices? ──► choose ──► next node / close
    │
    └── no choices ──► Continue() ──► close
```

A node with `"end": true` never offers choices. A node with `keep_choices`
behaves as a hub: after a reply, the previous node's choices stay visible while
the reply text is shown.

## IDs

- **NPC IDs** are positive integers and are the keys under `npcs`.
- **Node IDs** are positive integers. They are unique inside one NPC.
- **Choice IDs** are positive integers and must be unique across the whole
  dialogue database.

Keep IDs stable. Save data stores NPC IDs, node IDs and choice IDs, so changing
them breaks existing saves.

## Entry points

Each NPC has:

- `default_start` — the node used when no conditional start matches.
- `conditional_starts` — an ordered list of `{ "flag": "...", "dialogue": id }`
  entries. The first entry whose flag is set wins.

If `default_start` is missing or points at a node that does not exist, the runner
falls back to the lowest node ID for that NPC. Validation reports the broken
reference anyway.

## State

The runner keeps:

| State | Meaning |
|---|---|
| Flags | Named booleans set by nodes, choices and quest rewards |
| Variables | Named integers used by `var:` conditions |
| Responses | Which choices the player picked, per NPC |
| Met NPCs | NPCs the player has started a conversation with |
| Seen nodes | Nodes that have been entered |

All of this is included in `DialogueSaveData`.

## Events

| Event | Fires when |
|---|---|
| `ConversationStarted` | `Start()` successfully enters the first node |
| `NodeEntered` | A node is entered, including the first node |
| `ChoiceSelected` | The player selects an available choice |
| `ConversationEnded` | The conversation closes or the last node ends |

`DialogueView` uses these events to show and hide the panel and rebuild choices.
Your gameplay code can subscribe to them for animation, audio, camera changes or
analytics.

## Conditions are the glue

Dialogue can read game state through conditions:

- flags set by earlier choices or quest rewards
- prior choices (`chose:npc:choice`)
- NPCs already met (`met:npc`)
- integer variables (`var:gold:>=:25`)
- quest state (`quest:1:active`)

That is how a quest reward unlocks a new conversation, or how a shop choice stays
locked until the player has enough gold. See [Conditions](conditions.md).

## Authoring recommendations

- Give each NPC a stable, meaningful ID.
- Use a separate numeric range per NPC for node IDs (`1000`, `2000`, …) to keep
  hand-authored JSON readable.
- Keep choice IDs globally unique, even across NPCs.
- Write the first node as a short greeting; put long exposition behind a choice.
- Prefer flags for story facts and variables for numbers.
- Run validation after every content change.

Next: [Dialogue JSON reference](json-reference.md).
