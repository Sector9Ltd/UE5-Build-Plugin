# UE5-Build-Plugin

Build and package an Unreal plugin with RunUAT BuildPlugin.

By [Sector 9](https://sector9.ltd). [Tool page](https://sector9.ltd/ue5-tools/build-plugin) | [Documentation](https://sector9.ltd/docs/ue5-tools/build-plugin/)

## Requirements

- A Windows runner. The step uses `shell: powershell`.
- Unreal Engine installed on that runner, because the action calls the engine's `RunUAT.bat`. In practice that means a self-hosted runner.
- Your plugin checked out on the runner, so `UPLUGIN_PATH` points at a real `.uplugin` file.
- A directory for the packaged output that is yours alone. RunUAT deletes the contents of `PACKAGE_PATH` before it builds.

Find `RunUAT.bat` under `Engine\Build\BatchFiles` in your engine install.

## Usage

`RUNUAT_PATH`, `UPLUGIN_PATH` and `PACKAGE_PATH` have no default and must be set. With only those, the action builds the plugin for Win64 with strict includes on. It does not zip anything.

```yaml
jobs:
  build:
    runs-on: [self-hosted, Windows]
    steps:
      - uses: actions/checkout@v4

      - uses: Sector9Ltd/UE5-Build-Plugin@1.1.0
        with:
          RUNUAT_PATH: C:\UE_5.8\Engine\Build\BatchFiles\RunUAT.bat
          UPLUGIN_PATH: ${{ github.workspace }}\MyPlugin\MyPlugin.uplugin
          PACKAGE_PATH: ${{ runner.temp }}\MyPlugin-package
```

If RunUAT exits with a non-zero code, the step fails.

## Inputs

Inputs are set under `with:`. The action compares each switch to the text `true`, so only `true` turns it on. Values are pasted into a PowerShell script between double quotes, so do not put a `"` in one.

| Name               | Required | Default | Description                                                                                                                                                                                                                |
| ------------------ | -------- | ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `RUNUAT_PATH`      | Yes      | —       | Full path to `RunUAT.bat` in your engine install. The step fails if it does not exist.                                                                                                                                     |
| `UPLUGIN_PATH`     | Yes      | —       | Full path to the `.uplugin` file. The action reads `VersionName` from it and takes the plugin name from its filename.                                                                                                      |
| `PACKAGE_PATH`     | Yes      | —       | Directory the packaged plugin is written to. See the note below.                                                                                                                                                           |
| `TARGET_PLATFORMS` | No       | `Win64` | Comma- or `+`-separated platforms to build for, such as `Win64,Linux` or `Win64+Linux`. The action turns commas into `+`, the separator RunUAT reads. The action adds `-TargetPlatforms` only when the value is non-empty. |
| `HOST_PLATFORMS`   | No       | `""`    | Comma- or `+`-separated host platforms for the editor build. Commas become `+`, as for `TARGET_PLATFORMS`. Passed as `-HostPlatforms` when set. Empty passes no flag, which means the platform the runner is.              |
| `NO_HOST_PLATFORM` | No       | `false` | `true` adds `-NoHostPlatform`, which skips the editor build and builds runtime targets only.                                                                                                                               |
| `STRICT_INCLUDES`  | No       | `true`  | `true` adds `-StrictIncludes`, which compiles each file without the unity blob so a missing `#include` is an error. On by default; set `false` to turn it off.                                                             |
| `UNVERSIONED`      | No       | `false` | `true` adds `-unversioned`, which packages without stamping the engine version, for a plugin meant to load in more than one.                                                                                               |
| `DEPENDENCIES`     | No       | `""`    | Comma-separated paths to `.uplugin` files this plugin depends on. Each path, trimmed, becomes its own `-Dependencies=<path>` flag. Empty entries are skipped.                                                              |
| `ENGINE_DIR`       | No       | `""`    | Engine directory, for when it is not the one `RunUAT.bat` belongs to. Passed as `-EngineDir` when set.                                                                                                                     |
| `ARCHIVE`          | No       | `false` | `true` zips the packaged plugin after a successful build and sets the `ARCHIVE_FILE` output.                                                                                                                               |
| `ARCHIVE_PATH`     | No       | `""`    | Directory to write the zip into, created if it is missing. Empty means the parent of `PACKAGE_PATH`. Read only when `ARCHIVE` is `true`.                                                                                   |
| `ARCHIVE_NAME`     | No       | `""`    | Zip filename. Empty means `<PluginName>-<VersionName>.zip`, where the plugin name is the `.uplugin` filename without its extension. Read only when `ARCHIVE` is `true`.                                                    |

`PACKAGE_PATH`: RunUAT deletes the contents of this directory before it builds, so give it a directory of its own. It must be outside the engine directory and must not contain the plugin being built. The action also refuses a directory that holds a `.git` or `.github` folder or any `.uproject` file.

## Outputs

| Name           | Description                                                                     |
| -------------- | ------------------------------------------------------------------------------- |
| `VERSION_NAME` | `VersionName` from the `.uplugin`, or `0.0` when it has none.                   |
| `PACKAGE_PATH` | The packaged plugin directory, as given in the `PACKAGE_PATH` input.            |
| `ARCHIVE_FILE` | Full path of the zip. Set only when `ARCHIVE` is `true`; otherwise it is empty. |

Outputs are set only after RunUAT succeeds. Read them from a later step with the step's `id`, for example `${{ steps.plugin.outputs.ARCHIVE_FILE }}`.

## Other UE5 Tools

- [UE5-Build-Project](https://github.com/Sector9Ltd/UE5-Build-Project): Build, cook, stage, package and archive an Unreal project with RunUAT.
- [UE5-Semantic-Versioning](https://github.com/Sector9Ltd/UE5-Semantic-Versioning): Work out the version and build number from Git tags and the project or plugin version.
- [UE5-EOS-Config](https://github.com/Sector9Ltd/UE5-EOS-Config): Write Epic Online Services settings into DefaultEngine.ini, with an optional dedicated-server config.

## License

See [LICENSE](LICENSE).
