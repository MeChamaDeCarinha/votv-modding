# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Mods for **Voices of the Void (VotV)**, a UE4 game by MrDrNose. The goal is to **publish to Thunderstore** (https://thunderstore.io/c/voices-of-the-void/) for other players, so every mod must end up as a valid Thunderstore package and follow community conventions.

Primary reference: the community modding wiki, https://questwalker.github.io/votv-modding-wiki/ (check it before guessing at loader behavior or paths).

## Local game install

- Launched through **YeetPatch** (`~/Desktop/YeetPatch-v0.4.1`, config in `~/AppData/Local/YeetPatch/config.json`).
- Active install: `C:\Users\backe\Desktop\Games\Files\WindowsNoEditor\` (game version **0.9.0n**, read from YeetPatch config). Ignore the older `Games\Files\Voice of The Void\` copy.
- YeetPatch updates and version switches only back up `SaveGames` and `Config`. Anything installed into the game folder (UE4SS, LogicMods paks) may need reinstalling afterward. Re-check the game version before testing.
- Saves and logs: `%LOCALAPPDATA%\VotV\Saved\`.
- **Never modify or overwrite the base game files** (`VotV-WindowsNoEditor.pak`, `VotV-Win64-Shipping.exe`). Only add mod files to the mod folders below.

## Toolchain

- **Unreal Engine 4.27.2** (must be 4.27). Blank Games project. Enable "Allow ChunkID Assignments" (Editor Prefs) and "Generate Chunks" (Project Settings). Disable "Share Material Shader Code". Add `/Game` to "Additional Asset Directories to Cook". VictoryBPLibrary plugin is recommended.
- **Mod loader:** UE4SS **experimental-latest**, or Thunderstore's `unreal_shimloader` fork. Stable UE4SS v3.0.1 has a bug: it won't load blueprint mods from subfolders of LogicMods. `Mods/mods.txt` must have `BPModLoaderMod : 1`.
- **FModel**: inspect built `.pak` contents and game assets.
- **Fusion** (`NynrahGhost-Fusion`): needed only for mods that modify or remove existing game content or datatables.
- Some 0.9.0 builds crash when mods are present. Recovery: delete `dwmapi.dll` and `UE4SS.dll` from `VotV\Binaries\Win64`.

## Blueprint (.pak) mods

- Entry point is a Blueprint Actor named exactly **`ModActor`** at `Content/Mods/<ModName>/ModActor`. BPModLoaderMod spawns it. Use BeginPlay, or custom events `PreBeginPlay`/`PostBeginPlay`.
- Keep every asset under `Content/Mods/<ModName>/`. Pak contents merge with the game's filesystem, so a path that clashes with a game asset overrides or breaks it.
- Give each mod its own chunk with a **Primary Asset Label** (unique Chunk ID, "Label Assets in My Directory" on, "Apply Recursively" off).
- Package: File > Package Project > Windows (64-bit). The output is `<Output>/WindowsNoEditor/VotV/Content/Paks/pakchunk<ID>-WindowsNoEditor.pak`. **Rename it to exactly `<ModName>.pak`** (must match the `Content/Mods/<ModName>` folder) or it won't load.
- Unreferenced assets are not cooked. Verify the pak in FModel before shipping.
- Install for testing: `<game>\VotV\Content\Paks\LogicMods\<ModName>.pak`.

## Lua (UE4SS) mods

- Layout: `<ModName>/scripts/main.lua` plus `enabled.txt` (required, can be empty).
- Install for testing: `<game>\VotV\Binaries\Win64\Mods\<ModName>\`. Some wiki pages say `Win64\ue4ss\Mods`, depending on the UE4SS version, so check which folder the installed loader actually uses.

## Thunderstore packaging

Zip named `<Author>-<ModName>-<version>.zip`. These files must sit at the **zip root**, not inside a folder, and names are case-sensitive:

```
manifest.json  README.md  icon.png  CHANGELOG.md (optional)
pak/<ModName>.pak                     -> LogicMods
mod/enabled.txt, mod/scripts/main.lua -> Binaries/Win64/Mods/<ModName>
cfg/<Author>-<ModName>-cfg.json       (optional)
```

- `manifest.json` fields:
  - `name`: `[A-Za-z0-9_]` only, no spaces.
  - `version_number`: semver `X.Y.Z`. Bump it for every upload.
  - `description`: 250 characters max.
  - `website_url`: required (`""` if none).
  - `dependencies`: `"Team-Package-X.Y.Z"` strings, copied from each package page's "Dependency string". Usually `Thunderstore-unreal_shimloader-<ver>`, plus `NynrahGhost-Fusion-<ver>` if Fusion is used.
  - Validate with https://thunderstore.io/tools/manifest-v1-validator/.
- `icon.png` must be exactly **256x256**.
- README must state the target game version (currently 0.9.0n) and any dependencies.

## Community rules

- **Do not datamine the game to discover or spoil secrets.** The community bans for this.
- Don't ship the game's own assets in a package. Ship only original assets plus the mod's own logic.
- Before publishing, test with only the shimloader and this mod enabled. Then test alongside other popular mods for conflicts.
- Help: the VotV modding Discord (linked from the Thunderstore community page) and the wiki's troubleshooting section. Bug reports there expect OS, VotV version, modlist, and `UE4SS.log`.

## Repo layout (one repo, many mods)

Remote: `git@github.com:MeChamaDeCarinha/votv-modding.git`, branch `main`.

```
VotVMods/                  single UE 4.27 project shared by all blueprint mods
  Content/Mods/<ModName>/  one folder + one unique Chunk ID per mod
packages/<ModName>/        Thunderstore package sources: manifest.json, README.md,
                           CHANGELOG.md, icon.png, mod/ (Lua), cfg/
tools/                     packaging scripts
```

- Mod names must match exactly across `Content/Mods/<ModName>`, `<ModName>.pak`, `packages/<ModName>`, and the manifest `name`.
- Each mod is versioned on its own. Tag releases `<ModName>-v<X.Y.Z>` and keep the tag in sync with that mod's `manifest.json` and `CHANGELOG.md`.
- Unreal assets and source art go through **Git LFS** (see `.gitattributes`). Never commit `.pak` or `.zip` build outputs, or UE `Binaries/`, `Intermediate/`, `Saved/`, `DerivedDataCache/`.
- `website_url` in a manifest can point to `https://github.com/MeChamaDeCarinha/votv-modding/tree/main/packages/<ModName>`.
