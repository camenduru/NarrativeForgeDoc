# Dialogue JSON reference

The dialogue file is a single JSON object with one required member: `npcs`.

```json
{
  "npcs": {
    "1": {
      "name": "Maren",
      "default_start": 1000,
      "conditional_starts": [
        { "flag": "seal_found", "dialogue": 1100 }
      ],
      "default_image": "portraits/maren",
      "dialogues": {
        "1000": {
          "id": 1000,
          "text": "You came back.",
          "choices": [
            { "id": 1001, "text": "Tell me about the seal.", "next": 1010 },
            { "id": 1002, "text": "Goodbye.", "next": 0 }
          ]
        }
      }
    }
  }
}
```

## Top level

| Field | Type | Required | Description |
|---|---|---|---|
| `npcs` | object | yes | Map of NPC ID (string key) to NPC object |

NPC keys must parse as positive integers. Entries with invalid IDs are ignored.

## NPC fields

| Field | Type | Default | Description |
|---|---|---|---|
| `name` | string | empty | Speaker name used by `DialogueView` |
| `default_start` | integer | `0` | Entry node when no conditional start matches |
| `conditional_starts` | array | empty | Ordered flag-based entry points |
| `default_image` | string | empty | Optional image key returned by `CurrentNodeImage` |
| `dialogues` | object | empty | Map of node ID (string key) to node object |

An NPC with no usable nodes is ignored.

### `conditional_starts`

```json
"conditional_starts": [
  { "flag": "seal_found", "dialogue": 1100 },
  { "flag": "has_iron_key", "dialogue": 1200 }
]
```

| Field | Type | Description |
|---|---|---|
| `flag` | string | Flag to test |
| `dialogue` | integer | Node to start when the flag is set |

The first matching entry wins. Only flags are supported here; for richer gates,
start a node and use choice conditions.

## Node fields

| Field | Type | Default | Description |
|---|---|---|---|
| `id` | integer | `0` | Node ID; must be positive |
| `text` | string | empty | Body text |
| `choices` | array | empty | Choices shown at this node |
| `end` | boolean | `false` | End the conversation; no choices are shown |
| `keep_choices` | boolean | `false` | Hub mode: keep the previous node's choices visible after a reply |
| `flags` | array of strings | empty | Flags to set when the node is entered |
| `clear_flags` | array of strings | empty | Flags to remove when the node is entered |
| `vars` | object | empty | Variables to set when the node is entered |
| `add_vars` | object | empty | Variables to add when the node is entered |
| `audio` | string | empty | Optional audio key for your own resolver |
| `image` | string | empty | Optional image key for your own resolver |

`audio` and `image` are not loaded automatically. They are data hooks for your
own presentation code. `DialogueRunner.CurrentNodeImage` returns the node image,
falling back to the NPC's `default_image`.

## Choice fields

| Field | Type | Default | Description |
|---|---|---|---|
| `id` | integer | `0` | Globally unique choice ID; must be positive |
| `text` | string | empty | Player-facing label |
| `next` | integer | `-1` | Node to enter; `0` or a missing target closes the conversation |
| `conditions` | array of strings | empty | All conditions must pass |
| `flags` | array of strings | empty | Flags to set when the choice is selected |
| `locked_hint` | string | empty | Show the choice disabled with this hint when it is unavailable |

### `next` behavior

- `next > 0` and the node exists: enter that node.
- `next > 0` but the node does not exist: close the conversation.
- `next <= 0`: close the conversation.

Validation reports a `next` value that does not exist.

### Hubs and `keep_choices`

A hub is a node whose replies should not dismiss the original choices. Mark the
hub node with `"keep_choices": true`. When a reply has no choices of its own, the
runner shows the reply text and keeps the hub's choices visible.

```json
"1000": {
  "id": 1000,
  "text": "Ask me anything.",
  "keep_choices": true,
  "choices": [
    { "id": 1001, "text": "Tell me about the seal.", "next": 1010 },
    { "id": 1002, "text": "Where is the archive?", "next": 1020 },
    { "id": 1003, "text": "Goodbye.", "next": 0 }
  ]
},
"1010": {
  "id": 1010,
  "text": "The seal keeps the archive drowned.",
  "end": true
}
```

## Flags and variables

Flags are named booleans. Set them with `flags` on a node or choice, clear them
with `clear_flags`, or apply them as quest rewards.

Variables are named integers. Set or add them on a node:

```json
"vars": { "gold": 40, "reputation": 1 },
"add_vars": { "gold": -25 }
```

Read variables in conditions with `var:name:op:value`. Variables default to `0`.

## Tokens

`DialogueDatabase.TokenResolver` is an optional function that replaces `{token}`
markers in every string while loading:

```csharp
var database = new DialogueDatabase
{
    TokenResolver = key => localization.Get(key, string.Empty)
};
database.LoadFromJson(json);
```

Unresolved tokens are reported as validation warnings. If you do not need
tokens, leave the resolver unset.

## Complete example

```json
{
  "npcs": {
    "1": {
      "name": "Maren, Keeper of the Tide",
      "default_start": 1000,
      "conditional_starts": [
        { "flag": "seal_found", "dialogue": 1100 }
      ],
      "dialogues": {
        "1000": {
          "id": 1000,
          "text": "You came back. The tide has been restless.",
          "choices": [
            { "id": 1001, "text": "Tell me about the seal.", "next": 1010 },
            { "id": 1002, "text": "I should go.", "next": 0 }
          ]
        },
        "1010": {
          "id": 1010,
          "text": "The seal is the lock on the archive beneath the west jetty.",
          "choices": [
            {
              "id": 1011,
              "text": "I will find it.",
              "next": 1020,
              "flags": ["seal_quest_accepted"]
            }
          ]
        },
        "1020": {
          "id": 1020,
          "text": "Take the iron key from Bram. Bring the seal back before the water turns.",
          "flags": ["seal_quest_known"],
          "end": true
        },
        "1100": {
          "id": 1100,
          "text": "You found it. The tide is quiet for the first time in years.",
          "end": true
        }
      }
    }
  }
}
```

## Validation rules

The database warns about:

- duplicate NPC IDs
- missing `default_start`
- conditional starts that point at missing nodes
- duplicate choice IDs
- choices with no ID
- `next` links to missing nodes
- unreachable nodes
- conditions that reference unknown NPCs or choices
- flag conditions that no node or choice ever sets

See [Validation](../validation.md) for the full list.
