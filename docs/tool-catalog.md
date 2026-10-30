# Tool catalog and DevKit2 migration

DevKit3 registers all **69 concrete applications** from DevKit2. The legacy WinForms profile dialogs are represented by the project editor's tool settings, extra PATH folders, and environment variables. They are not separate software tools.

## Packages and platforms

All portable tool versions come from the matching release tag (the exact Tool ID below). No version list or upstream installer URL is embedded in these adapters. Use prepared ZIP or TAR.GZ assets named `<ToolId>-<win|linux|mac>-<architecture>-<version>`.

The migration preserves DevKit2's **Windows** executable names and known folder layouts. Linux now has separate executable layouts, terminal/folder integration, desktop keyring storage, and native server configuration; see the [Linux guide](linux.md). Windows-only applications remain Windows-only, and macOS application integration is not complete. Package compatibility still requires testing the binaries and dependencies on the target distribution. DMG installers are not supported.

Prefer placing the ready-to-run installation at the archive root. The table shows the expected Windows files; common DevKit2 wrapper folders are also accepted. `{version}` in a path means the selected tool version.

| Tool ID / release tag | Display name | Windows package contents / executable | DevKit2 wrapper directories also accepted |
| --- | --- | --- | --- |
| `Apache` | Apache | `Apache24/bin/httpd.exe` | `—` |
| `Arduino` | Arduino IDE | `Arduino IDE.exe` | `—` |
| `Bitwarden` | Bitwarden | `Bitwarden.exe`, `Bitwarden-Portable-{version}.exe` | — |
| `Blender` | Blender | `blender.exe` | `blender-{version}-windows-x64` |
| `Bombardier` | Bombardier | `bombardier.exe` | — |
| `Bruno` | Bruno | `Bruno.exe` | — |
| `CMake` | CMake | `bin/cmake-gui.exe`, `cmake-gui.exe` | `cmake-{version}-windows-x86_64` |
| `Caddy` | Caddy | `caddy.exe` | `—` |
| `Cmd` | Command Prompt | `Windows cmd.exe; no release` | `—` |
| `CodeBlocks` | Code::Blocks | `codeblocks.exe` | — |
| `Codelite` | CodeLite | `codelite.exe`, `bin/codelite.exe` | — |
| `Composer` | Composer | `composer.phar` | — |
| `Consul` | Consul | `consul.exe` | — |
| `Curl` | curl | `bin/curl.exe`, `curl.exe` | `curl-{version}-win64-mingw` |
| `DotnetSDK` | .NET SDK | `dotnet.exe` | `—` |
| `Etcd` | etcd | `etcd.exe` | `etcd-v{version}-windows-amd64` |
| `Fd` | fd | `fd.exe` | `fd-v{version}-x86_64-pc-windows-gnu` |
| `FlutterSDK` | Flutter SDK | `bin/flutter.bat`, `bin/flutter` | `flutter` |
| `FreeCAD` | FreeCAD | `FreeCAD.exe`, `bin/FreeCAD.exe` | `FreeCAD_{version}-Windows-x86_64-py311` |
| `Fzf` | fzf | `fzf.exe` | — |
| `Geany` | Geany | `bin/geany.exe`, `geany.exe` | — |
| `Git` | Git | `bin/git.exe` or root `git.exe` (keep the full distribution); Linux uses system Git | `—` |
| `GithubCli` | GitHub CLI | `bin/gh.exe`, `gh.exe` | — |
| `Go` | Go | `bin/go.exe`, `bin/go` | `go` |
| `Godot` | Godot | `Godot.exe`, `Godot_v{version}-stable_win64.exe` | — |
| `GodotSharp` | Godot .NET | `Godot.exe`, `Godot_v{version}-stable_mono_win64.exe` | `Godot_v{version}-stable_mono_win64` |
| `HeidiSQL` | HeidiSQL | `heidisql.exe` | — |
| `ILSpy` | ILSpy | `ILSpy.exe` | — |
| `Inkscape` | Inkscape | `bin/inkscape.exe`, `inkscape.exe` | `inkscape` |
| `Java` | Java JDK | `bin/java.exe`, `bin/java` | `jdk-{version}` |
| `Kate` | Kate | `bin/kate.exe`, `kate.exe` | — |
| `Krita` | Krita | `bin/krita.exe`, `krita.exe` | `krita-x64-{version}` |
| `Llvm` | LLVM | `bin/clang.exe`, `bin/clang` | `clang+llvm-{version}-x86_64-pc-windows-msvc` |
| `Lua` | Lua | `lua.exe`, `lua{major}{minor}.exe` | — |
| `Mariadb` | MariaDB | `bin/mariadbd.exe or mariadb-<version>-winx64/bin/mariadbd.exe` | `—` |
| `Msvc` | MSVC | `VC/Tools/MSVC/<version>/ and Windows Kits/10/` | `—` |
| `NetBeans` | Apache NetBeans | `bin/netbeans64.exe`, `bin/netbeans.exe` | `netbeans` |
| `Nginx` | Nginx | `nginx.exe`, `conf/mime.types` | `—` |
| `Nim` | Nim | `bin/nim.exe`, `bin/nim` | `nim-{version}` |
| `Node` | Node.js | `node.exe or node-v<version>-win-x64/node.exe` | `—` |
| `Npp` | Notepad++ | `notepad++.exe` | — |
| `Php` | PHP | `php.exe`, `php.ini`, `ext/` | `—` |
| `Pixieditor` | PixiEditor | `Pixieditor.exe` | `PixiEditor` |
| `Pororoca` | Pororoca | `Pororoca.exe` | — |
| `PostgreSQL` | PostgreSQL | `bin/postgres.exe`, `bin/initdb.exe`, `bin/pg_ctl.exe` | `—` |
| `Processing` | Processing | `Processing.exe` | `Processing` |
| `Putty` | PuTTY | `PUTTY.EXE`, `putty.exe` | — |
| `Python` | Python | `python.exe`, `python3`, `bin/python3` | `python-{version}-embed-amd64` |
| `QtCreator` | Qt Creator | `bin/qtcreator.exe`, `qtcreator.exe` | — |
| `Ripgrep` | ripgrep | `rg.exe` | `ripgrep-{version}-x86_64-pc-windows-gnu` |
| `RustGcc` | Rust (GNU) | `bin/rustc.exe`, `rustc/bin/rustc.exe` | `rust-{version}-x86_64-pc-windows-gnu` |
| `RustMsvc` | Rust (MSVC) | `bin/rustc.exe`, `rustc/bin/rustc.exe` | `rust-{version}-x86_64-pc-windows-msvc` |
| `SQLiteStudio` | SQLiteStudio | `SQLiteStudio.exe` | `SQLiteStudio` |
| `ShareX` | ShareX | `ShareX.exe` | — |
| `Shotcut` | Shotcut | `shotcut.exe` | `Shotcut` |
| `SolveSpace` | SolveSpace | `solvespace.exe`, `solvespace_x64.exe` | — |
| `SphinxSearch` | Sphinx Search | `bin/searchd.exe` | `—` |
| `Spyder` | Spyder | `Scripts/spyder.exe`, `spyder.exe` | `python-{version}-embed-amd64` |
| `SumatraPDF` | SumatraPDF | `SumatraPDF.exe`, `SumatraPDF-{version}-64.exe` | — |
| `SuperPutty` | SuperPuTTY | `SuperPutty.exe` | `SuperPuTTY-{version}` |
| `Tabby` | Tabby | `Tabby.exe` | — |
| `TeaCli` | Forgejo/Gitea CLI (tea) | `tea.exe` | — |
| `Thonny` | Thonny | `thonny.exe` | — |
| `VSCode` | VSCode | `Code.exe`, `bin/`, `resources/` | `—` |
| `VSCodium` | VSCodium | `VSCodium.exe` | — |
| `VisualStudio` | Visual Studio | `System installation discovered with vswhere; no release` | `—` |
| `Winlibs` | WinLibs (MinGW-w64) | `bin/gcc.exe` | `mingw64` |
| `Winmerge` | WinMerge | `WinMergeU.exe` | `WinMerge` |
| `Zed` | Zed | `Zed.exe` | — |
| `Zig` | Zig | `zig.exe`, `zig` | `zig-x86_64-windows-{version}` |

