# Save and load

Narrative Forge separates narrative state from your game's save format. You can
serialize the narrative payload with Unity's `JsonUtility`, your own serializer,
or store it inside a larger save object.

## What is saved

`NarrativeSaveData` contains:

| Member | Contents |
|---|---|
| `Version` | Save format version, currently `1` |
| `Dialogue` | Flags, variables, responses, met NPCs and seen nodes |
| `Quests` | Quest status and objective progress |

Dialogue save data includes:

- every flag
- every integer variable
- every recorded response (`NPC`, node, choice)
- every met NPC
- every seen node

Quest save data includes one entry per quest with a status and a progress list.

## Capture

```csharp
using NarrativeForge;
using UnityEngine;

public sealed class SaveExample : MonoBehaviour
{
    public NarrativeForgeBehaviour narrative;

    public string Capture()
    {
        NarrativeSaveData save = narrative.CaptureSave();
        return JsonUtility.ToJson(save, true);
    }
}
```

`NarrativeForgeBehaviour.CaptureSave()` returns a new payload. It does not write a
file.

## Restore

```csharp
public void Restore(string json)
{
    NarrativeSaveData save = JsonUtility.FromJson<NarrativeSaveData>(json);
    narrative.RestoreSave(save);
}
```

`RestoreSave`:

- clears the current dialogue state and closes any active conversation;
- restores flags, variables, responses, met NPCs and seen nodes;
- clears and restores quest status and progress.

It does not fire `Loaded`, so views may need a refresh:

```csharp
narrative.RestoreSave(save);
dialogueView.Bind();
questLog.Bind();
```

`Bind()` unsubscribes and resubscribes, then refreshes the presenter.

## Saving during a conversation

`CaptureSave` records the conversation's persistent state, not the currently open
node. On restore, the conversation is closed; start it again with
`Dialogue.Start(npcId)` if you want the player to resume talking.

If you need to resume the exact open node, store the NPC and node IDs in your own
save wrapper and call `Start` for the NPC. The runner will choose the entry point;
there is no public "jump to node" API by design.

## Versioning

The `Version` field is reserved for future migrations. Check it before restoring:

```csharp
if (save.Version != 1)
{
    // Migrate or reject the save.
}
```

## Core-only use

If you use the core without `NarrativeForgeBehaviour`:

```csharp
DialogueSaveData dialogueSave = dialogue.CaptureSave();
QuestSaveData questSave = quests.CaptureSave();

dialogue.RestoreSave(dialogueSave);
quests.RestoreSave(questSave);
```

`DialogueRunner.RestoreSave` and `QuestSystem.RestoreSave` replace the current
state. They do not merge.

## Serialization notes

- The save classes are `[Serializable]` and use public fields, so
  `JsonUtility` works without custom converters.
- Lists are sorted where practical (flags, variables, quest entries) to keep
  output stable and diff-friendly.
- Do not serialize live `DialogueRunner` or `QuestSystem` objects. Capture a
  `NarrativeSaveData` payload instead.
- IDs are the contract. Changing NPC, node, choice or quest IDs invalidates old
  saves.
