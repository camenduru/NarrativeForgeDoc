# Quest overview

`QuestSystem` tracks quest state and advances typed objectives from gameplay
events. It is engine-free, serialisable and independent of the dialogue runner.

## Lifecycle

```text
NotStarted ──StartQuest()──► InProgress ──all objectives complete──► Completed
                                   │
                                   └──FailQuest()──► Failed
```

| State | Meaning |
|---|---|
| `NotStarted` | The quest has no recorded state |
| `InProgress` | The quest is active |
| `Completed` | Every objective reached its required amount |
| `Failed` | The quest was explicitly failed |

A completed quest cannot be started again. `StartQuest` returns `false` if the
quest is unknown, already has state, or has an incomplete prerequisite.

## Prerequisites

```json
"prereqs": [1]
```

Every listed quest must be `Completed` before `StartQuest` succeeds. A quest
cannot list itself. Validation reports missing prerequisites and self-references.

## Objectives

A quest is complete when **every** objective reaches its required `amount`.
Progress is clamped to the required amount, so calling an event twice does not
overshoot.

| Objective | Target | Event |
|---|---|---|
| `talk` | NPC ID | `OnTalk(npcId)` |
| `kill` | Enemy type | `OnKill(enemyType)` |
| `collect` | Item key | `OnCollect(name, amount)` |
| `reach` | Zone key | `OnReach(zone)` |
| `discover` | Place key | `OnDiscover(place)` |
| `dungeon_boss` | Dungeon number 1–6 | `OnDungeonBossKilled(number)` |

You can also advance a specific objective directly with
`AdvanceObjective(questId, objectiveIndex, amount)`. That is useful for custom
objective types or scripted sequences.

## Rewards

Rewards are defined on the quest and fire exactly once, when the quest completes.

```json
"rewards": {
  "gold": 50,
  "xp": 100,
  "reputation": 1,
  "items": [101, 102],
  "flags": ["seal_found"]
}
```

`QuestSystem` does not know about your inventory or stats. It calls a
`RewardSink` delegate:

```csharp
quests.RewardSink = (questId, quest) =>
{
    // Apply gold, XP, items and flags to your game.
};
```

`NarrativeForgeBehaviour` installs a default sink when
**Apply Rewards To Dialogue State** is enabled. That sink:

- sets every reward flag on the dialogue runner;
- adds reward gold to the `gold` variable;
- adds reward XP to the `xp` variable;
- adds reward reputation to the `reputation` variable.

It does **not** add reward items to an inventory. Assign your own `RewardSink` if
you need item rewards, or to replace the default behavior.

!!! warning "One sink at a time"
    `RewardSink` is a single delegate, not an event. Assigning a custom sink
    replaces the default. Either apply the dialogue-state rewards yourself or
    leave **Apply Rewards To Dialogue State** disabled.

## Events

| Event | Signature |
|---|---|
| `QuestStarted` | `(int questId, QuestDefinition quest)` |
| `QuestCompleted` | `(int questId, QuestDefinition quest)` |
| `QuestFailed` | `(int questId, QuestDefinition quest)` |
| `QuestProgressChanged` | `(int questId, int objectiveIndex, int progress)` |

Use `QuestCompleted` to start follow-up quests, open a door, play a cutscene or
refresh a journal.

## Active quests

```csharp
List<QuestDefinition> main = quests.ActiveQuests(QuestType.Main);
List<QuestDefinition> side = quests.ActiveQuests(QuestType.Sub);
```

`QuestLogView` uses these methods to build the journal.

## Save state

`QuestSystem.CaptureSave()` stores each quest's status and objective progress.
`RestoreSave()` clears current state and restores the saved entries. See
[Save and load](../save-load.md).
