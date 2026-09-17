# UE5-Build-Plugin

Builds and packages an Unreal Engine plugin with `RunUAT BuildPlugin`.

The sibling of [UE5-Build-Project](https://github.com/Sector9Ltd/UE5-Build-Project), which wraps
`BuildCookRun`. They are separate actions because they are separate UAT commands with almost no
arguments in common: cook, stage, pak, maps, server and anticheat mean nothing to a plugin, and
`-StrictIncludes`, `-Dependencies` and `-NoHostPlatform` mean nothing to a cooked game.

## Usage

```yaml
- uses: Sector9Ltd/UE5-Build-Plugin@v1
  id: plugin
  with:
    RUNUAT_PATH: 'F:/Epic Games/UE_5.8/Engine/Build/BatchFiles/RunUAT.bat'
    UPLUGIN_PATH: '${{ github.workspace }}/MyPlugin/MyPlugin.uplugin'
    PACKAGE_PATH: '${{ runner.temp }}/PluginBuild'
    TARGET_PLATFORMS: 'Win64'
    ARCHIVE: 'true'

- run: echo "built ${{ steps.plugin.outputs.ARCHIVE_FILE }}"
```

Pairs with [UE5-Semantic-Versioning](https://github.com/Sector9Ltd/UE5-Semantic-Versioning) once that
action can write a `.uplugin`: tags drive `VersionName`, and `VersionName` names the zip.

## Inputs

| Input | Required | Default | Notes |
| --- | :---: | --- | --- |
| `RUNUAT_PATH` | yes | | Path to `RunUAT.bat`. |
| `UPLUGIN_PATH` | yes | | Path to the `.uplugin`. |
| `PACKAGE_PATH` | yes | | Output directory. **See the warning below.** |
| `TARGET_PLATFORMS` | no | `Win64` | Comma separated. Empty builds everything the plugin declares. |
| `HOST_PLATFORMS` | no | | Comma separated. Empty means this runner's platform. |
| `NO_HOST_PLATFORM` | no | `false` | Skip the editor build; runtime targets only. |
| `STRICT_INCLUDES` | no | `true` | See below. |
| `UNVERSIONED` | no | `false` | Package without stamping the engine version. |
| `DEPENDENCIES` | no | | Comma-separated `.uplugin` paths this plugin depends on. |
| `ENGINE_DIR` | no | | When the engine is not the one `RunUAT` belongs to. |
| `ARCHIVE` | no | `false` | Zip the result. |
| `ARCHIVE_PATH` | no | parent of `PACKAGE_PATH` | Where the zip goes. |
| `ARCHIVE_NAME` | no | `<PluginName>-<VersionName>.zip` | Read from the `.uplugin`. |

## Outputs

`VERSION_NAME`, `PACKAGE_PATH`, `ARCHIVE_FILE`.

## PACKAGE_PATH is deleted

`BuildPlugin` calls `DeleteDirectoryContents` on `-Package=` before it builds
(`BuildPluginCommand.Automation.cs`). It is silent and total. Give it a directory of its own.

UAT itself refuses two cases — an output directory inside the engine, and one containing the plugin
being built — but it will happily empty your documents folder. This action additionally refuses a
`PACKAGE_PATH` that contains `.git`, `.github` or a `.uproject`, which covers the mistakes that
actually happen.

## STRICT_INCLUDES defaults to on

UAT defaults it off; this action does not.

Without it, files compile in a unity blob, where a missing `#include` is satisfied by whichever
sibling file happens to share the blob. The file then fails to compile in a customer's editor,
whose adaptive unity excludes recently-edited files and builds them alone. CI that does not catch
this is CI that certifies a build the customer cannot make.

Set it to `false` only if your plugin does not build with it and you have accepted that.

## Exit codes

`RunUAT` reports failure only through its exit code, so this action checks `$LASTEXITCODE` and
throws. A wrapper that does not do this reports success for a plugin that never compiled.

## Notes

Arguments are passed as an array rather than assembled into a string and run through
`Invoke-Expression`. Engine and project paths routinely contain spaces — `Epic Games`,
`Unreal Projects` — and string assembly is where that breaks, usually silently.