## GUI tools and terminals

GUI tools support `open` (an optional file, project, or folder) and `workingDirectory`, relative to the project root. The DevKit2 names `StartupFile` and `WorkingDirectory` are aliases. Spyder preserves its `-p` project argument.

Command-line tools support **InPath**, or **Launch** to open a Windows Command Prompt with the project environment, as in DevKit2. The existing Git/PHP/Node/.NET integrations retain their existing InPath behavior. Cmd is a system tool and never downloaded.

### Git on Linux

Git is treated as a system tool on Linux. DevKit3 scans absolute entries in the machine PATH, verifies candidates with `git --version`, and shows the detected version as **Git (system)**. Startup and Sync rescan installed Git. Relative and empty PATH entries are ignored.

Config synchronization, repository cloning and project environments all use the detected executable. A synced project's Windows PortableGit version does not force a matching download on Linux: the local system Git is used regardless of that saved version. DevKit3 never installs or uninstalls system Git. If Git is missing, it asks the user to install Git with the distribution's package manager and retry. Windows retains prepared portable Git and its selected version.

This implements the Git integration only. Other application components, including the secret store, still require further Linux support before the full application can run there.

The new generic package adapters also accept `packageRoot` (directory relative to the installation) and `executable` (file path relative to the installation) for prepared packages with a different layout. These are explicit overrides, not automatic OS compatibility.

