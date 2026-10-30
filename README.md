# Survive the Internet – "Photos Only" Mod

A tiny mod for **Survive the Internet** (*The Jackbox Party Pack 4*) that makes **every round use the Photo category** instead of picking a random website category.

The mod is a **single-byte change** in `SurviveTheInternet.swf`.

> **Unofficial fan project.** Not affiliated with or endorsed by Jackbox Games. You need your own legitimate copy of the game. <!-- TODO: decide whether to publish the patched .swf or only the patch instructions/script (see "Distribution note" below). -->

---

## What it does

Normally, each round of Survive the Internet takes place on a random website category (Video, Chirp, News, FriendFeed, CrowdFund, Store, Forum, CheckIn, Recommend…). Only the **final round** uses the special **Photo** category ("photosharing site", where players pick a photo and write a caption).

With this mod, **all 4 rounds** (the game's default) are Photo rounds.

| | Round 1 | Round 2 | Round 3 | Round 4 (final) |
|---|---|---|---|---|
| Original | random category | random category | random category | Photo |
| Modded | Photo | Photo | Photo | Photo |

---

## Installation

1. **Close the game.**
2. Go to the game's root folder, the one containing `SurviveTheInternet.swf`, `Localization.json` and `PauseContent.swf`.
   <!-- TODO: add the exact path on your system / how to open it (e.g. Steam → Properties → Local Files → Browse). -->
3. **Make a backup of the original file** (strongly recommended):
   rename or copy `SurviveTheInternet.swf` to `SurviveTheInternet.swf.bak`.
4. Copy the modded `SurviveTheInternet.swf` from this repository into that folder, replacing the original.
5. Start the game and play Survive the Internet as usual.

### Uninstall / restore

Delete the modded `SurviveTheInternet.swf` and restore your backup (rename `SurviveTheInternet.swf.bak` back to `SurviveTheInternet.swf`).
If anything goes wrong, or after a game update, use your game launcher's "Verify integrity of game files" to restore the original.
<!-- TODO: confirm the exact wording/steps for your platform (Steam, Epic, etc.). -->

---

## Known limitations

- The main menu, lobby and the rest of the game flow are unchanged. Only the category selection is modified.
- Rounds 1–3 now use a category that was originally only used in the final round. In testing, the game worked for at least 2 rounds. <!-- TODO: update this line after you've played a full game (all 4 rounds + end screen). -->
- A game update may overwrite the modded file.

---

## How the mod was made

This section explains the process, so you can verify or reproduce it.

### 1. Finding where the game logic lives

The game's files are Flash (`.swf`) projects plus content files (`.jet`, which are JSON):

- `Playback.swf` / `loadui.swf`: only the Jellyvision "Talkshow" engine loader and the loading bar. No game logic.
- `project.zip` contents (`data/`, `actions/`, `plugins/`): the flowcharts and graphics. Flowcharts such as `Main`, `Menu`, `Lobby`, `Intro`, `GatherResponses`, `DoVoting`… define the screen order, and they read the category from `g.gs.currentTheme.name`, a variable that is **not defined** in any of these files.
- **`SurviveTheInternet.swf`**: the main game code (ActionScript 3). It contains `GameState`, which holds `g.gs`.

### 2. Understanding the categories

Survive the Internet has no "game modes"; rounds just take place on different website **categories** (called *themes* in the code). Their definitions are in content files:

- `STITheme.jet`: the 9 regular categories (Video, Chirp, News, FriendFeed, CrowdFund, Store, Forum, CheckIn, Recommend). **No Photo.**
- `STIFinalRoundTheme.jet`: contains **only** the `Photo` category (prompt type `STIPhotoPrompt`).
- `manifest.jet`: lists the content types, including `STITheme`, `STIFinalRoundTheme` and `STIPhotoPrompt`.

### 3. Reading the round setup code

In `GameState.setupCurrentRound` (AVM2 bytecode), the theme is chosen like this:

```actionscript
// simplified
if (TuneableValues.instance.getValue("lockedTheme").val) {
    theme = ContentManager.instance.getContentByProperty("STITheme", "id", lockedTheme)[0];
} else {
    theme = ContentManager.instance.getRandomUnusedContent(
        isFinalRound ? "STIFinalRoundTheme" : "STITheme", 1, null, true, false)[0];
}
```

The game has a built-in developer setting (`lockedTheme`, default `null`), but it only searches inside `STITheme`, where Photo doesn't exist, so it can't be used directly.

### 4. The patch

Change the non-final-round branch to also use the `STIFinalRoundTheme` content type. Since that list contains only `Photo`, every round becomes a Photo round.

| | |
|---|---|
| File | `SurviveTheInternet.swf` |
| Class / method | `GameState` → `setupCurrentRound` |
| P-code location | `pushstring` at method offset `186` |
| Before | `pushstring "STITheme"` → bytes `2C 87 01` |
| After | `pushstring "STIFinalRoundTheme"` → bytes `2C 8C 01` |
| Changed byte | `0x87` → `0x8C` (string constant index 135 → 140) |
| Byte position | 32370 in the **decompressed** SWF (0-based; the file on disk is zlib-compressed, so the position there differs) |

Exactly one byte differs between the original and the modded file once both are decompressed.

### 5. Verify it yourself with JPEXS Free Flash Decompiler

1. Open the original and the modded `SurviveTheInternet.swf` in [JPEXS](https://github.com/jindrapetrik/jpexs-decompiler).
2. Go to **Scripts → `GameState` → `setupCurrentRound`**.
3. Compare the call to `getRandomUnusedContent`, or switch to the **P-code** view and look at offset 186.

**Original:**

<!-- TODO: add screenshot of the ORIGINAL file in JPEXS, e.g. docs/images/jpexs-before.png -->
![Before: original SurviveTheInternet.swf in JPEXS](docs/images/jpexs-before.png)

**Modded:**

<!-- TODO: add screenshot of the MODDED file in JPEXS, e.g. docs/images/jpexs-after.png -->
![After: modded SurviveTheInternet.swf in JPEXS](docs/images/jpexs-after.png)

---

## Doing it yourself (manual patch)

If you would rather patch your own copy instead of downloading a modified file:

1. Back up `SurviveTheInternet.swf`.
2. Open it in JPEXS and go to `GameState` → `setupCurrentRound` → P-code.
3. At offset 186, change `pushstring "STITheme"` to `pushstring "STIFinalRoundTheme"`.
4. Save the SWF and replace the file in the game folder.

<!-- TODO (optional): add a small patch script (e.g. patch.py) here if you decide to include one. -->

---

## Optional tweaks (not applied)

These are available in the game's own defaults (`Global.handleActionInit`), but are **not** part of this mod:

- `numRounds`: number of rounds (default `-1`, which means the built-in value of 4).
- `skipIntro`, `skipResponse`, `skipTwist`, `skipReveal`, `skipToEndGame`: developer skip flags.

---

## Tested with

- Game: Survive the Internet, *The Jackbox Party Pack 4*
- Build: `buildVersion 16`, build timestamp `2024-08-07 16:30:20` (from `jbg.config.jet`)
- Platform: <!-- TODO: PC / Steam / other -->
- Language: <!-- TODO: language you tested with (the localization files provided were Spanish) -->

## Distribution note

`SurviveTheInternet.swf` is copyrighted game code owned by Jackbox Games. Consider publishing only the instructions (and optionally a patch script) rather than the modified `.swf` itself.
<!-- TODO: decide, then delete or adjust this section. -->

## License

<!-- TODO: choose a license for your own text/scripts in this repo (e.g. MIT). It does not cover the game or its files. -->

## Credits

- Game and content © Jackbox Games.
- Analysis done with the help of Claude (Anthropic); verification with JPEXS Free Flash Decompiler.
