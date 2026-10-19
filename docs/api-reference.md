# API reference

The runtime API lives in the `NarrativeForge` namespace. The core classes have no
Unity dependency; the components are thin adapters.

## DialogueDatabase

Parses and validates a dialogue JSON file.

| Member | Description |
|---|---|
| `Func<string, string> TokenResolver` | Optional `{token}` resolver run over every string during load |
| `string LastError` | Last parse or load error |
| `IReadOnlyList<DialogueNpc> Npcs` | Parsed NPCs |
| `IReadOnlyList<string> Warnings` | Validation warnings |
| `bool HasNpcs` | True when at least one NPC loaded |
| `bool LoadFromFile(string path)` | Load and parse a file |
| `bool LoadFromJson(string json)` | Parse and validate JSON |
| `DialogueNpc GetNpc(int npcId)` | Find an NPC by ID |
| `DialogueNode GetDialogue(int npcId, int nodeId)` | Find a node |
| `DialogueNode GetFirstDialogue(DialogueNpc npc)` | Lowest node ID for an NPC |
| `List<string> Validate()` | Run validation and return warnings |

## DialogueRunner

Drives one active conversation.

### State

| Member | Description |
|---|---|
| `DialogueDatabase Database` | Source database |
| `bool IsActive` | A conversation is open |
| `int ActiveNpcId` | Current NPC ID |
| `int CurrentNodeId` | Current node ID |
| `int SelectedIndex` | Selected visible choice index |
| `IReadOnlyList<DialogueChoice> VisibleChoices` | Choices currently shown |
| `IReadOnlyCollection<string> Flags` | Set flags |
| `IReadOnlyDictionary<string, int> Vars` | Integer variables |
| `DialogueNpc ActiveNpc` | Current NPC or `null` |
| `DialogueNode CurrentNode` | Current node or `null` |
| `string CurrentNodeImage` | Node image, then NPC default image |

### Events

| Event | Signature |
|---|---|
| `ConversationStarted` | `Action<DialogueNpc>` |
| `ConversationEnded` | `Action<DialogueNpc>` |
| `NodeEntered` | `Action<DialogueNpc, DialogueNode>` |
| `ChoiceSelected` | `Action<DialogueChoice>` |

### Methods

| Method | Description |
|---|---|
| `DialogueRunner(DialogueDatabase)` | Create a runner |
| `void SetQuestCondition(Func<string, bool>)` | Connect `quest:` conditions |
| `bool Start(int npcId)` | Start an NPC conversation |
| `void Close()` | Close the active conversation |
| `void MoveSelection(int direction)` | Move the selection, skipping unavailable choices |
| `void SelectCurrent()` | Select the current choice or continue |
| `void SelectVisibleChoice(int index)` | Select by visible index |
| `void Continue()` | Close when there are no choices |
| `bool ChoiceAvailable(DialogueChoice)` | All conditions pass |
| `bool IsChoiceLocked(DialogueChoice)` | Unavailable but has a `locked_hint` |
| `bool HiddenByStateGate(DialogueChoice)` | Fails a hard `!flag` gate |
| `bool ConditionMet(string)` | Evaluate one condition |
| `bool HasFlag(string)` | Flag lookup |
| `void SetFlag(string)` | Set a flag |
| `int GetVar(string)` | Read an integer variable |
| `void SetVar(string, int)` | Set an integer variable |
| `void AddVar(string, int)` | Add to an integer variable |
| `void RecordResponse(int npcId, int nodeId, int choiceId)` | Record a choice |
| `IReadOnlyList<Response> Responses(int npcId)` | Responses for an NPC |
| `bool HasChosen(int npcId, int choiceId)` | Prior choice lookup |
| `void RecordSeenDialogue(int npcId, int nodeId)` | Mark a node seen |
| `IReadOnlyList<int> SeenDialogues(int npcId)` | Seen node IDs |
| `bool HasMet(int npcId)` | Conversation history lookup |
| `void Reset()` | Close and clear all state |
| `DialogueSaveData CaptureSave()` | Capture dialogue state |
| `void RestoreSave(DialogueSaveData)` | Replace dialogue state |

`DialogueRunner.Response` has `DialogueId` and `ChoiceId` fields.

