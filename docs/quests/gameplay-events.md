# Quest gameplay events

`QuestSystem` does not poll your game. You tell it when something happens, and it
advances every active quest objective that matches the event.

## Event hooks

```csharp
quests.OnKill(enemyType);
quests.OnTalk(npcId);
quests.OnCollect("seal");
quests.OnCollect("coin", 10);
quests.OnReach("drowned_archive");
quests.OnDiscover("shrine_7");
quests.OnDungeonBossKilled(3);
```

| Method | Typical call site |
|---|---|
| `OnKill(int)` | Enemy death, after the enemy type is known |
| `OnTalk(int)` | Conversation start or first conversation with an NPC |
| `OnCollect(string, int)` | Inventory pickup; pass the amount picked up |
| `OnReach(string)` | Trigger volume or zone entry |
| `OnDiscover(string)` | Map reveal, lore discovery or first visit |
| `OnDungeonBossKilled(int)` | Dungeon boss death, with the dungeon number |

Only **active** quests are advanced. Calling an event for a quest that has not
started does nothing.

## Unity examples

### Enemy death

```csharp
public sealed class Enemy : MonoBehaviour
{
    public int enemyType;
    public NarrativeForgeBehaviour narrative;

    public void Die()
    {
        narrative.Quests.OnKill(enemyType);
    }
}
```

### Item pickup

```csharp
public sealed class Pickup : MonoBehaviour
{
    public string itemKey = "seal";
    public NarrativeForgeBehaviour narrative;

    private void OnTriggerEnter(Collider other)
    {
        if (!other.CompareTag("Player")) return;

        // Add to your inventory first, then tell the quest system.
        Inventory.Instance.Add(itemKey, 1);
        narrative.Quests.OnCollect(itemKey, 1);

        Destroy(gameObject);
    }
}
```

### Zone entry

```csharp
public sealed class ZoneTrigger : MonoBehaviour
{
    public string zoneKey = "drowned_archive";
    public NarrativeForgeBehaviour narrative;

    private void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            narrative.Quests.OnReach(zoneKey);
        }
    }
}
```

### Dungeon boss

```csharp
narrative.Quests.OnDungeonBossKilled(dungeonNumber);
```

Dungeon numbers are integers from `1` to `6`.

## Advancing a specific objective

Use `AdvanceObjective` when an objective does not map to a built-in event:

```csharp
// Quest 7, objective index 2, one step.
narrative.Quests.AdvanceObjective(7, 2, 1);
```

Objective indices are zero-based and follow the order in the quest JSON.

## Reconciling an inventory

If the player already owns an item when a collect quest starts, use
`ReconcileCollect`:

```csharp
narrative.Quests.StartQuest(2);
narrative.Quests.ReconcileCollect(2, new[] { "iron_key" });
```

It marks matching collect objectives complete for the owned item keys. This is
also useful after restoring a save when the inventory is authoritative.

## Starting follow-up quests

```csharp
private void OnEnable()
{
    narrative.Quests.QuestCompleted += HandleQuestCompleted;
}

private void HandleQuestCompleted(int questId, QuestDefinition quest)
{
    if (questId == 1)
    {
        narrative.Quests.StartQuest(2);
    }
}
```

Remember to unsubscribe in `OnDisable` or `OnDestroy`.

## Custom rewards

Assign a `RewardSink` to connect quest rewards to your inventory, stats, UI or
save system:

```csharp
narrative.Quests.RewardSink = (questId, quest) =>
{
    player.Gold += quest.Rewards.Gold;
    player.Xp += quest.Rewards.Xp;
    player.Reputation += quest.Rewards.Reputation;

    foreach (int itemId in quest.Rewards.Items)
    {
        inventory.Add(itemId);
    }

    foreach (string flag in quest.Rewards.Flags)
    {
        narrative.Dialogue.SetFlag(flag);
    }
};
```

`RewardSink` is a single delegate. Assigning your own replaces the default sink
installed by `NarrativeForgeBehaviour`. If you want to keep the default
gold/XP/reputation/flag behavior, either call the same logic in your sink or
leave **Apply Rewards To Dialogue State** enabled and only add item handling.

## Events

```csharp
narrative.Quests.QuestStarted += (id, quest) => { };
narrative.Quests.QuestCompleted += (id, quest) => { };
narrative.Quests.QuestFailed += (id, quest) => { };
narrative.Quests.QuestProgressChanged += (id, objective, progress) => { };
```

`QuestProgressChanged` fires after progress is clamped. The `progress` value is
the new progress for that objective.
