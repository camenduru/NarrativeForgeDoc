# Dialogue examples

These examples are intentionally small. Combine them into a real conversation by
linking nodes with `next`.

## Conditional greeting

Maren has a different opening after the seal is found.

```json
"conditional_starts": [
  { "flag": "seal_found", "dialogue": 1100 }
],
"default_start": 1000
```

`conditional_starts` is checked in order. The first flag that is set wins.

## Locked choice with a hint

The player can see the key option but cannot buy it yet.

```json
{
  "id": 2001,
  "text": "A key for the drowned archive.",
  "next": 2010,
  "conditions": ["var:gold:>=:25"],
  "locked_hint": "Need 25 gold"
}
```

Without `locked_hint`, the choice would be hidden until the condition passed.

## Hidden choice

A choice that should not exist after a story event uses a hard gate.

```json
{
  "id": 2002,
  "text": "Ask about the archive.",
  "next": 2020,
  "conditions": ["!archive_destroyed"]
}
```

## Hub conversation

A hub keeps its choices visible while the player reads replies.

```json
"3000": {
  "id": 3000,
  "text": "Ask me anything.",
  "keep_choices": true,
  "choices": [
    { "id": 3001, "text": "What is the archive?", "next": 3010 },
    { "id": 3002, "text": "Who built it?", "next": 3020 },
    { "id": 3003, "text": "Goodbye.", "next": 0 }
  ]
},
"3010": {
  "id": 3010,
  "text": "A library that sank with its doors locked.",
  "end": true
}
```

The reply nodes use `"end": true` and no choices. Because the hub has
`keep_choices`, the runner shows the reply and restores the hub choices.

## Quest gate

Bram will only discuss the seal after Maren has sent the player and quest 1 is
active.

```json
{
  "id": 2003,
  "text": "Tell me about the seal.",
  "next": 2030,
  "conditions": ["met:1", "quest:1:active"]
}
```

## Buying an item with gold

The choice checks the price, and the reply node deducts the gold and grants the
key.

```json
"2010": {
  "id": 2010,
  "text": "Twenty-five gold. It opens the lower door, not the sea.",
  "choices": [
    { "id": 2011, "text": "Take the gold.", "next": 2012, "flags": ["paid_for_key"] },
    { "id": 2012, "text": "Keep it.", "next": 0 }
  ]
},
"2012": {
  "id": 2012,
  "text": "The key is yours. If the archive takes it, don't follow.",
  "flags": ["has_iron_key"],
  "add_vars": { "gold": -25 },
  "end": true
}
```

The choice sets `paid_for_key`; the destination node grants `has_iron_key` and
removes the gold. In a real game, you would also call your inventory system when
the choice is selected.

## A complete short conversation

```json
{
  "npcs": {
    "1": {
      "name": "Maren",
      "default_start": 1000,
      "dialogues": {
        "1000": {
          "id": 1000,
          "text": "The tide is restless.",
          "choices": [
            { "id": 1001, "text": "What is the seal?", "next": 1010 },
            { "id": 1002, "text": "Goodbye.", "next": 0 }
          ]
        },
        "1010": {
          "id": 1010,
          "text": "The lock on the archive. Find it before the water turns.",
          "choices": [
            { "id": 1011, "text": "I will find it.", "next": 1020, "flags": ["seal_quest_accepted"] },
            { "id": 1012, "text": "Not yet.", "next": 0 }
          ]
        },
        "1020": {
          "id": 1020,
          "text": "Take the iron key from Bram.",
          "end": true
        }
      }
    }
  }
}
```