## QuestDatabase

Parses and validates a quest JSON file.

| Member | Description |
|---|---|
| `string LastError` | Last parse or load error |
| `IReadOnlyList<QuestDefinition> Quests` | Parsed quests |
| `IReadOnlyList<string> Warnings` | Validation warnings |
| `bool HasQuests` | True when at least one quest loaded |
| `bool LoadFromFile(string path)` | Load and parse a file |
| `bool LoadFromJson(string json)` | Parse and validate JSON |
| `QuestDefinition GetQuest(int questId)` | Find a quest |
| `List<string> Validate()` | Run validation and return warnings |

## QuestSystem

Tracks quest state and advances objectives.

### State and events

| Member | Description |
|---|---|
| `QuestDatabase Database` | Source database |
| `Action<int, QuestDefinition> RewardSink` | Called once when a quest completes |
| `QuestStarted` | `Action<int, QuestDefinition>` |
| `QuestCompleted` | `Action<int, QuestDefinition>` |
| `QuestFailed` | `Action<int, QuestDefinition>` |
| `QuestProgressChanged` | `Action<int, int, int>` (quest, objective, progress) |

### Methods

| Method | Description |
|---|---|
| `QuestSystem(QuestDatabase)` | Create a system |
| `bool StartQuest(int questId)` | Start a quest if unknown/prereqs allow |
| `void FailQuest(int questId)` | Mark an in-progress quest failed |
| `void CompleteQuest(int questId)` | Complete an in-progress quest and fire rewards |
| `QuestStatus Status(int questId)` | Current status |
| `bool IsActive(int questId)` | Status is `InProgress` |
| `bool IsCompleted(int questId)` | Status is `Completed` |
| `bool HasStarted(int questId)` | Any state exists |
| `int ObjectiveProgress(int questId, int index)` | Progress for one objective |
| `List<QuestDefinition> ActiveQuests(QuestType type)` | Active main or side quests |
| `void OnKill(int enemyType)` | Advance kill objectives |
| `void OnTalk(int npcId)` | Advance talk objectives |
| `void OnCollect(string name, int amount = 1)` | Advance collect objectives |
| `void OnReach(string zone)` | Advance reach objectives |
| `void OnDiscover(string place)` | Advance discover objectives |
| `void OnDungeonBossKilled(int number)` | Advance dungeon-boss objectives |
| `void ReconcileCollect(int questId, IReadOnlyList<string> owned)` | Complete collect objectives for owned keys |
| `void AdvanceObjective(int questId, int index, int amount = 1)` | Advance by index |
| `bool ConditionMet(string condition)` | Evaluate `quest:` conditions |
| `void Reset()` | Clear all quest state |
| `QuestSaveData CaptureSave()` | Capture quest state |
| `void RestoreSave(QuestSaveData)` | Replace quest state |

## LocalizationTable

| Member | Description |
|---|---|
| `string LanguageCode` | Free-form language tag |
| `string LastError` | Last load error |
| `IReadOnlyDictionary<string, string> Values` | Flattened values |
| `int Count` | Entry count |
| `bool LoadFromFile(string path)` | Load a JSON file |
| `bool LoadFromJson(string json)` | Parse nested JSON |
| `bool TryGet(string key, out string value)` | Non-throwing lookup |
| `string Get(string key, string fallback = null)` | Value, fallback or key |
| `void Set(string key, string value)` | Add or replace |
| `string Format(string template)` | Replace `{key}` markers |

## Save data

| Type | Members |
|---|---|
| `NarrativeSaveData` | `int Version`, `DialogueSaveData Dialogue`, `QuestSaveData Quests` |
| `DialogueSaveData` | `Responses`, `Flags`, `Vars`, `MetNpcs`, `Seen` |
| `QuestSaveData` | `List<QuestSaveEntry> Quests` |

All save types are `[Serializable]` and use public fields, so `JsonUtility`
works directly. See [Save and load](save-load.md).

## Component API

For `NarrativeForgeBehaviour`, `DialogueView`, `QuestLogView`,
`DialogueDatabaseAsset` and `QuestDatabaseAsset`, see
[Unity components](unity-components.md).
