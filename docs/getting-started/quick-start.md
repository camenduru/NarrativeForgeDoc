# Quick start

This walkthrough builds a minimal conversation, a quest and a working uGUI
presenter. It takes about ten minutes.

## 1. Create the content files

Create a folder such as `Assets/Narrative/Data` and add two text assets.

### `dialogues.json`

```json
{
  "npcs": {
    "1": {
      "name": "Maren",
      "default_start": 1000,
      "dialogues": {
        "1000": {
          "id": 1000,
          "text": "You came back. The tide has been restless.",
          "choices": [
            { "id": 1001, "text": "Tell me about the seal.", "next": 1010 },
            { "id": 1002, "text": "Goodbye.", "next": 0 }
          ]
        },
        "1010": {
          "id": 1010,
          "text": "The seal keeps the archive drowned. Find it before the water turns.",
          "choices": [
            { "id": 1011, "text": "I will find it.", "next": 0, "flags": ["seal_quest_accepted"] }
          ]
        }
      }
    }
  }
}
```

### `quests.json`

```json
{
  "quests": [
    {
      "id": 1,
      "title": "The Drowned Seal",
      "type": "main",
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

The JSON parser is strict about syntax but forgiving about unknown fields. Use
the package's validation warnings to catch broken references.

## 2. Add the runtime component

1. Create an empty GameObject named `Narrative`.
2. Add **Narrative Forge** (`NarrativeForgeBehaviour`).
3. Assign `dialogues.json` to **Dialogue Json** and `quests.json` to
   **Quest Json**.
4. Leave **Load On Awake** enabled.

`NarrativeForgeBehaviour` loads both databases in `Awake` and exposes:

- `narrative.Dialogue` — the `DialogueRunner`
- `narrative.Quests` — the `QuestSystem`
- `narrative.Loaded` — an event fired after a successful load

## 3. Build the dialogue panel

Create a Canvas with an **EventSystem**. Under the Canvas, create a panel with:

- a speaker `Text`
- a body `Text`
- an empty `Choices Root` transform
- a `Button` named `Choice Template`
- a `Button` named `Continue`

Add **Dialogue View** to the panel and assign the fields. Set the choice template
inactive; the view clones it for each visible choice.

| Field | Assignment |
|---|---|
| Narrative | The `NarrativeForgeBehaviour` object |
| Panel | The dialogue panel |
| Speaker Label | Speaker `Text` |
| Body Label | Body `Text` |
| Choices Root | Empty transform under the panel |
| Choice Button Template | Inactive choice `Button` |
| Continue Button | Continue `Button` |

The view shows the panel when a conversation starts, rebuilds the choice list on
every node, disables locked choices and hides the panel when the conversation
ends.

## 4. Start a conversation

Add a small MonoBehaviour to your scene and call `Start`:

```csharp
using NarrativeForge;
using UnityEngine;

public sealed class TalkToMaren : MonoBehaviour
{
    public NarrativeForgeBehaviour narrative;

    public void Talk()
    {
        narrative.Dialogue.Start(1);
    }

    public void PickUpSeal()
    {
        narrative.Quests.OnCollect("seal");
    }
}
```

Wire `Talk()` to a character button or an interaction key. Wire `PickUpSeal()`
to the moment your game actually adds the seal to the inventory.

!!! tip "Start the quest"
    A conversation and a quest are independent. Start a quest with
    `narrative.Quests.StartQuest(1)` when the player accepts it.

## 5. Optional: add the quest log

Add a second panel with two `Text` labels and the **Quest Log View** component.
Assign the labels and the same `NarrativeForgeBehaviour`. The log rebuilds when a
quest starts, completes, fails or advances an objective.

## 6. Test

1. Press Play.
2. Call `narrative.Dialogue.Start(1)`.
3. Pick a reply and confirm the conversation advances.
4. Call `narrative.Quests.StartQuest(1)`.
5. Call `narrative.Quests.OnTalk(1)` and `narrative.Quests.OnCollect("seal")`.
6. Confirm the quest completes, the reward flags are applied and the quest log
   refreshes.

## Next steps

- [Scene setup](scene-setup.md) — the full component and hierarchy reference.
- [Dialogue JSON reference](../dialogue/json-reference.md) — every node and
  choice field.
- [Conditions](../dialogue/conditions.md) — flags, variables, choices and quest
  gates.
- [Quest gameplay events](../quests/gameplay-events.md) — connect kills, pickups,
  triggers and bosses.
