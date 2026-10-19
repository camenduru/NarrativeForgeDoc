# Installation

## Requirements

- **Unity 6000.6 or newer.** The package is tested with Unity 6000.6.6f1.
- **uGUI.** `com.unity.ugui` is declared as a package dependency and installs
  automatically.
- **Input System is optional.** The runtime does not reference it. The sample
  selects the matching UI input module at runtime.
- **No render-pipeline dependency.** The UI and runtime work with the Built-in
  Render Pipeline, URP and HDRP.

!!! note "Unity version"
    The package manifest declares Unity 6000.6. If you need a different minimum
    version, test the package in that editor before lowering the `unity` and
    `unityRelease` fields in `package.json`.

## Install from the Unity Asset Store

1. Open **Window → Package Manager**.
2. Select **My Assets**.
3. Search for **Narrative Forge**.
4. Select the package and click **Install**.

The Asset Store version is delivered as a UPM package. Unity resolves the uGUI
dependency automatically.

## Install from a UPM tarball

Use this route for an offline build or a package you received directly.

1. Open **Window → Package Manager**.
2. Click the **+** button in the toolbar.
3. Choose **Add package from tarball…**.
4. Select `com.narrativeforge.dialogue-1.0.0.tgz`.
5. Wait for Unity to import and compile the package.

## Install as an embedded package

1. Copy the `com.narrativeforge.dialogue` folder into your project's
   `Packages/` folder.
2. Open `Packages/manifest.json` and add:

```json
"com.narrativeforge.dialogue": "file:com.narrativeforge.dialogue"
```

3. Return to Unity and let the package manager resolve the project.

An embedded package is editable, which makes it convenient while you are
integrating or extending the runtime.

## Import the sample

The **Harbour Demo** is a separate sample so it does not affect your project
unless you ask for it.

1. Open **Window → Package Manager**.
2. Select **Narrative Forge** in the package list.
3. Open the **Samples** section.
4. Select **Harbour Demo** and click **Import**.

The sample is copied into `Assets/Samples/Narrative Forge/…`. Open
`Scenes/NarrativeForgeDemo.unity` and press Play.

## Verify the install

1. Create or open a scene.
2. Select a GameObject and choose **Add Component**.
3. Search for **Narrative Forge**.

You should see the **Narrative Forge** component, plus **Dialogue View** and
**Quest Log View** under the same add-component menu.

If the components do not appear, check the Console for compile errors and
confirm that the package is listed under **Packages: In Project**.

## Updating

- **Asset Store:** update from **Package Manager → My Assets**.
- **Tarball:** remove the old package, then add the new tarball. Keep your JSON
  assets and scene references outside the package folder.
- **Embedded:** replace the package folder and let Unity recompile.

Save data is versioned. The current save version is `1`.
