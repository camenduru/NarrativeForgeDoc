# Localisation

Narrative Forge includes a small localisation table for UI strings and a
token-resolution hook for dialogue content. It does not ship a translation
service or a language-selection UI; those belong to your game.

## LocalizationTable

`LocalizationTable` loads nested JSON and flattens it into dot keys.

```json
{
  "_comment": "Comments are ignored during flattening.",
  "journal": {
    "title": "Journal",
    "empty": "No active quests."
  },
  "status": {
    "gold": "Gold",
    "xp": "XP"
  },
  "greeting": "Welcome back, {player_name}.",
  "npc": {
    "maren": "Maren, Keeper of the Tide"
  }
}
```

The table exposes:

- `journal.title`
- `journal.empty`
- `status.gold`
- `greeting`
- `npc.maren`

The `_comment` key is skipped at every level.

### Load and read

```csharp
using NarrativeForge;
using UnityEngine;

public sealed class JournalLabels : MonoBehaviour
{
    public TextAsset englishJson;
    public UnityEngine.UI.Text title;

    private readonly LocalizationTable table = new LocalizationTable();

    private void Start()
    {
        table.LanguageCode = "en";
        if (!table.LoadFromJson(englishJson.text))
        {
            Debug.LogError(table.LastError);
            return;
        }

        title.text = table.Get("journal.title", "Journal");
    }
}
```

| Member | Purpose |
|---|---|
| `LoadFromJson(string)` | Parse and flatten a JSON object |
| `LoadFromFile(string)` | Load a file from disk |
| `Get(key, fallback)` | Return a value, a fallback, or the key itself |
| `TryGet(key, out value)` | Non-throwing lookup |
| `Set(key, value)` | Add or replace a value at runtime |
| `Format(template)` | Replace `{key}` markers; unknown markers stay intact |
| `Values` | Read-only dictionary of flattened values |
| `Count` | Number of entries |
| `LanguageCode` | Free-form language tag, default `en` |
| `LastError` | Parse or file error from the last load |

### Formatting

```csharp
string text = table.Format("Welcome back, {player_name}. Gold: {status.gold}");
```

Unknown keys are left as `{key}` so missing translations are visible during
development.

## Translating dialogue

Dialogue text is plain JSON. There are two practical approaches.

### 1. One dialogue file per language

Create `dialogues_en.json`, `dialogues_de.json`, and so on. Swap the `TextAsset`
assigned to `NarrativeForgeBehaviour` and call `Load()` when the language
changes.

This is the simplest approach for hand-authored content and keeps each language
readable.

### 2. Tokens plus a table

For a key-based workflow, load dialogue text with `{key}` markers and resolve
them while the database loads:

```csharp
var table = new LocalizationTable();
table.LoadFromJson(localizationJson.text);

var database = new DialogueDatabase
{
    TokenResolver = key => table.Get(key, string.Empty)
};
database.LoadFromJson(dialogueJson.text);
```

`DialogueDatabase.TokenResolver` runs over every string in the JSON, including
NPC names, node text, choice text and locked hints. Unresolved tokens are
reported as warnings.

`NarrativeForgeBehaviour` creates its databases internally, so the token route is
best used with the core API or with a small wrapper that builds the databases
yourself.

## Localising quests

Quest titles, descriptions and objective text are also plain strings. Use one
quest file per language, or the same token approach with a custom loader.

## Language switching

A safe pattern:

1. Keep the current language code in your settings.
2. Load the matching dialogue and quest JSON.
3. Call `narrative.Load()`.
4. Rebind or refresh any views.
5. Preserve save data separately; save data stores IDs, not translated strings.

Because IDs are stable, a save created in one language can be restored in
another as long as the IDs match.

## Demo localisation

The Harbour Demo includes `loc_en.json` and uses `LocalizationTable` for its
static labels, buttons and status text. It is a compact example of flattening,
`Get` with fallbacks and `Format`-style keys.
