# OLED Sleeper

A Windows desktop app (C# / .NET 8 / WPF) that protects OLED monitors from burn-in by detecting user inactivity and applying idle behaviors (blackout overlay or dimming).

## Architecture

- **Pattern:** Dependency injection, Mediator (command/handler), per-monitor state machine
- **Idle detection loop:** Runs every 200ms in `MonitorIdleDetectionService`, gathering a `SystemState` snapshot and processing each managed monitor through a 3-state machine: Active -> Counting -> Idle
- **Native interop:** All P/Invoke declarations in `Native/NativeMethods.cs`

## Key Directories

```
OLED-Sleeper/
  Features/
    MonitorIdleDetection/   # Idle detection state machine and models
    MonitorBehavior/        # Blackout/dim command handlers
    MonitorInformation/     # Monitor enumeration and info
    UserSettings/           # Settings models and persistence
  Native/                   # P/Invoke (Win32 API) declarations
  UI/
    Views/                  # WPF XAML views
    ViewModels/             # MVVM view models
  Core/                     # Mediator, DI, orchestrator
installer/                  # Inno Setup scripts (.iss)
.github/workflows/          # CI/CD
```

## Activity Detection Modes

Three independent per-monitor toggles (in `MonitorSettings`):

| Setting                  | UI Label                              | How it works                                                  |
|--------------------------|---------------------------------------|---------------------------------------------------------------|
| `IsActiveOnMousePosition`| "Mouse cursor is on this monitor"     | Cursor position within monitor bounds                         |
| `IsActiveOnActiveWindow` | "Application is focused on this monitor" | Foreground window intersects monitor                       |
| `IsActiveOnInput`        | "System has user input"               | `GetLastInputInfo` + `ES_DISPLAY_REQUIRED` execution state    |

The `IsActiveOnInput` mode also respects `ES_DISPLAY_REQUIRED` (queried via `CallNtPowerInformation`), so media players, games, video calls, etc. prevent the monitor from blanking.

## Building

```bash
# Build
dotnet build OLED-Sleeper/OLED-Sleeper.csproj

# Run
dotnet run --project OLED-Sleeper/OLED-Sleeper.csproj

# Publish + installer (requires Inno Setup)
dotnet publish OLED-Sleeper/OLED-Sleeper.csproj -c Release -r win-x64 --self-contained false -o installer/publish-x64
dotnet publish OLED-Sleeper/OLED-Sleeper.csproj -c Release -r win-x86 --self-contained false -o installer/publish-x86
iscc installer/OLED-Sleeper.iss
# Output: installer/InstallerOutput/OLED-Sleeper-<version>-Setup.exe
```

## Releasing

Push a version tag to trigger the GitHub Actions release workflow:

```bash
# Update version in installer/OLED-Sleeper.iss first (#define AppVersion)
git tag v<version>
git push origin master --tags
```

The workflow builds both architectures, compiles the Inno Setup installer, and publishes it as a GitHub Release.

## Fork Info

- **Origin:** `ShyanInc/OLED-Sleeper`
- **Upstream:** `Quorthon13/OLED-Sleeper`
