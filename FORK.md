# Fork notes

Personal fork of [lihaoyun6/AirBattery](https://github.com/lihaoyun6/AirBattery). This file lists every change from upstream and where it came from.

**Base:** upstream `main` at `134e02f` (2 Nov 2025). That is release 1.6.3 plus PRs #161, #162, #163 and #177, which were merged upstream but never released.

**Rules:** bug fixes only, no new features or UI redesigns, and no prebuilt binaries from anyone else. Cherry-picks keep the original author and carry a `(cherry picked from commit …)` line.

## What changed

### Build, identity and updates

- Bundle IDs `io.davison.AirBattery`, `io.davison.AirBattery.widget` and `io.davison.AirBatteryHelper`, signed by team `Z67MW98588`. The prefix is set in one place: the `APP_ID_PREFIX` build setting, mirrored in `Shared/AppIdentifiers.swift`. Source: `af2deb6`, plus `838d37d`.
- **Sparkle removed** (package, update views, `appcast.xml`). Upstream checks `raw.githubusercontent.com/lihaoyun6/AirBattery/main/appcast.xml` and installs whatever that feed points to. This fork never checks for updates. Source: `416206f`, then `838d37d`.
- Deployment target raised to macOS 12.0, which the PR #216 fix needs (`ab9f17a`).
- Missing `import ServiceManagement` added, so the project builds (`c5ff81b`).
- **libimobiledevice rebuilt from source.** Upstream ships 5 prebuilt executables and 6 dylibs in `AirBattery/libimobiledevice/`, with no build script and no source for `comptest`. `scripts/build-libimobiledevice.sh` downloads pinned release tarballs, checks their SHA-256 against the digests GitHub publishes for those releases, and builds universal binaries locally. The pinned versions are OpenSSL 3.5.9 LTS, libplist 2.8.0, libimobiledevice-glue 1.3.3, libusbmuxd 2.1.1, libtatsu 1.0.5 and libimobiledevice 1.4.0. `comptest.c` comes from libimobiledevice maintainer nikias's gist, with fixes. Every binary in this repo was produced by that script on this Mac (`c5fb43f`); `AirBattery/libimobiledevice/VERSIONS.txt` records the inputs. Script and `comptest.c` by Côme Bouquet (`623248c`).

### Crash and stability fixes

| Commit | Fix | Upstream ref | Origin |
| --- | --- | --- | --- |
| `7c1dfe7` | Over-release of `IOBluetoothDevice` in the connect handler | #215, likely #206 | PR #216 (R0cketBean) |
| `6cf1e37` | Data race when many Bluetooth devices reconnect after wake (SIGABRT) | #215 | PR #216 |
| `408d895` | `IOServiceClose()` on a never-opened service: two Mach port `EXC_GUARD` violations every second | #207 | MsieurPafi |
| `9dc5016` | iPhone/iPad/Watch battery output parsed without force-unwraps; `comptest` times out after 15 s | #198 | MsieurPafi, from callieskye |
| `623248c` | `comptest` used an uninitialised device pointer when the phone was unreachable (the real cause of #198) | #198 | MsieurPafi |
| `1c53735` | Forced casts removed from Bluetooth log and IOKit parsing | #106 | MsieurPafi, from callieskye |

### Bluetooth behaviour

| Commit | Fix | Upstream ref | Origin |
| --- | --- | --- | --- |
| `f5036ae` `2de7560` `e98e4dd` `44addb3` | Connect over BLE only to iOS devices the user has authorised. No more pairing pop-ups from other people's phones at work or in cafés | #89, #158, #169 | PR #214 (Moyuin-aka) |
| `49452e6` | Generic BLE devices are disconnected after reading, and battery-less ones are not retried. These connections kept the Mac awake and made audio stutter | #132, #178 | MsieurPafi, from callieskye |
| `33b1098` `1819d48` | BLE scan skipped when no setting needs it, and run once per refresh interval instead of constantly | #178 | MsieurPafi |
| `613c835` | Read BLE mice and keyboards that are already connected to the Mac (they stop advertising, so the scan never saw them; tested on an MX Anywhere 3) | related: #209 | MsieurPafi |
| `3673b94` | An iPhone that dropped off Wi-Fi kept its last Wi-Fi level and charging state for up to 20 minutes; Bluetooth now takes over once the Wi-Fi reading is 3 refresh intervals old. Over Bluetooth the charging state is guessed from the level (an iPhone offers no charging signal there: its Battery Service has only the level, and Continuity adverts carry none), and it reset to "not charging" on every read without a rise, so a charging iPhone flickered. A steady level now keeps the last state | | this fork |
| `f21761c` | iPhone read over Bluetooth (the Mac keeps it connected for Continuity) showed the generic Bluetooth icon: its model number arrived before its battery level and was dropped, and later refreshes only re-read the level | | this fork |
| `94caf9d` | Read macOS's accessory power sources (what `pmset -g accps` shows), so BLE keyboards and mice with only a standard GATT battery appear | | DecKirin/GlassyBattery (Xiding Luo) |
| `c7fa9a3` | AirPods named after their model when macOS 26 reports them as "iPhone" | #172 | MsieurPafi, from callieskye |
| `ffb513c` | AirPods levels clamped to 100% | #76 | MsieurPafi, from callieskye |

### iPhone, iPad and Watch

| Commit | Fix | Upstream ref | Origin |
| --- | --- | --- | --- |
| `2c8945e` | With iOS "Private Wi-Fi Address" on (the default since iOS 18), the MAC the phone announces on the network doesn't match the one in the Mac's pairing record, so the phone never shows up over Wi-Fi after the cable is pulled. `wificonnection syncmac` updates `WiFiMACAddress` in the pairing record through usbmuxd while the phone is on USB. This is the only change that writes outside the app: one field of the system pairing record for that phone | #79, #115, #192 | PR #210 (JingYao152) |
| `6548e5d` | iOS devices scanned on their own 15 s timer, not a 5 s timer tied to a SwiftUI view | #91 | MsieurPafi |
| `fa1f84f` | `abt` command-line tool reads data from the correct container | | MsieurPafi |

### Mac battery, alerts and UI

| Commit | Fix | Upstream ref | Origin |
| --- | --- | --- | --- |
| `cac4828` | Mac battery % read from `IOPowerSources`, so it matches the menu bar | #152 | MsieurPafi, from callieskye |
| `a4b33fc` | One device in alert cooldown no longer silences alerts for every other device | #78 | MsieurPafi, from callieskye |
| `d904192` | Popover attached to the menu bar on macOS 26+ | | MsieurPafi |
| `28db6fb` | Icons for AirPods 4 ANC, AirPods Max 2 and MacBook Neo | #166, #195, #204, #205 | MsieurPafi, from callieskye |
| `7cb052b` | Nearcast row highlight cleared when the cursor leaves | #159 | MsieurPafi, from callieskye |
| `8990d10` | Pinned menu bar icons stay current and keep their position across launches | #182 | MsieurPafi, from callieskye |
| `8e02c56` | Settings window comes to the front when you click the gear (it opened behind other apps on macOS 14+); grammar of the Bluetooth tip | #175 | this fork |

### Changes Craig asked for

| Commit | Change |
| --- | --- |
| `e132760` | Last-update time on hover reads "now", "1 min ago", "2 hr ago" instead of "0 mins ago", and is no longer cut off by the hover buttons |
| `e132760` | Widget settings: "Show This Mac" toggle to leave the Mac's own battery off widgets (upstream had this commented out) |
| `aad0135` | English fixes in settings, widget picker and alerts ("More ways to displays", "will be showed", "VisionPro" and others) |
| `f8e1b6b` | AirPods show while in their case, from the levels macOS shares from the iPhone (the system Batteries widget's source); before, they only appeared with the lid open or in your ears |
| `f8e1b6b` | Earbud Merging Threshold can be set to 0 to always show left and right separately |
| `dd1a148` | Left AirPod always listed before the right (it followed whichever reported first) |
| `dd1a148` | Menu Bar settings: "Show This Mac in List" toggle to leave the Mac's own battery out of the popover (upstream always shows it; the Dock's "Built-in Battery Style" didn't apply there) |

### Security fixes (from this fork's audit, `430d19e`)

- Nearcast: the unencrypted `sender` field of an incoming message was used as a file name, so anyone on the same network could write a `.json` file to any path the user can write to. The sender must now be a plain ID (`[A-Za-z0-9-]`, at most 64 characters).
- Command-line tool install: the app path went into an `administrator privileges` shell command unescaped. It is now shell-quoted and escaped for AppleScript.

## Security review (23 Oct 2026)

Upstream `134e02f` was read in full, and the bundled binaries were inspected statically. Nothing malicious turned up: the app contacts no hosts besides the Sparkle feed (now removed), has no obfuscation, no runtime code loading, and reads no contacts, keychain or location data. It touches the clipboard only when you click a button that uses it. Things to know:

- **Nearcast** (off by default) uses MultipeerKit with no transport encryption and accepts every peer on the LAN. Device lists are encrypted with a key derived from the Group ID, but only 8 characters of it are secret. Leave it off unless you need to sync between your own Macs.
- **iPhone trust** gives any app you run full lockdown access to the phone; that comes from macOS, not this app. This app reads the main lockdown domain (it uses only the name and model; the rest, including serial and phone number, is neither stored nor sent), the battery domain, and the Watch list through `companion_proxy`. It **sets `EnableWifiConnections` on the phone** each time it is on USB, which is what lets it read the phone over Wi-Fi afterwards. PR #210 also updates `WiFiMACAddress` in the Mac's pairing record (see above).
- The upstream binaries were Homebrew OpenSSL 3.2 (end of life) and a libimobiledevice development snapshot. They looked unmodified, but they are replaced here anyway. The rebuilt OpenSSL looks for its config under `/usr/local/airbattery-openssl`, a path that doesn't exist and needs root to create.
- "Discover more BLE devices" (off by default) connects to any nearby named non-Apple BLE device to read its battery.

## Considered and left out

| Source | Why not |
| --- | --- |
| [MsieurPafi/AirBattery](https://github.com/MsieurPafi/AirBattery) docs, "AirBattery Plus" rename, list sorting, Liquid Glass layout pass | Docs (in French) and features, not fixes. That fork's `docs/UPSTREAM.md` and `docs/DECISIONS.md` explain the reasoning behind each fix above |
| Their prebuilt binaries, and the binary in PR #210 | Rebuilt from source here instead |
| PR #191 (Roccat mouse via `logReader.sh`) | More fragile log parsing. `94caf9d` reads the same devices from the system power-source API |
| PR #202 (MacBook Neo icon) | Covered by `28db6fb` |
| PR #200, #211, #164, #170 | UI redesign, energy-mode switcher, CLI command, French translation: features |
| andrewcincotta "fix bluetooth mode in clustered areas" | Superseded by PR #214 and `49452e6` |
| Cheerpipe thread-safety/crash commits | Superseded by PR #216. The rest of that fork is a UI overhaul |
| DecKirin battery-health fix | Upstream never displays battery health |
| jenkinpan Watch flicker fix | Fixes a regression in that fork's own refactor |
| callieskye "1.6.4/1.6.5 Community Patch" (#208) | Only prebuilt downloads plus a source import with no shared history. MsieurPafi ported the parts worth having, listed above |
| josephcourtney (759 commits) | UI-test harness experiments |
| GlassyBattery, BatteryFlow, jenkinpan v3 | Rebrands and redesigns |

## Building

```sh
scripts/build-libimobiledevice.sh   # only when changing the libimobiledevice stack
xcodebuild -project AirBattery.xcodeproj -scheme AirBattery -configuration Release build
```