## SDK and dependency packages

- **MSVC:** package the VC toolchain and Windows SDK under the paths above. DevKit3 discovers the highest installed component versions numerically, or uses `msvcVersion` / `sdkVersion` settings. `host` and `target` default to `x64`. PATH, INCLUDE, LIB, and the SDK/compiler variables are constructed from the package. No compiler downloader or license acceptance is run by DevKit3.
- **Go:** adds the Go bin directory and GOPATH/bin, and sets GOROOT, GOPATH, GOCACHE, and GOTELEMETRYDIR.
- **Java / Flutter / LLVM:** set JAVA_HOME / FLUTTER_HOME / LLVM_HOME and LIBCLANG_PATH.
- **RustGcc / RustMsvc:** support the DevKit2 component layout and a merged sysroot layout, set CARGO_HOME and RUSTFLAGS, and add compiler/Cargo/rustfmt directories. Include the matching standard library and linker dependencies. GNU Rust can use Winlibs; MSVC Rust needs the MSVC toolchain.
- **Python / Spyder:** include Python, pip/site configuration, and all required packages in the prepared release. Spyder is a separate release containing its runnable Python environment; DevKit3 does not execute pip to install or reinstall packages.
- **Composer:** include composer.phar and add PHP to the same project. A Windows composer.bat wrapper is created if absent.
- **Lua:** supports lua.exe and the versioned executable name used in DevKit2; creates a Windows lua.cmd wrapper when needed.
- **NetBeans, Godot .NET and other dependent applications:** include their dependencies, or select the required Java/.NET/other tool in the project as appropriate.

## Services

Nginx, Caddy, PostgreSQL and SphinxSearch have their own processes and per-project instance directories. Services wait for their configured port before launch is considered successful. Stop attempts graceful shutdown where supported and falls back to process termination.

| Service | Settings | Behavior |
| --- | --- | --- |
| Nginx | `port=80`, `webRoot=public`, `phpPort=9000`, optional `config=` | Generates nginx.conf, logs and temp paths in the instance directory. Runs in the foreground. |
| Caddy | `port=80`, `webRoot=public`, `phpPort=9000`, optional `config=` | Generates a Caddyfile with automatic HTTPS and the admin endpoint disabled for local per-project instances. |
| PostgreSQL | `port=5432`, optional `dataDirectory=` | Initializes a new cluster only when PG_VERSION is absent. Binds to localhost; initial local development authentication follows DevKit2's trust setup. Uses pg_ctl for graceful shutdown. |
| SphinxSearch | `port=9312`, optional `config=`, `dataDirectory=`, `configDirectory=` | Creates a default RT index and local searchd config if no config exists. An existing config is preserved; update it yourself when changing its port or indexes. |

Nginx and Caddy start PHP FastCGI automatically when the project includes Php. The PHP package must include php-cgi.exe. By default the FastCGI port is the web port plus one; set phpPort explicitly if needed. The FastCGI process is tracked with its service and stopped when the service stops or exits.

For Nginx and Caddy, generated configs are rewritten on each start. Use `config=path/to/custom.conf` to preserve custom configuration. A custom config must use the same listener port as the tool setting so readiness checks can succeed. Paths such as `DataDirectory`, `ConfigDirectory`, `InstanceDirectory`, `WebRootDirectory` and `Port` retain their service-specific DevKit2 aliases. Additional PATH entries go in the editor's **Add to PATH** field.

## Tabby package layouts

The inspected Tabby 1.0.238 packages use `Tabby.exe` at the archive root on Windows (x64/arm64), `Tabby.app/Contents/MacOS/Tabby` on macOS (x64/arm64), and `tabby-{version}-linux-<architecture>/tabby` on Linux (x64/arm64). The portable tool resolver searches the Linux wrapper directory automatically. Keep the complete application package, including resources and libraries.

The ARM64 Linux archive named `Tabby-linux-arm64-1.0.238.gz` contains a TAR archive. Rename it to `Tabby-linux-arm64-1.0.238.tar.gz` before publishing: release discovery and extraction accept `.zip` and `.tar.gz`.

## Release discovery and verification

DevKit3 reads the published release index in batches, uses embedded assets for ordinary releases, and paginates large asset lists. Unpublished tool tags are skipped without generating one error for each absent package. Network failures retain cached versions; system tools still refresh locally.

The migration tests check the inventory against a fixture extracted from DevKit2, representative executable layouts and launch arguments, SDK environments, MSVC component discovery, service configuration, and process/auxiliary cleanup using harmless loopback-listener fixtures. They do not claim that all 69 real applications have been launched successfully. Those require the actual prepared packages and OS-specific testing.

Run:

```powershell
dotnet build DevKit3.csproj
dotnet run --project tests/ReleaseCatalog.Tests.csproj
dotnet run --project tests/ArduinoIntegration/Arduino.Tests.csproj
```
