# GameHelper

GameHelper is a Windows x64 .NET overlay application with a plugin-based architecture. The launcher (`GameHelper.exe`) checks for updates, starts the overlay (`GameHelper.App.exe`), reads data from a running game process, renders an ImGui/ClickableTransparentOverlay UI, and loads plugins from the runtime `Plugins` directory.

This guide is written for users who want to build and run the project from Visual Studio without using command-line tools.

**Fork by Lafko**

## Download

Pre-built binaries are published on [GitHub Releases](https://github.com/MordWraith/Gamehelper/releases), not in the source tree.

| What | Link |
|------|------|
| Installer (recommended) | https://github.com/MordWraith/Gamehelper/releases/latest/download/GameHelperDownloader.exe |
| Latest release (ZIP + all files) | https://github.com/MordWraith/Gamehelper/releases/latest |
| Auto-update manifest | https://github.com/MordWraith/Gamehelper/releases/latest/download/manifest.json |

Run `GameHelperDownloader.exe` in an **empty folder** to download all files automatically.

## Required Tools

Install these items before opening the project:

- [Visual Studio](https://visualstudio.microsoft.com/downloads/) for Windows. Visual Studio Community is enough.
- Visual Studio workload: [.NET desktop development](https://learn.microsoft.com/en-us/visualstudio/install/workload-component-id-vs-community?view=visualstudio).
- [.NET 10 SDK for Windows x64](https://dotnet.microsoft.com/en-us/download/dotnet/10.0). The Desktop Runtime alone is not enough for compiling; the SDK is required.
- The project source code. You can download it as a ZIP from the repository page, use [GitHub Desktop](https://desktop.github.com/MordWraith/Gamehelper/GameHelper.csproj) (`GameHelper.App.exe`)
- Launcher / updater project: [`Launcher/Launcher.csproj`](Launcher/Launcher.csproj) (`GameHelper.exe`)
- Shared runtime setting: [`Directory.Build.props`](Directory.Build.props)
- NuGet package configuration: [`NuGet.config`](NuGet.config)

The solution platform is named `Any CPU`, but the projects set `PlatformTarget` to `x64` and the repository sets `RuntimeIdentifier` to `win-x64`.

## Open The Project

1. Start Visual Studio.
2. Choose `Open a project or solution`.
3. Select [`GameOverlay.sln`](GameOverlay.sln) from the repository root.
4. Wait for Visual Studio to finish loading the solution.
5. If Visual Studio asks whether to restore NuGet packages, allow it.

Open the solution file, not an individual `.csproj` file. Building only the main project can skip the launcher and plugin copy steps.

## Restore Packages

Visual Studio usually restores packages automatically. If it does not:

1. In `Solution Explorer`, right-click the solution `GameOverlay`.
2. Select `Restore NuGet Packages`.
3. Wait until the restore operation finishes without errors.

Keep [`NuGet.config`](NuGet.config) in the repository root. The project uses it during package restore.

## Build In Visual Studio

1. In the Visual Studio toolbar, select `Release`.
2. Keep the platform as `Any CPU`.
3. Open the top menu `Build`.
4. Select `Rebuild Solution`.
5. Wait for `Build succeeded` in the Output window.

For normal use, build `Release`. Use `Debug` only when developing or debugging the code.

### Build With PowerShell (optional)

```powershell
powershell -ExecutionPolicy Bypass -File scripts\build.ps1
```

This rebuilds the solution and copies the runnable output into `publish/`.

## Expected Build Output

After a successful `Release` build, the runnable application is created here:

```text
GameHelper\bin\Release\net10.0-windows\win-x64\
```

When using `scripts\build.ps1`, the same files are also copied to:

```text
publish\
```

That folder should contain files such as:

```text
GameHelper.exe
GameHelper.App.exe
GameHelper.App.dll
GameOffsets.dll
cimgui.dll
Plugins\
```

The `Plugins` folder should contain the built plugin folders, for example:

```text
Plugins\Atlas\
Plugins\Autopot\   (`AutoPot.dll`)
Plugins\AutoHotKeyTrigger\
Plugins\HealthBars\
Plugins\PreloadAlert\
Plugins\Radar\
Plugins\RitualHelper\
Plugins\RuneforgeHelper\
```

## Run The Program

Recommended launch method:

1. In File Explorer, open:

```text
publish\
```

or

```text
GameHelper\bin\Release\net10.0-windows\win-x64\
```

2. Double-click `GameHelper.exe`.
3. If Windows asks for administrator permission, accept it.

`GameHelper.exe` is the intended entry point: it checks for updates and starts `GameHelper.App.exe` (the overlay). Do not run `GameHelper.App.exe` directly unless you know what you are doing.

If the target game is running as administrator, start GameHelper as administrator too. The overlay application manifest requests administrator privileges.

## Solution Projects

The Visual Studio solution builds these projects:

- `GameHelper` — main overlay executable (`GameHelper.App.exe`).
- `GameOffsets` — game offset/native structure definitions.
- `Launcher` — launcher and auto-updater (`GameHelper.exe`), copied into the main output folder.
- `Downloader` — optional standalone downloader for fresh installs.
- `Atlas` — plugin.
- `AutoPot` — plugin.
- `AutoHotKeyTrigger` — plugin.
- `HealthBars` — plugin.
- `PreloadAlert` — plugin.
- `Radar` — plugin and plugin assets.
- `RitualHelper` — plugin.
- `RuneforgeHelper` — plugin.

Additional folders such as `Plugins/SamplePluginTemplate` and `Plugins/WorldDrawing` are present in the repository but are not included in the solution build by default.

## Why Rebuild The Whole Solution

A full solution build is required because several projects have Visual Studio/MSBuild copy steps:

- `Launcher` copies `GameHelper.exe` and its required DLLs into the main GameHelper output folder.
- Plugin projects copy their DLLs and assets into `GameHelper\bin\<Configuration>\net10.0-windows\win-x64\Plugins\<PluginName>\`.
- `GameHelper` copies repository documentation into the output folder when present.

If you build only `GameHelper`, the application may start without the launcher, plugin DLLs, or plugin assets.

## Runtime Configuration

Runtime settings are generated next to the executable when the program runs:

```text
configs\core_settings.json
configs\plugins.json
Plugins\<PluginName>\config\
```

These files are local runtime data and are ignored by Git.

## Troubleshooting

`The current .NET SDK does not support targeting .NET 10.0`

Install the [.NET 10 SDK for Windows x64](https://dotnet.microsoft.com/en-us/download/dotnet/10.0), update Visual Studio, restart Visual Studio, and reopen [`GameOverlay.sln`](GameOverlay.sln).

`NuGet packages did not restore`

Use `Solution Explorer > right-click GameOverlay > Restore NuGet Packages`. Also check `Tools > NuGet Package Manager > Package Manager Settings > Package Sources` and make sure `nuget.org` is enabled.

`Build succeeded, but plugins are missing`

Use `Build > Rebuild Solution`, not `Build Project`. Then check the `Plugins` folder under the `GameHelper` output directory or under `publish/`.

`Launcher says GameHelper.App.exe was not found`

You probably started the launcher from the wrong folder. Open `publish/` or `GameHelper\bin\<Configuration>\net10.0-windows\win-x64\` and run `GameHelper.exe` from there.

`Overlay does not attach to the game`

Run GameHelper with the same privilege level as the game. If the game is elevated, run GameHelper as administrator.

## Useful Official Links

- [Visual Studio downloads](https://visualstudio.microsoft.com/downloads/)
- [Install Visual Studio and choose workloads](https://learn.microsoft.com/en-us/visualstudio/install/install-visual-studio?view=visualstudio)
- [.NET desktop development workload](https://learn.microsoft.com/en-us/visualstudio/install/workload-component-id-vs-community?view=visualstudio)
- [.NET 10 SDK downloads](https://dotnet.microsoft.com/en-us/download/dotnet/10.0)
