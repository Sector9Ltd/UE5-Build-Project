# UE5-Build-Project

Build, cook, stage, package and archive an Unreal project with RunUAT.

By [Sector 9](https://sector9.ltd). [Tool page](https://sector9.ltd/ue5-tools/build-project) | [Documentation](https://sector9.ltd/docs/ue5-tools/build-project/)

## Requirements

- A Windows runner. The step uses `shell: powershell`.
- Unreal Engine installed on that runner, because the action calls the engine's `RunUAT.bat`. In practice that means a self-hosted runner.
- Your project checked out on the runner, so `UPROJECT_PATH` points at a real `.uproject`.

Find `RunUAT.bat` under `Engine\Build\BatchFiles` in your engine install.

## Usage

`RUNUAT_PATH`, `UPROJECT_PATH` and `PLATFORM` have no default and must be set. With only those, the action cooks and stages a Development build. It does not package or archive.

```yaml
jobs:
  build:
    runs-on: [self-hosted, Windows]
    steps:
      - uses: actions/checkout@v4

      - uses: Sector9Ltd/UE5-Build-Project@0.3.1
        with:
          RUNUAT_PATH: C:\UE_5.8\Engine\Build\BatchFiles\RunUAT.bat
          UPROJECT_PATH: ${{ github.workspace }}\MyProject\MyProject.uproject
          PLATFORM: Win64
```

If RunUAT exits with a non-zero code, the step fails.

## Inputs

Inputs are set under `with:`. The action compares each switch to the text `true`, so only `true` turns it on. Values are pasted into the script between double quotes, so do not put a `"` in one.

| Name                    | Required | Default       | Description                                                                                                                                                                                                                             |
| ----------------------- | -------- | ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `RUNUAT_PATH`           | Yes      | —             | Full path to `RunUAT.bat` in your engine install.                                                                                                                                                                                       |
| `UPROJECT_PATH`         | Yes      | —             | Full path to the `.uproject` file. The action also uses its folder for the anticheat files and for `DELETE_PDB`.                                                                                                                        |
| `BUILD_CONFIG`          | Yes      | `Development` | Build configuration, such as `Development` or `Shipping`. Passed as both `-clientconfig` and `-serverconfig`.                                                                                                                           |
| `PLATFORM`              | Yes      | —             | Target platform, passed as `-platform`. With `SERVER: true` it is also the `-serverplatform`.                                                                                                                                           |
| `CLEAN`                 | No       | `false`       | `true` adds `-clean`.                                                                                                                                                                                                                   |
| `COOK`                  | No       | `true`        | `true` adds `-cook`.                                                                                                                                                                                                                    |
| `STAGE`                 | No       | `true`        | `true` adds `-stage`.                                                                                                                                                                                                                   |
| `PACKAGE`               | No       | `false`       | `true` adds `-package`.                                                                                                                                                                                                                 |
| `PAK`                   | No       | `false`       | `true` adds `-pak`.                                                                                                                                                                                                                     |
| `SERVER`                | No       | `false`       | `true` adds `-server -serverplatform=<PLATFORM> -noclient`. The `-noclient` flag means this run builds the dedicated server only, not the game client.                                                                                  |
| `ARCHIVE`               | No       | `false`       | `true` adds `-archive` with `-archivedirectory` set to `ARCHIVE_PATH`.                                                                                                                                                                  |
| `ARCHIVE_PATH`          | No       | —             | Folder to archive into. Read only when `ARCHIVE` is `true`; set it then, because the action passes it as given even when empty.                                                                                                         |
| `NULLRHI`               | No       | `false`       | `true` adds `-nullrhi`, which runs without video output, for example on a machine with no display.                                                                                                                                      |
| `EDITOR`                | No       | `true`        | Compile the editor as well. Any value other than `true` adds `-nocompileeditor`.                                                                                                                                                        |
| `ENCRYPT_INI`           | No       | `false`       | `true` adds `-encryptinifiles`.                                                                                                                                                                                                         |
| `RELEASE`               | No       | `false`       | A release version number. Any value other than `false` adds `-createreleaseversion=<value>`.                                                                                                                                            |
| `PATCH`                 | No       | `false`       | The release version to base a patch on. Any value other than `false` adds `-generatepatch -basedonreleaseversion=<value>`.                                                                                                              |
| `MAPS`                  | No       | `true`        | `true` builds all maps and passes no `-map` flag. A comma- or `+`-separated list of map names, such as `MapA,MapB`, builds only those maps; commas become `+`, the separator RunUAT reads. Passed as `-map=<list>`. See the note below. |
| `DELETE_PDB`            | No       | `false`       | `true` deletes every `.pdb` under `Saved\StagedBuilds` in the project folder after a successful build.                                                                                                                                  |
| `ANTICHEAT_ENABLED`     | No       | `false`       | `true` writes the two anticheat files before the build. See the note below.                                                                                                                                                             |
| `ANTICHEAT_PRIVATE_KEY` | No       | —             | Base64-encoded private key. Written to `Build\NoRedist\base_private.key` in the project folder when `ANTICHEAT_ENABLED` is `true`. Pass it as a repository secret.                                                                      |
| `ANTICHEAT_PUBLIC_CERT` | No       | —             | Base64-encoded public certificate. Written to `Build\NoRedist\base_public.cer` in the project folder when `ANTICHEAT_ENABLED` is `true`. Pass it as a repository secret.                                                                |

`MAPS`: the default `true` builds every map, so omit it to build all maps. A list of map names, separated by commas or `+`, builds only those maps; the action turns commas into `+`, the separator RunUAT reads.

`ANTICHEAT_ENABLED: true` writes `Build\NoRedist\base_private.key` and `base_public.cer` in the project folder from the two Base64 inputs before the build. The keys are never put on the RunUAT command line. Pass them as `${{ secrets.NAME }}`.

## Outputs

None. The result is the step's pass or fail status and the files RunUAT produces.

## Other UE5 Tools

- [UE5-Build-Plugin](https://github.com/Sector9Ltd/UE5-Build-Plugin): Build and package an Unreal plugin with RunUAT BuildPlugin.
- [UE5-Semantic-Versioning](https://github.com/Sector9Ltd/UE5-Semantic-Versioning): Work out the version and build number from Git tags and the project or plugin version.
- [UE5-EOS-Config](https://github.com/Sector9Ltd/UE5-EOS-Config): Write Epic Online Services settings into DefaultEngine.ini, with an optional dedicated-server config.

## License

See [LICENSE](LICENSE).
