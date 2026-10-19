# Troubleshooting

## The Narrative Forge components are missing

- Check the Console for compile errors.
- Confirm **Narrative Forge** appears under **Package Manager → In Project**.
- Confirm the project uses Unity 6000.6 or newer.
- If you installed a tarball, remove the old package before adding a new one.
- If you copied an embedded package, make sure `Packages/manifest.json` references
  the folder correctly.

## “assign both dialogueJson and questJson”

`NarrativeForgeBehaviour.Load()` needs both `TextAsset` fields. Assign both JSON
files in the Inspector, or call `Load()` after assigning them from code.

## “no 'npcs' object found” or “no 'quests' array found”

The JSON shape is wrong. Dialogue needs a top-level `npcs` object; quests need a
top-level `quests` array. Check for a missing brace, a trailing comma or a file
that contains the wrong asset.

## Dialogue does not start

`Dialogue.Start(npcId)` returns `false` when:

- the NPC ID does not exist;
- the NPC has no nodes;
- the entry node cannot be resolved.

Check `narrative.Dialogue.Start(1)` in a debug log and verify the NPC key and
`default_start` in the JSON. Validation reports missing entry nodes.

## The dialogue panel stays hidden

- Assign the **Panel** field on `DialogueView`.
- Confirm the view's **Narrative** field points at the loaded behaviour.
- Check that `Start()` succeeded.
- If you created the view after load, call `Bind()` or subscribe to `Loaded`.

## No choices appear

- The node may have `"end": true`.
- The node may have an empty or missing `choices` array.
- A choice with a hard `!flag` gate can be hidden.
- A choice without `locked_hint` is hidden when its conditions fail.
- The **Choice Button Template** may be unassigned or inactive in a way that
  prevents cloning.

## A locked choice is not visible

A failed normal condition is only visible when the choice has a `locked_hint`.
Add a hint if you want the player to see the requirement. A failed `!flag` gate
is always hidden by design.

## A quest does not start

`StartQuest` returns `false` when:

- the quest ID is unknown;
- the quest already has state (including completed or failed);
- a prerequisite is not completed.

Check `narrative.Quests.Status(questId)` and the `prereqs` array.

## An objective does not advance

- Confirm the quest is active: `narrative.Quests.IsActive(questId)`.
- Confirm the event type matches the objective type.
- Confirm the target matches exactly:
  - `talk` uses the NPC ID;
  - `kill` uses your enemy type integer;
  - `collect`, `reach` and `discover` use the exact string key;
  - `dungeon_boss` uses 1–6.
- Remember that string comparisons are case-sensitive.
- Check that you are calling `OnCollect` with the amount you expect.

## Rewards are not applied

`NarrativeForgeBehaviour` installs a default reward sink only when **Apply
Rewards To Dialogue State** is enabled. Assigning your own `RewardSink` replaces
it. If you need both, implement the default flag/gold/XP/reputation behavior in
your sink.

Reward `items` are never applied automatically; they are data for your inventory
code.

## Gold or XP does not change

The default sink adds rewards to the dialogue variables `gold`, `xp` and
`reputation`. Read them with:

```csharp
int gold = narrative.Dialogue.GetVar("gold");
```

If you use a custom sink, update your own currency values instead.

## The journal is stale after restoring a save

`RestoreSave` does not fire `Loaded`. Call `Refresh()` or `Bind()` on the views
after restoring:

```csharp
narrative.RestoreSave(save);
questLog.Refresh();
dialogueView.Refresh();
```

## UI clicks do nothing

- Ensure the scene has an `EventSystem`.
- Add `InputSystemUIInputModule` when the project uses the Input System.
- Add `StandaloneInputModule` when the project uses the legacy Input Manager.
- Do not add both at the same time.
- Check that a full-screen UI element is not blocking the buttons.

The sample's `DemoInputModule` is a reference for selecting the module at
runtime.

## The sample works but my scene does not

Compare against the sample hierarchy and the fields listed in
[Scene setup](getting-started/scene-setup.md). The most common differences are a
missing `EventSystem`, an unassigned panel, an unassigned choice template or a
`NarrativeForgeBehaviour` that was never loaded.

## Localisation keys show as `{key}`

The key is missing from the table. `Get` returns the fallback or the key, and
`Format` leaves unknown markers intact. Check the flattened key path and the
`_comment` exclusions.

## I changed an ID and old saves broke

Save data stores NPC, node, choice and quest IDs. Keep IDs stable or write a
migration that maps old IDs to new ones. Increasing `NarrativeSaveData.Version`
is the intended place to record a migration.

## Still stuck?

See [Support](support.md) and include the Unity version, package version, the
relevant JSON, the Console error and the steps that reproduce the issue.
