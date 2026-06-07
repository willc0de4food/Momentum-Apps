# Fork notes (willc0de4food)

This is a personal fork of
[Next-Flip/Momentum-Apps](https://github.com/Next-Flip/Momentum-Apps) with
modifications scoped to **`wifi_marauder_companion/`** in support of an
open-source ALPR-camera research project. All other apps in this repo are
unchanged from upstream.

## What's modified

- **`wifi_marauder_companion/`** — adds a "Sniff + GPS" top-level menu with
  channel-locked / channel-hop variants (`raw`, `raw ch1`, `raw ch6`,
  `raw ch11`, `raw hop`, etc.), wires the `-g`/`--gps` flag onto every
  sniff command, and updates the prefix-extraction logic to handle
  multi-line UART command sequences.

## Where to find build / install info

All fork-specific docs live inside the `wifi_marauder_companion/` subdir:

- [`wifi_marauder_companion/ReadMe.md`](wifi_marauder_companion/ReadMe.md)
  — the full feature breakdown and human-facing changelog.
- [`wifi_marauder_companion/CLAUDE.md`](wifi_marauder_companion/CLAUDE.md)
  — step-by-step build & install instructions written for AI coding
  assistants (Claude Code, etc.). If you want your LLM session to build
  the `.fap` and tell you how to flash it, point it at that file.

## Important — requires the matching firmware fork

The new menu entries in `wifi_marauder_companion/` send commands using a
`-g`/`--gps` flag that **does not exist in stock Marauder firmware**. To use
them, the Marauder firmware on your attached ESP32 board must be from this
project's matching firmware fork:

➡️ **[willc0de4food/ESP32Marauder](https://github.com/willc0de4food/ESP32Marauder)**

That firmware fork adds DLT-192 PPI per-frame GPS tagging, a radiotap header
embedded inside the PPI capsule (per-frame RSSI + channel), and the
`-g`/`--gps` CLI flag that this companion app's menus rely on.

---

# Momentum-Apps
Bundle of external apps tweaked for [Momentum Firmware](https://github.com/Next-Flip/Momentum-Firmware).

> [!IMPORTANT]
> These apps are already included with all Momentum Firmware releases.
> This repository serves only as a way to keep them updated and maintained easier.

### Why?
Many apps included in Momentum are modified (some lots more than others). This includes:
- Removing/tweaking icons and/or their usages to **support our Asset Packs system**
- Removing duplicate keyboard implementations to **use our extended system keyboard**
- With our system keyboard also **support our CLI command `input keyboard`** to type with PC keyboard
- Tweak UART/SPI usage to **support our GPIO Pins mapping settings**
- **Moving location of save files** to a more appropriate location or changing how they are saved
- **Changing application display names** to fit our naming scheme
- **Changing how some menus work/look** or adding **new exclusive menus and features**
- **Improving or extending functionality** and better integrating with the firmware
- **Updating and fixing apps** that were abandoned by the original developers

### How?
**Apps made by our team are developed right here, the latest versions will always originate from this repository.**

**For all other apps we use git subtrees to pull updates from the creator's repository / other sources such as [@xMasterX's pack](https://github.com/xMasterX/all-the-plugins), while also keeping our own tweaks and additions.**

We didn't want to have fork repos for each single app since it would get out of hand very quick. Instead, we opted for subtrees.

Subtrees work in a very peculiar way: they pull and compare commit history from a remote repo and apply it to a subdirectory of this repo.
That's why the commit history for this repo is so huge, it contains all the commits for all the apps, plus our edits.

To make updating more manageable, we have added some scripts on top of subtrees (requires [Python](https://python.org) installed to use):
- add a new app with `.subtrees/add.py <path> <repo url> <branch> [subdir]`, this will pull the history and create `path/.gitsubtree` to remember the url, branch and subdir
- run `.subtrees/update.py <path> [path2] [pathN...]` to pull updates for some subtrees
- or run `.subtrees/update.py` with no arguments to update all subtrees

Most apps have a remote subtree URL for both the original repository, and for any forks / other sources such as [@xMasterX's pack](https://github.com/xMasterX/all-the-plugins).
This process is assisted by `.subtrees/add.py`, if the specified subtree path already exists, it will:
- remove the previous subtree with a commit
- add the subtree from the new remote
- restore the previous subtree and merge the remotes

After this, you will just need to resolve the conflicts (content differences between remotes) manually to keep the best of both, and commit.

If you're an app developer wanting to add your app, or a third party who wants to include something they find useful, you don't need to bother with this process.
We will do it for you, just add from a single remote URL, or simply make an issue requesting an app to be added!
