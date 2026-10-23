# AirBattery (personal fork)

Fork of lihaoyun6/AirBattery. Remotes: `origin` = davisonio/AirBattery (push here, branch `main`), `upstream` = lihaoyun6, `MsieurPafi` = the best-maintained other fork, worth checking for new fixes.

- Every change from upstream is listed in `FORK.md`, with its commit and origin. Keep it current.
- Fixes only. No features, rebrands or UI redesigns unless Craig asks.
- Cherry-pick with `-x` and keep the original author. Never commit someone else's prebuilt binaries: rebuild `AirBattery/libimobiledevice/` with `scripts/build-libimobiledevice.sh` (pins and SHA-256 checks every source).
- No auto-update (Sparkle removed). Bundle ID prefix `io.davison`, team `Z67MW98588`.

Build and install:

```sh
xcodebuild -project AirBattery.xcodeproj -scheme AirBattery -configuration Release -derivedDataPath build/dd -allowProvisioningUpdates build
osascript -e 'quit app "AirBattery"'; rm -rf /Applications/AirBattery.app
ditto build/dd/Build/Products/Release/AirBattery.app /Applications/AirBattery.app && open /Applications/AirBattery.app
```

`/Applications/AirBattery.app/Contents/Resources/abt` prints what the app currently sees.
