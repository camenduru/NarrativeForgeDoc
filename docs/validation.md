# Validation

Narrative Forge validates content while loading. Warnings are:

- printed to the Unity Console by `NarrativeForgeBehaviour`;
- available through `DialogueDatabase.Warnings` and `QuestDatabase.Warnings`;
- shown by the custom inspectors on the database assets.

Validation never throws on content problems. A database with warnings still
loads; a database with a fatal parse or shape error returns `false` and sets
`LastError`.

## Dialogue checks

| Check | Example warning |
|---|---|
| Duplicate NPC IDs | `NPC 1: duplicate NPC id` |
| Missing default start | `NPC 1: default_start 1000 has no matching dialogue` |
| Conditional start target | `conditional start 'seal_found' points at missing dialogue 1100` |
| Duplicate choice IDs | `duplicate choice id 1001` |
| Choice without an ID | `NPC 1 dialogue 1000: choice has no id` |
| Broken `next` link | `choice 1001: next 1010 has no matching dialogue` |
| Unknown `met:` NPC | `met:9 references an unknown NPC id` |
| Unknown `chose:` NPC | `chose:9:1001 references an unknown NPC id` |
| `chose:` choice belongs to another NPC | `chose:1:2001 references a choice that belongs to another NPC` |
| Flag never set | `condition 'seal_found' is never set by any node` |
| Unreachable node | `dialogue 1999 is not reachable from any entry point` |
| Unresolved token | `unresolved token '{npc.maren}' - check the token resolver` |

## Quest checks

| Check | Example warning |
|---|---|
| Duplicate quest IDs | `duplicate quest id 1` |
| Self prerequisite | `Quest 1: prereq references itself` |
| Missing prerequisite | `Quest 2: prereq quest 9 does not exist` |
| Talk target | `talk objective needs a valid NPC id target` |
| Collect/reach/discover target | `collect/reach/discover objective needs a target name` |
| Dungeon boss range | `dungeon_boss objective needs a dungeon number target (1-6)` |
| Negative kill target | `kill objective needs a valid enemy type target` |
| Invalid amount | `amount must be >= 1` |

## Fatal errors

These stop a load and set `LastError`:

- invalid JSON syntax
- missing `npcs` object
- no usable NPCs
- missing `quests` array
- no usable quests
- a missing or unassigned `TextAsset`

`NarrativeForgeBehaviour.Load()` logs the error with context and returns `false`.

## Using validation in CI

The core validators are engine-free. A test can load a JSON file and assert that
there are no warnings:

```csharp
[Test]
public void DialogueContentHasNoWarnings()
{
    var database = new DialogueDatabase();
    Assert.IsTrue(database.LoadFromFile("Assets/Narrative/Data/dialogues.json"));
    CollectionAssert.IsEmpty(database.Warnings);
}
```

The package's own EditMode tests use this pattern for the demo content.

## Fixing warnings

1. Open the Console and read the first warning.
2. Fix the referenced ID or field in the JSON.
3. Re-import or press Play to reload.
4. Repeat until the console is clean.

Validation is intentionally conservative. A warning about a flag that is never
set may be a false positive if the flag is set from code, a quest reward or a
save. You can keep such flags intentionally, but document why.
