# Conditions

Conditions decide whether a choice is available. Every condition in a choice's
`conditions` array must pass (logical **AND**).

```json
{
  "id": 2001,
  "text": "A key for the drowned archive.",
  "next": 2010,
  "conditions": ["var:gold:>=:25"],
  "locked_hint": "Need 25 gold"
}
```

## Condition forms

| Form | Passes when |
|---|---|
| `flag` | The flag is set |
| `!flag` | The flag is **not** set |
| `met:npc` | The player has started a conversation with that NPC |
| `chose:npc:choice` | The player has selected that choice for that NPC |
| `var:name:>=:value` | Variable comparison |
| `quest:id:active` | Quest is in progress |
| `quest:id:completed` | Quest is completed |
| `quest:id:failed` | Quest is failed |
| `quest:id:started` | Quest has any recorded state |
| `quest:id:objective:n` | Objective `n` is complete |
| `quest:id` | Shorthand for `quest:id:completed` |

### Variables

The supported operators are `>=`, `<=`, `>`, `<`, `==`, `=` and `!=`.

```json
"conditions": [
  "var:gold:>=:25",
  "var:reputation:>0"
]
```

Variables default to `0` when they have never been set. Use `vars` and
`add_vars` on nodes, or `DialogueRunner.SetVar` / `AddVar` from code.

### `met:`

`met:npc` becomes true when `DialogueRunner.Start(npcId)` succeeds. It remains
true until the runner is reset or a save is restored without that NPC.

```json
"conditions": ["met:1"]
```

### `chose:`

`chose:npc:choice` reads the response history. Choice IDs are globally unique,
but the condition also names the NPC that owns the choice.

```json
"conditions": ["chose:1:1001"]
```

### Quest conditions

In a Unity scene, `NarrativeForgeBehaviour` automatically connects dialogue quest
conditions to its `QuestSystem`. If you use the core directly, connect it
yourself:

```csharp
var dialogue = new DialogueRunner(dialogueDatabase);
var quests = new QuestSystem(questDatabase);
dialogue.SetQuestCondition(quests.ConditionMet);
```

Quest conditions are only evaluated when a resolver is connected. Without one,
they fail.

## Hidden versus locked choices

The `!` prefix is a **hard gate**. A choice whose `!flag` condition fails is
removed from the visible list entirely, even if it has a `locked_hint`.

A choice that fails a normal condition is:

- **visible but disabled** when it has a `locked_hint`;
- **hidden** when it has no `locked_hint`.

This lets you choose between mystery and signposting:

```json
{
  "id": 3001,
  "text": "Open the sealed door.",
  "next": 3010,
  "conditions": ["has_archive_key"],
  "locked_hint": "Find the archive key"
}
```

The choice stays visible with the hint and cannot be selected until the flag is
set.

```json
{
  "id": 3002,
  "text": "Ask about the archive.",
  "next": 3020,
  "conditions": ["!archive_destroyed"]
}
```

If `archive_destroyed` is set, the choice disappears.

## Common patterns

### Reward unlocks dialogue

Quest 1 rewards `seal_found`:

```json
"rewards": { "flags": ["seal_found"] }
```

Maren's conditional start then uses node `1100`:

```json
"conditional_starts": [
  { "flag": "seal_found", "dialogue": 1100 }
]
```

### Gold gate with a visible hint

```json
"conditions": ["var:gold:>=:25"],
"locked_hint": "Need 25 gold"
```

### Quest gate

```json
"conditions": ["quest:1:active", "met:1"]
```

### Prior choice with an NPC check

```json
"conditions": ["chose:1:1001", "met:1"]
```

### Optional content that disappears

```json
"conditions": ["!seal_found"]
```

## Debugging conditions

- Use `DialogueRunner.ConditionMet("...")` in a test or debug UI.
- Inspect `DialogueRunner.Flags` and `Vars` while playing.
- Check the Console for validation warnings about flags that are never set.
- Remember that `quest:` conditions need a connected `QuestSystem`.
