# DynaIDE Unity Package

Unity editor integration package that makes DynaIDE (a VS Code fork) work as an external script editor in Unity. Forked from `boxqkrtm/com.unity.ide.cursor` → `needle-mirror/com.unity.ide.visualstudio`.

## Package identity

| Field | Value |
|---|---|
| Package name | `com.sarabunari.ide.dynaide` (must stay all-lowercase — UPM rejects mixed case) |
| Display name | `DynaIDE Editor` |
| Assembly name | `Unity.DynaIDE.Editor` |
| Assembly def file | `Editor/com.sarabunari.ide.dynaide.asmdef` |

## Key files

- `Editor/VisualStudioCursorInstallation.cs` — main DynaIDE-specific class (`VisualStudioDynaIDEInstallation`). Handles discovery, process detection, and `.vscode/` file generation.
- `Editor/Discovery.cs` — calls into `VisualStudioDynaIDEInstallation` and `VisualStudioCodiumInstallation`. Don't break the Codium path.
- `Editor/ProcessRunner.cs` — reads DynaIDE workspace storage to find open workspaces.
- `Editor/VisualStudioEditor.cs` — Unity `IExternalCodeEditor` implementation; shows DynaIDE-specific GUI toggle.

## DynaIDE runtime details

DynaIDE is a VS Code fork that **cannot access the VS Code Marketplace**. It uses Open VSX.

| Concern | Value |
|---|---|
| Extension recommendation (`extensions.json`) | `muhammad-sammy.csharp` (OmniSharp fork on Open VSX) |
| Debugger type (`launch.json`) | `mono` with `address: localhost`, `port: 56000` |
| Extensions folder | `~/.dynaide/extensions` (lowercase confirmed) |
| Workspace storage — macOS | `~/Library/Application Support/DynaIDE/User/workspaceStorage` |
| Workspace storage — Linux | `~/.config/DynaIDE/User/workspaceStorage` |
| Workspace storage — Windows | `%APPDATA%/DynaIDE/User/workspaceStorage` |

## Discovery paths

**macOS** — scans `/Applications` for `DynaIDE*.app`, process names: `DynaIDE`, `DynaIDE Helper`  
**Windows** — `%LOCALAPPDATA%\Programs\dynaide\dynaide.exe` and `%ProgramFiles%\dynaide\dynaide.exe`  
**Linux** — `/usr/bin/dynaide`, `/bin/dynaide`, `/usr/local/bin/dynaide` + XDG fallback

EditorPrefs key for reuse-window toggle: `dynaide_reuse_existing_window`

## Debugger note

The `mono` attach config uses base port `56000`. Unity's actual listen port is `56000 + (PID % 1000)`. Unlike VSTU, there is no auto-attach — users attach manually via F5 in DynaIDE. If the port doesn't connect, check Unity's console for the actual port.

## GetExtensionPath / analyzers

`GetExtensionPath()` scans `~/.dynaide/extensions` for `muhammad-sammy.csharp*`. `muhammad-sammy.csharp` ships no Unity Roslyn analyzers, so `GetAnalyzers()` always returns empty. This is harmless. If analyzer support is added later, this is the method to update.
