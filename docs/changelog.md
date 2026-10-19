# Changelog

## 1.0.0 — 2026-10-19

- Initial public release.
- Extracted the dialogue, quest and localisation systems from an existing C++
  narrative stack into an engine-free C# core.
- Added `DialogueDatabase`, `DialogueRunner`, `QuestDatabase`, `QuestSystem`,
  `LocalizationTable` and Unity-serialisable save data.
- Added Unity integration: `NarrativeForgeBehaviour`, `DialogueView`,
  `QuestLogView`, database assets and validation inspectors.
- Added a playable Harbour Demo with three NPCs, two quests and live
  reward/flag flow.
- The demo adds the Input System UI module when the Input System package is
  active and falls back to `StandaloneInputModule`; the runtime package has no
  hard input dependency.
- Added EditMode tests for conditions, hubs, validation, quest completion,
  rewards and save/load.
- Added URP-compatible project setup and refreshed marketing captures.
