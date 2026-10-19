# Unity components

The package has three runtime components and two optional database assets. All
of them live in the `NarrativeForge` namespace.

## NarrativeForgeBehaviour

The scene entry point. It loads the dialogue and quest databases and exposes
them to your code.

```csharp
[AddComponentMenu("Narrative Forge/Narrative Forge")]
public sealed class NarrativeForgeBehaviour : MonoBehaviour
```

| Member | Type | Description |
|---|---|---|
| `dialogueJson` | `TextAsset` | Dialogue JSON |
| `questJson` | `TextAsset` | Quest JSON |
| `loadOnAwake` | `bool` | Load in `Awake` |
| `applyRewardsToDialogueState` | `bool` | Install the default reward sink |
| `Dialogue` | `DialogueRunner` | Runtime conversation state |
| `Quests` | `QuestSystem` | Runtime quest state |
| `LastError` | `string` | Last load error |
| `Loaded` | `event Action` | Raised after a successful load |

Methods:

- `bool Load()` — parse both assets and create the runners. Logs errors and
  returns `false` on failure.
- `NarrativeSaveData CaptureSave()` — capture both systems.
- `void RestoreSave(NarrativeSaveData)` — restore both systems.

The default reward sink sets reward flags and adds gold, XP and reputation to the
dialogue variables. It does not add item rewards to an inventory.

## DialogueView

Optional uGUI presenter for a conversation.

```csharp
[AddComponentMenu("Narrative Forge/Dialogue View")]
public sealed class DialogueView : MonoBehaviour
```

| Field | Description |
|---|---|
| `narrative` | Runtime owner; falls back to a parent component |
| `panel` | Root object shown during a conversation |
| `speakerLabel` | NPC name |
| `bodyLabel` | Node text |
| `choicesRoot` | Parent for generated choice buttons |
| `choiceButtonTemplate` | Inactive button cloned per choice |
| `continueButton` | Button shown when there are no choices |

Methods:

- `void Bind()` — subscribe to the current runner and refresh.
- `void Refresh()` — rebuild the current view from runner state.

The view displays a choice when it is available, or when it is unavailable but
has a `locked_hint`. It hides choices blocked by a hard `!flag` gate.

## QuestLogView

Optional read-only journal.

```csharp
[AddComponentMenu("Narrative Forge/Quest Log View")]
public sealed class QuestLogView : MonoBehaviour
```

| Field | Description |
|---|---|
| `narrative` | Runtime owner; falls back to a parent component |
| `questLabel` | Journal body |
| `statusLabel` | Optional gold and XP line |
| `noQuestsText` | Text when there are no active quests |
| `showCompletedCount` | Append the completed-quest count |

Methods:

- `void Bind()` — subscribe and refresh.
- `void Refresh()` — rebuild the journal text.

The journal groups active **main** and **side** quests, prints objective
progress, and can show a completed count.

## DialogueDatabaseAsset

A `ScriptableObject` wrapper around a dialogue JSON file. Create one with
**Assets → Create → Narrative Forge → Dialogue Database**.

| Member | Description |
|---|---|
| `json` | Assigned `TextAsset` |
| `Database` | Lazily built `DialogueDatabase` |
| `LastError` | Build error |
| `Build()` | Parse and validate the JSON |

The custom inspector runs validation and shows warnings in the Inspector and
Console. This is useful for authoring and CI checks.

## QuestDatabaseAsset

The quest equivalent of `DialogueDatabaseAsset`.

| Member | Description |
|---|---|
| `json` | Assigned `TextAsset` |
| `Database` | Lazily built `QuestDatabase` |
| `LastError` | Build error |
| `Build()` | Parse and validate the JSON |

## Sample-only components

The Harbour Demo ships with sample scripts that are not part of the runtime API:

- `DemoBootstrap` — wires the demo UI, starts quest 1, connects quest events and
  applies rewards.
- `DemoInputModule` — selects `InputSystemUIInputModule` or
  `StandaloneInputModule` at runtime.

Copy their patterns into your own project or replace them with your own
presenters. They are intentionally kept outside the runtime assembly.
