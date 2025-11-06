# Repository Guidelines

## Project Structure & Module Organization
The solution `shadowsocks-csharp.sln` groups the desktop client and supporting utilities. The main Windows client lives under `shadowsocks-csharp/` with clear folders for `Controller`, `Encryption`, `Model`, `Obfs`, `Util`, and WinForms `View` assets along with resources in `Resources/`. Shared third-party sources sit under `shadowsocks-csharp/3rd/`, while automated tests reside in `test/`, and packaging scripts in `packaging/`. Keep config artifacts such as `app.config` and manifests in place so the signed binaries build cleanly.

## Build, Test, and Development Commands
Restore dependencies before building: `nuget restore shadowsocks-csharp.sln` (from the repo root). Build the desktop targets with `msbuild shadowsocks-csharp.sln /p:Configuration=Release` or switch to `Debug` while iterating. To run the full MSTest suite headless, first build the tests (`msbuild test\test.csproj /p:Configuration=Release`), then execute `vstest.console.exe test\bin\Release\test.dll`. When using Visual Studio 2017+, the same commands are available through Build > Build Solution and Test Explorer.

## Coding Style & Naming Conventions
Follow the existing C# conventions: four-space indentation, braces on new lines for namespaces and types, and `PascalCase` for classes, forms, and public members. Stick to `camelCase` for locals and private fields, reserving leading underscores only for WinForms designer variables. Keep UI resources in `.resx` files and ensure any new public-facing strings pass through the localization resources in `Resources/Strings.resx`.

## Testing Guidelines
MSTest is the expected framework (`Microsoft.VisualStudio.QualityTools.UnitTestFramework`). Mirror existing naming by creating `*Test.cs` classes under `test/`, one fixture per feature. Aim to cover encryption profiles, proxy startup, and controller behaviors when modifying protocol paths; new bugs should come with regression tests. Run tests before opening a PR and confirm the Release build passes to catch configuration-specific issues.

## Commit & Pull Request Guidelines
Commits in this repo are terse and imperative (e.g., "Fix PAC updater timeout"), often grouped by component. Reference issues or PR numbers where helpful (`#123`). For pull requests, include a short summary, testing notes, and screenshots whenever UI changes touch `View/`. Keep translations and resource edits in the same commit as the feature to ease auditing.

## Security & Configuration Tips
Never commit live server credentials or PAC secrets; use sample placeholders instead. When editing configuration defaults, validate the upgrade path by launching both `ShadowsocksR-dotnet2.0.exe` and `ShadowsocksR-dotnet4.0.exe` to ensure compatibility with legacy systems.
