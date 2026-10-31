# Tasks

## Later (optional)

- [ ] Show a plug or bolt on an iPhone or iPad that is still plugged in but full. iOS shows the bolt then; AirBattery shows none, because the phone reports `BatteryIsCharging: false` once full. Don't fold `ExternalConnected` into `isCharging`: the "Fully Charged" alert (`BatteryAlertView.swift`) has no cooldown unless snoozed, so it would repeat every refresh until the phone is unplugged. Instead store `ExternalConnected` and `FullyCharged` as `acPowered` and `isCharged` in `IDeviceBattery.parseBattery`, show them like the Mac's own row (the `@MacInternalBattery` branches in `BatteryView.swift` and the widgets), and have the Bluetooth reader in `BLEBattery.swift` clear them when it takes over or the level drops. Discussed 31 Oct 2026.
