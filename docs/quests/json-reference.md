# Quest JSON reference

The quest file is a single JSON object with one required member: `quests`.

```json
{
  "quests": [
    {
      "id": 1,
      "title": "The Drowned Seal",
      "description": "Recover the seal before the tide turns.",
      "type": "main",
      "prereqs": [],
      "objectives": [
        { "type": "talk", "target": 1, "text": "Speak with Maren" },
        { "type": "collect", "target": "seal", "text": "Recover the seal" }
      ],
      "rewards": {
        "gold": 50,
        "xp": 100,
        "flags": ["seal_found"]
      }
    }
  ]
}
```

## Quest fields

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | integer | yes | Positive, unique quest ID |
| `title` | string | no | Display name |
| `description` | string | no | Journal or tooltip text |
| `type` | string | no | `main` or `sub`; anything else becomes `sub` |
| `prereqs` | array of integers | no | Quest IDs that must be completed first |
| `objectives` | array | yes | At least one objective is required |
| `rewards` | object | no | Rewards applied on completion |

A quest with no positive ID or no objectives is ignored during load.

## Objective fields

| Field | Type | Default | Description |
|---|---|---|---|
| `type` | string | `kill` | `kill`, `talk`, `collect`, `reach`, `discover` or `dungeon_boss` |
| `target` | integer or string | none | Numeric target for `kill`, `talk`, `dungeon_boss`; string key for `collect`, `reach`, `discover` |
| `amount` | integer | `1` | Required count; values below 1 become 1 |
| `text` | string | empty | Objective text for the journal |

### Objective types

| Type | Target | Notes |
|---|---|---|
| `kill` | integer enemy type | Advanced by `OnKill(enemyType)` |
| `talk` | positive NPC ID | Advanced by `OnTalk(npcId)` |
| `collect` | string item key | Advanced by `OnCollect(name, amount)` |
| `reach` | string zone key | Advanced by `OnReach(zone)` |
| `discover` | string place key | Advanced by `OnDiscover(place)` |
| `dungeon_boss` | integer 1–6 | Advanced by `OnDungeonBossKilled(number)` |

For `collect`, `reach` and `discover`, put the string in `target`:

```json
{ "type": "collect", "target": "seal", "amount": 1, "text": "Recover the seal" }
```

For numeric targets, use a number:

```json
{ "type": "talk", "target": 1, "text": "Speak with Maren" }
```

## Reward fields

| Field | Type | Description |
|---|---|---|
| `gold` | integer | Added to the `gold` dialogue variable by the default sink |
| `xp` | integer | Added to the `xp` dialogue variable by the default sink |
| `reputation` | integer | Added to the `reputation` dialogue variable by the default sink |
| `items` | array of integers | Item IDs; your custom `RewardSink` decides what to do |
| `flags` | array of strings | Flags set on the dialogue runner by the default sink |

The default sink is installed by `NarrativeForgeBehaviour` when **Apply Rewards
To Dialogue State** is enabled. It does not touch an inventory. See
[Gameplay events](gameplay-events.md) for a custom sink.

## Multiple objectives

All objectives must complete. Use `amount` for counts:

```json
"objectives": [
  { "type": "kill", "target": 3, "amount": 5, "text": "Defeat five drowned" },
  { "type": "reach", "target": "west_jetty", "text": "Reach the west jetty" }
]
```

`OnKill(3)` advances the first objective by one each time, up to five.

## Prerequisites

```json
{
  "id": 2,
  "title": "Iron for the Road",
  "prereqs": [1],
  "objectives": [
    { "type": "collect", "target": "iron_key", "text": "Buy the iron key" }
  ]
}
```

Quest 2 cannot start until quest 1 is completed. `StartQuest(2)` returns `false`
otherwise.

## Validation rules

The quest database warns about:

- duplicate quest IDs
- self-referencing prerequisites
- prerequisites that do not exist
- `talk` objectives without a positive NPC ID
- `collect`, `reach` and `discover` objectives without a target name
- `dungeon_boss` objectives outside 1–6
- negative kill targets
- amounts below 1

See [Validation](../validation.md).
