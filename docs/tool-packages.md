# Tool packages on GitHub Releases

DevKit3 reads packages from the public repository `minhnguyenerp/devkit3`.
Publish **one release per tool**, with the exact **tag** below. The release title is only a display label.
Add a new asset to the same release for each version; no DevKit3 rebuild is needed.

All 69 DevKit2 application IDs are registered; see the [full tool catalog](tool-catalog.md) for every release tag, executable layout, SDK dependency and service setting. Cmd and VisualStudio are system tools and do not need releases. Git is also a system tool on Linux and needs no Linux release asset.

| Release tag | Example asset | ZIP contents, relative to archive root |
| --- | --- | --- |
| `Node` | `Node-win-x64-24.16.0.zip` | `node.exe`, `npm.cmd`, `npx.cmd`, `node_modules/`, etc. |
| `Php` | `Php-win-x64-8.5.6.zip` | `php.exe`, `php.ini`, `ext/`, dependency DLLs, Apache module if needed |
| `Git` | `Git-win-x64-2.55.0.3.zip` | Extracted PortableGit: `bin/git.exe` (or `git.exe` at the root), `mingw64/`, `usr/`, etc. |
| `Mariadb` | `Mariadb-win-x64-12.3.2.zip` | `bin/mariadbd.exe`, `bin/mariadb-install-db.exe`, `share/`, etc. |
| `Apache` | `Apache-win-x64-2.4.69-261002.zip` | `Apache24/bin/httpd.exe`, `Apache24/conf/mime.types`, `Apache24/modules/`, etc. |
| `DotnetSDK` | `DotnetSDK-win-x64-10.0.302.zip` | `dotnet.exe`, `sdk/`, `shared/`, `host/`, etc. |
| `VSCode` | `VSCode-win-x64-1.130.0.zip` | `Code.exe`, `bin/`, `resources/`, etc. |
| `Arduino` | `Arduino-win-x64-2.3.10.zip` | `Arduino IDE.exe` and the remaining IDE distribution files |

Use the exact casing shown. New filenames follow `<ToolId>-<win|linux|mac>-<architecture>-<version>.zip` or the same pattern ending in `.tar.gz`.
For example, `Arduino-linux-x64-2.3.10.tar.gz` and `Arduino-mac-arm64-2.3.10.zip` belong to the same `Arduino` release.
The catalog selects the native OS architecture (`x64`, `arm64`, etc.) and exposes only the version in the editor.
An asset for another OS or architecture is never selected as a fallback.
Both ZIP and TAR.GZ packages are recognized and extracted automatically. When both formats exist for the same platform and version, Windows prefers ZIP; Linux and macOS prefer TAR.GZ. Only one version entry is shown, regardless of asset order.
Original `<ToolId>-<version>.zip` assets remain supported on Windows x64 only. A matching new-format asset takes precedence over a legacy asset for the same version.
Versions support integer releases (NetBeans `29`), years (MSVC `2026`), dotted numbers (including four-part Git versions), and suffixes such as Apache's build number.
Other files such as checksum attachments and source archives are ignored.

Platform-specific discovery does not make every existing tool integration cross-platform. Other tools still need their Windows-specific launch, environment, or configuration handling adapted before use on Linux/macOS.

Put the files directly at the archive root as shown. Do not add a `Node-24.16.0/` or `Php-8.5.6/` wrapper directory. Extraction preserves the archive's directory structure; an extra wrapper directory is not removed automatically.
Node's original `node-v<version>-win-x64/` directory and MariaDB's original `mariadb-<version>-winx64/` directory are also supported, including existing installations.

Include all extensions, modules, configuration and dependencies in the package. DevKit3 downloads one archive and extracts it. TAR.GZ extraction preserves Unix modes and links using the .NET TAR extractor; there is no external `tar` dependency. Keep the real file format when naming assets: renaming a TAR.GZ file to ZIP does not convert it.
PHP uses a thread-safe build for Apache integration; DevKit3 adjusts `extension_dir` in the packaged `php.ini` to the final installation directory.
Apache's additional `.so` files must already be in `Apache24/modules/`.
Git must be a ZIP of the extracted portable installation, not its upstream `.7z.exe` installer.
VSCode's portable data directory and .NET's local cache directories are created by DevKit3.

## Arduino IDE

The `Arduino` release uses these executable paths relative to the archive root:

| OS | Executable |
| --- | --- |
| `win` | `Arduino IDE.exe` |
| `linux` | `arduino-ide` |
| `mac` | `Arduino IDE.app/Contents/MacOS/Arduino IDE` |

Use the actual distribution for each platform, not a renamed Windows archive. Preserve executable permissions and relative symbolic links when packaging Unix distributions. TAR.GZ packages can be used directly when their directory layout matches the paths above. Arduino installation restores ZIP symbolic links within the installation directory and checks that the expected executable exists before marking a version as installed.

In the project editor, add **Arduino IDE**, choose **Launch**, and optionally mark it **Primary**. Settings:

```text
open=Blink/Blink.ino
workingDirectory=.
```

Both settings are optional and resolve relative to the project root. `open` may identify a sketch or folder. The DevKit2 names `StartupFile` and `WorkingDirectory` are accepted as aliases.
Arduino IDE manages its own preferences, boards, libraries, and instance behavior; DevKit3 passes the selected project environment and sketch.

## Catalog and caching

The catalog loads at startup and before opening the project editor, with a ten-minute refresh interval. The published release index is read in batches, so tools without releases do not trigger individual failed requests or alerts.
**Sync** forces a fresh request. Asset listing is paginated, and versions are sorted numerically, newest first.
Catalog files live under `~/DevKit3/Data/downloads/catalog` (or `DEVKIT3_HOME/downloads/catalog`).
When requests fail, DevKit3 reports the failure and retains the previous catalog. Installed versions remain available.
A new Windows machine needs the releases published first, especially `Git`, which DevKit3 needs for configuration setup. On Linux, install Git through the distribution's package manager instead; DevKit3 discovers it from the machine PATH and does not download a Git asset.
Public releases require no token; a private repository is not supported by this unauthenticated catalog.

Keep published version assets unchanged. For a changed package, publish a new version: installed copies and downloaded ZIPs are cached.
When GitHub supplies an asset SHA256 digest, DevKit3 verifies both downloads and cached ZIPs before installation.

Adding versions for these tools requires only uploading assets. Adding a new type of software still requires registering its tool and defining its environment/start behavior in DevKit3.

## Verification

Run `dotnet build` and `dotnet run --project tests/ReleaseCatalog.Tests.csproj`.
The catalog checks use a fake HTTP server handler; they do not download software or require GitHub access.

Run `dotnet run --project tests/ArduinoIntegration/Arduino.Tests.csproj` to test Arduino installation and launch arguments with a small fixture package, without opening the IDE.
Append `-- --tar-fixture` to run the same installation checks with a TAR.GZ fixture.
To verify the published Windows 2.3.10 ZIP instead, append `-- <path-to-Arduino-win-x64-2.3.10.zip>`.
