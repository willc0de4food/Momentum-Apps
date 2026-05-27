# Build & install guide (for Claude / LLM agents)

This file is intended for AI coding assistants (Claude Code, etc.) helping a
user build and install the WiFi Marauder companion `.fap` from this fork.
Humans can read it too, but the structure assumes an agent that will
execute the commands.

## What this fork's app adds

vs. the upstream
[Next-Flip/Momentum-Apps](https://github.com/Next-Flip/Momentum-Apps)
version of `wifi_marauder_companion`:

- **"Sniff + GPS" top-level menu** with sub-entries pre-wired with the
  `-g` / `--gps` flag (raw, beacon, probe, deauth, pmkid, bt, mactrack,
  packetcount). Each capture emits a DLT-192 PPI pcap with per-frame
  GPS tags.
- **Channel-locked variants** (`raw ch1`, `raw ch6`, `raw ch11`) that
  send `settings -s ChanHop disable\nchannel -s N\nsniffraw -g` to
  force-lock the radio to one 2.4 GHz channel before sniffing.
- **Channel-hop variant** (`raw hop` and the plain `raw` entry) that
  forces `settings -s ChanHop enable` first, so hopping behavior is
  deterministic regardless of prior firmware state.
- **Multi-line UART command support** — the prefix-detection logic
  (`_wifi_marauder_last_line()` in `wifi_marauder_scene_console_output.c`)
  inspects the *last* line of an embedded-`\n` command when deciding
  whether to open a pcap file and append `-serial`. This makes
  multi-step setup commands (e.g. `channel -s 1\nsniffraw -g`) work
  end-to-end with the pcap-capture plumbing.

See [ReadMe.md](ReadMe.md) for the longer human-facing version.

## ⚠️ Pairs with a specific firmware fork

These menu entries send commands using `-g` / `--gps` flags that **do not
exist in stock Marauder firmware**. The ESP32 attached to the Flipper must
be running the matching firmware fork:

➡️ **[willc0de4food/ESP32Marauder](https://github.com/willc0de4food/ESP32Marauder)**

That repo's [`CLAUDE.md`](https://github.com/willc0de4food/ESP32Marauder/blob/master/CLAUDE.md)
has the firmware build instructions. Build and flash that **first** if the
user doesn't already have it on their Dev Board.

---

# Build instructions

## Prerequisites

1. **`ufbt`** (micro Flipper Build Tool) — the standard tool for building
   Flipper apps. Install via pip:
   ```bash
   python3 -m pip install --upgrade ufbt
   ```
   On externally-managed distros (Arch/etc.) where plain `pip install`
   is blocked (PEP 668), use `pipx` — it isolates ufbt and puts it on
   PATH at `~/.local/bin/ufbt`:
   ```bash
   # Arch: pacman -S python-pipx  (Debian/Ubuntu: apt install pipx)
   pipx install ufbt
   ```
   Or a plain venv:
   ```bash
   python3 -m venv ~/.ufbt-venv
   source ~/.ufbt-venv/bin/activate
   pip install ufbt
   ```

2. **First-time `ufbt update`** — fetches the Flipper SDK matching the
   target firmware. By default it tracks the official Flipper firmware
   release channel. **Momentum firmware** (which this fork targets) is
   served from Momentum's own update index, selected with `--index-url`.
   Stock `ufbt` only accepts `dev`/`rc`/`release` for `--channel`, so
   there is **no** `momentum-release` channel — use the index URL:
   ```bash
   ufbt update --index-url=https://up.momentum-fw.dev/firmware/directory.json
   # ^ pulls Momentum's release SDK (the index's default channel).
   # For Momentum dev/rc builds, add the channel:
   # ufbt update --index-url=https://up.momentum-fw.dev/firmware/directory.json --channel=dev
   ```

3. **Sparse checkout of just this app** (optional but recommended — the
   Momentum-Apps repo contains ~150 apps, you only need this one):
   ```bash
   git clone --no-checkout https://github.com/willc0de4food/Momentum-Apps.git
   cd Momentum-Apps
   git sparse-checkout init --cone
   git sparse-checkout set wifi_marauder_companion
   git checkout dev
   ```

## Build

From inside the `wifi_marauder_companion/` directory:

```bash
cd wifi_marauder_companion
ufbt
```

Successful build prints the standard scons output ending with:
```
APPCHK    .../esp32_wifi_marauder.fap
        Target: 7, API: 87.1
```

The compiled `.fap` lands at:
```
wifi_marauder_companion/dist/esp32_wifi_marauder.fap
```
(`ufbt` also leaves a copy in `~/.ufbt/build/esp32_wifi_marauder.fap`, but
the `dist/` copy is canonical.)

## Install on the Flipper

1. Connect the Flipper via USB (or use the SD card directly).
2. Copy `dist/esp32_wifi_marauder.fap` to the Flipper SD card under
   `/ext/apps/GPIO/` (overwrite any existing
   `esp32_wifi_marauder.fap` there):
   ```bash
   # via qFlipper — drag-and-drop in the file manager

   # OR over USB CDC with mlfqp or similar:
   # cp dist/esp32_wifi_marauder.fap /run/media/$USER/Flipper/apps/GPIO/
   ```
3. On the Flipper: navigate to `Apps → GPIO → [esp32] wifi marauder`. The
   app should launch. If you see the new menu entries (`Sniff + GPS` with
   `raw ch1`, `raw hop`, etc.) you're good.

## Troubleshooting

- **`ufbt: command not found`** — pip install didn't add ufbt to PATH.
  Check `~/.local/bin/ufbt`. Add `~/.local/bin` to PATH, or call ufbt
  with the full path.
- **Build error mentioning `gui/scene_manager.h` or similar SDK
  headers** — `ufbt update` hasn't been run, or the wrong SDK was
  fetched. Run `ufbt update --index-url=https://up.momentum-fw.dev/firmware/directory.json`
  (add `--channel=dev` for a Momentum dev build) and try again.
- **Build error: `MAX_OPTIONS` exceeded** — the menu items array in
  `scenes/wifi_marauder_scene_start.c` exceeds 16 entries. Either trim
  entries or bump `MAX_OPTIONS` and `NUM_MENU_ITEMS` correspondingly
  (see `wifi_marauder_app_i.h` for the latter).
- **"Sniff + GPS" menu entries appear, but every capture is empty / no
  file in the Flipper's `apps_data/marauder/pcaps/` folder** — the
  attached ESP32 isn't running the matching firmware fork. Stock
  Marauder doesn't recognize `-g`, so the capture starts but produces
  no PPI pcap. Flash
  [willc0de4food/ESP32Marauder](https://github.com/willc0de4food/ESP32Marauder).
- **App crashes on launch** — usually a Flipper-firmware-vs-SDK API
  mismatch. Confirm the target firmware running on the Flipper matches
  the SDK you ran `ufbt update` against (e.g. Momentum release vs. dev).
  If unsure, re-run `ufbt update --index-url=https://up.momentum-fw.dev/firmware/directory.json`
  (with `--channel=` matching the firmware on the device), then rebuild.
- **Channel-locked entries (`raw ch1` etc.) still hop / channel-hop
  entries (`raw hop`) stay locked** — the firmware's persistent
  `ChanHop` setting in SPIFFS got out of sync with what the menu
  thinks it set. This SHOULDN'T happen because each entry sends an
  explicit `settings -s ChanHop disable/enable` line first, but if it
  does: launch USB-UART Bridge on the Flipper, connect to the ESP at
  115200 baud, and send `settings -s ChanHop enable` (or `disable`)
  manually to reset state.

# For Claude agents specifically

If a user asks "build the companion app for me" the canonical sequence is:

1. Verify `ufbt` is installed (`which ufbt` or `python3 -m ufbt --help`).
2. Verify `ufbt update` has been run against Momentum's index
   (`--index-url=https://up.momentum-fw.dev/firmware/directory.json`).
   If unsure which channel, ask the user (Momentum release — the index's
   default channel — for this fork).
3. `cd` into `wifi_marauder_companion/` and run `ufbt`.
4. Confirm `dist/esp32_wifi_marauder.fap` was produced.
5. Tell the user the path to the `.fap` and where to drop it on the SD
   card (`/ext/apps/GPIO/`).

If the user reports that captures aren't producing files: the most likely
cause is the ESP32 isn't running the matching firmware fork. Refer them
to the firmware fork's CLAUDE.md.

Don't write captures to or modify the Flipper SD card directly without
explicit user authorization — they may have other apps or important data
there.

Diagnostics produced by IDE linters (clang complaining about
`gui/scene_manager.h not found`, `Unknown type name 'bool'`, etc.) are
expected false positives because the linter doesn't have the Flipper SDK
include paths configured. They do NOT mean the build failed. Trust the
`ufbt` exit code and the final `APPCHK` line in its output, not the
linter.
