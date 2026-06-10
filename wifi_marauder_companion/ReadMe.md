# Fork notes (willc0de4food)

This fork adds GPS-aware menu entries to support
[willc0de4food/ESP32Marauder](https://github.com/willc0de4food/ESP32Marauder),
a personal fork of the firmware that emits DLT-192 PPI pcaps with per-frame
GPS tags and per-frame radiotap RSSI/channel. **These menu entries will not
work on stock Marauder firmware** — the `-g` / `--gps` flag they pass on every
sniff command is only recognized by that firmware fork. Use the matching
firmware or these entries will fail / fall back to non-GPS captures.

## What's different from upstream

- **New "Sniff + GPS" top-level menu** with sub-entries pre-wired with `-g`:
  - `raw` / `raw hop` — start a raw 802.11 sniff and ensure channel-hop is
    enabled before sniffing (via `settings -s ChanHop enable`)
  - `raw ch1` / `raw ch6` / `raw ch11` — disable channel-hop and lock the
    radio to one 2.4 GHz channel before sniffing (`settings -s ChanHop disable`
    + `channel -s N`). Useful for catching duty-cycled devices that beacon
    or probe on a known channel (e.g. Flock ALPR cameras).
  - `flock ch1` / `flock hop` — the same lock / channel-hop choice applied to
    the Flock sniff (`sniffbt -t flock -g`). Plain `flock` inherits whatever
    `ChanHop` state was last set, so these make it explicit: `ch1` to dwell on
    a known camera, `hop` to also catch cameras on ch6 / ch11.
  - `probe`, `beacon`, `deauth`, `pmkid`, `bt`, `flock`, `airtag`, `flipper`,
    `mactrack`, `packetcount` — standard variants of those sniffs, with `-g`
    applied so each pcap carries per-frame GPS coordinates.
- **Multi-line UART commands** for the channel-locked / channel-hop entries
  use `\n`-separated sequences (e.g. `settings -s ChanHop disable\nchannel -s 1\nsniffraw -g`).
  The companion's prefix-extraction logic (`_wifi_marauder_last_line()`) was
  updated to inspect the *last* line of the command when deciding whether
  to open a pcap file and append `-serial` for streaming, so multi-step
  setup commands don't break the capture-file plumbing.
- **GPS no-fix alert.** During a "Sniff + GPS" capture the firmware emits
  `[GPS] FIX/NOFIX sats=N` lines over UART; the companion parses them and fires
  a Flipper notification (buzzer + vibrate + blinking red LED) whenever the
  capture has no GPS fix, with the satellite count shown in the console. Without
  it, a fixless capture silently records the firmware's `-180` no-fix sentinel
  for every frame and you only discover the positionless pcap back home.

## Why

This is the Flipper-side half of an open-source research project tracking
Flock Safety ALPR cameras and similar public-safety surveillance infrastructure
in the user's neighborhood — same category of work as Wireshark, Kismet,
hcxdumptool, and [deflock.me](https://deflock.me). All captures are of public
802.11 broadcasts (beacons, probe-requests, deauths); no decryption, no
client targeting.

The matching firmware lives at
[willc0de4food/ESP32Marauder](https://github.com/willc0de4food/ESP32Marauder).
See that repo's README for what was added on the firmware side
(PPI per-frame GPS tagging, embedded radiotap header for per-frame RSSI +
channel, `-g`/`--gps` CLI flag).

---

[![FAP Build](https://github.com/0xchocolate/flipperzero-wifi-marauder/actions/workflows/build.yml/badge.svg)](https://github.com/0xchocolate/flipperzero-wifi-marauder/actions/workflows/build.yml)

# WiFi Marauder companion app for Flipper Zero

Requires a connected dev board running Marauder FW. [See install instructions from UberGuidoZ here.](https://github.com/UberGuidoZ/Flipper/tree/main/Wifi_DevBoard#marauder-install-information)

<img src="https://github.com/0xchocolate/flipperzero-wifi-marauder/blob/feature_wifi_marauder_app/screenshots/marauder-topmenu.png?raw=true" width=20% height=20% /> <img src="https://github.com/0xchocolate/flipperzero-wifi-marauder/blob/feature_wifi_marauder_app/screenshots/marauder-script-demo.png?raw=true" width=20% height=20% /> <img src="https://github.com/0xchocolate/flipperzero-wifi-marauder/blob/feature_wifi_marauder_app/screenshots/marauder-save-pcaps.png?raw=true" width=20% height=20% />

## Get the app
1. Make sure you're logged in with a github account (otherwise the downloads in step 2 won't work)
2. Navigate to the [FAP Build](https://github.com/0xchocolate/flipperzero-wifi-marauder/actions/workflows/build.yml)
   GitHub action workflow, and select the most recent run, scroll down to artifacts.
3. The FAP is built for the `dev` and `release` channels of both official and unleashed
   firmware. Download the artifact corresponding to your firmware version.
4. (Optional step to avoid confusion) Go to "Apps/GPIO" on the Flipper SD Card, delete any existing Marauder app, on some firmwares there will be a `ESP32CAM_Marauder.fap` or similar.
5. Extract `esp32_wifi_marauder.fap` from the ZIP file downloaded in step 3 to your Flipper Zero SD card, preferably under Apps/GPIO along with the rest of the GPIO apps. (If you're using qFlipper to transfer files you need to extract the content of the ZIP file to your computer before you drag it to qFlipper, as qFlipper does not support direct dragging from a ZIP file (at least on Windows)).

From a local clone of this repo, you can also build the app yourself using ufbt.

### FYI - the ESP flasher is now its own app: https://github.com/0xchocolate/flipperzero-esp-flasher


## Support

For app feedback, bugs, and feature requests, please [create an issue here](https://github.com/0xchocolate/flipperzero-firmware-with-wifi-marauder-companion/issues).

You can find me (0xchocolate) on discord as @cococode.

If you'd like to donate to the app development effort:

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/O4O1R7X6K)

**ETH**: `0xf32A1F0CD6122C97d8953183E53cB889cc087C9b`  
**BTC**: `bc1qtw7s25cwdkuaups22yna8sttfxn0usm2f35wc3`

Find more info about Marauder and support its developer (justcallmekoko aka WillStunForFood) here: https://github.com/justcallmekoko/ESP32Marauder

If you found the app preinstalled in a firmware release, consider supporting the maintainers!
