# Scene setup

This page is the reference for wiring Narrative Forge into a scene. The sample
scene follows the same layout.

## Suggested hierarchy

```text
Narrative
└── NarrativeForgeBehaviour        dialogueJson, questJson

Canvas
├── DialoguePanel
│   ├── SpeakerLabel               Text
│   ├── BodyLabel                  Text
│   ├── ChoicesRoot                Transform
│   │   └── ChoiceTemplate         Button (inactive)
│   ├── ContinueButton             Button
│   └── DialogueView
└── QuestPanel
    ├── QuestLabel                 Text
    ├── StatusLabel                Text
    └── QuestLogView

EventSystem
└── Input module                   InputSystemUIInputModule or StandaloneInputModule
```

The `NarrativeForgeBehaviour` does not have to be on the Canvas. Putting it on a
separate object keeps the runtime and the presentation separate.

## NarrativeForgeBehaviour

| Field | Default | Purpose |
|---|---|---|
| Dialogue Json | none | Text asset containing the dialogue JSON |
| Quest Json | none | Text asset containing the quest JSON |
| Load On Awake | `true` | Load both databases when the component wakes |
| Apply Rewards To Dialogue State | `true` | Apply reward flags, gold, XP and reputation to the dialogue state |

If **Load On Awake** is disabled, call `Load()` yourself after assigning the
assets. `Load()` returns `false` and sets `LastError` if either asset is missing
or invalid.

`NarrativeForgeBehaviour` creates the `DialogueRunner` and `QuestSystem`, wires
the quest condition resolver and raises `Loaded` after a successful load.

## DialogueView

| Field | Purpose |
|---|---|
| Narrative | The `NarrativeForgeBehaviour` that owns the conversation |
| Panel | Root object to show and hide |
| Speaker Label | `Text` for the NPC name |
| Body Label | `Text` for the current node text |
| Choices Root | Parent transform for generated choice buttons |
| Choice Button Template | Inactive `Button` cloned for each choice |
| Continue Button | Button shown when the node has no choices |

The view subscribes to `ConversationStarted`, `ConversationEnded` and
`NodeEntered`. It clears and rebuilds choice buttons on every node, so the
template should be inactive and should not be reused as a live button.

A choice is shown when it is available, or when it has a `locked_hint` and is
unavailable. A choice that fails a hard `!flag` gate is hidden instead. See
[Conditions](../dialogue/conditions.md).

## QuestLogView

| Field | Purpose |
|---|---|
| Narrative | The `NarrativeForgeBehaviour` that owns the quests |
| Quest Label | `Text` that receives the journal body |
| Status Label | Optional `Text` for gold and XP |
| No Quests Text | Text shown when no quests are active |
| Show Completed Count | Append a completed-quest count |

The journal shows active main and side quests, objective progress and a completed
count. It refreshes on `QuestStarted`, `QuestCompleted`, `QuestFailed` and
`QuestProgressChanged`.

## Input

The runtime is input-agnostic. The sample uses a small `DemoInputModule` that
adds:

- `InputSystemUIInputModule` when the Input System package is active;
- `StandaloneInputModule` otherwise.

For your own UI, add the module that matches your project's **Active Input
Handling** setting. The package does not require the Input System package.

## Render pipeline

The runtime and sample have no render-pipeline dependency. They work with the
Built-in Render Pipeline, URP and HDRP. The sample uses screen-space overlay UI,
so it looks the same in every pipeline.

## Loading order

1. `Awake` — `NarrativeForgeBehaviour` loads the JSON databases.
2. `Start` — `DialogueView` and `QuestLogView` bind to the runtime.
3. `Loaded` — views that were created later can bind when this event fires.

If you instantiate a view after the load has already happened, call `Bind()`
manually or subscribe to `Loaded` before the load.
