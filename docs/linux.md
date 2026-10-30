# Running DevKit3 on Linux

Use DevKit3 in a desktop session as your normal user. Linux software packages must match your machine's architecture and distribution libraries; Windows executables cannot be used as Linux packages.

## Desktop requirements

Install Git, a terminal emulator, `xdg-open`, and a Secret Service password keyring. For Debian/Ubuntu with GNOME Keyring:

```bash
sudo apt install git xterm xdg-utils libsecret-tools gnome-keyring libx11-6 libice6 libsm6 libfontconfig1
```

Use your desktop's existing Secret Service provider if it already has one. The keyring must be available on your desktop session's D-Bus and unlocked. DevKit3 stores the configuration token and remembered vault password in this keyring using `secret-tool`; only variable names are saved in `keyring-names.json`. Other project secrets continue to use the encrypted, synced token vault. See the [secret-tool manual](https://manpages.debian.org/testing/libsecret-tools/secret-tool.1.en.html) and [Avalonia Linux requirements](https://docs.avaloniaui.net/docs/platform-specific-guides/linux).

Git is detected from the machine's `PATH`, including its actual version. A Windows project's Git version selection resolves to this local Git installation on Linux.

The **Terminal** tool keeps the stable project ID `Cmd`. DevKit3 detects x-terminal-emulator, GNOME Terminal, Konsole, Xfce Terminal, xterm, Alacritty, Kitty or foot. It opens your shell in the project directory with the project's environment. **Open folder** uses `xdg-open`.

## Software packages

Publish assets named `<ToolId>-linux-<x64|arm64>-<version>.tar.gz` or `.zip`. TAR.GZ is preferred because Linux distributions normally preserve executable modes and relative symbolic links in this format. ZIP links are also restored inside the installation directory. Never repackage a Windows binary under a Linux asset name.

For most tools, put the executable at the archive root or in `bin/`. A single distribution directory wrapping the package is also detected. For a different prepared layout, use `packageRoot=` and `executable=` in the tool settings. The Linux executable candidates are defined in [LinuxToolLayout.cs](../Tools/LinuxToolLayout.cs). Arduino expects `arduino-ide`; VSCode expects `code`. Node accepts `bin/node` and exports both the package root and its `bin` directory. .NET expects `dotnet` at the archive root. PHP expects `php.ini` at the package root and `php`/`php-cgi` there or in `bin`; preserve its Linux extension configuration.

Visual Studio, MSVC, WinLibs, Rust (MSVC), Notepad++, HeidiSQL, the Windows ILSpy adapter, ShareX, SumatraPDF, SuperPuTTY and WinMerge are excluded from new Linux tool selections. Existing project entries are preserved so you can edit them. Replace these tools or hide their projects on Linux.

## Services

- Apache uses native `bin/httpd` in the foreground, with bundled modules and `conf/mime.types`. Include a compatible MPM and all modules referenced by your configuration. PHP uses `mod_proxy`/`mod_proxy_fcgi` and `php-cgi`, with an automatically managed per-project FastCGI process. See [Apache FastCGI configuration](https://httpd.apache.org/docs/2.4/mod/mod_proxy_fcgi.html).
- MariaDB expects `bin/mariadbd`, `bin/mariadb-admin`, and `scripts/mariadb-install-db` or `bin/mariadb-install-db`. It initializes per-project data using the current Linux user, and uses a private socket/PID file and loopback TCP. See [MariaDB initialization options](https://mariadb.com/docs/server/clients-and-utilities/deployment-tools/mariadb-install-db).
- PostgreSQL uses `bin/postgres`, `bin/initdb` and `bin/pg_ctl`, with per-project data/logs and loopback TCP. Unix sockets are disabled to avoid system socket directories.
- Nginx, Caddy and Sphinx Search use native Linux executables with per-project configuration and process tracking.

Choose a web port such as `8080` on Linux if your account cannot bind port `80`. Database packages may need distribution libraries supplied alongside the binaries or installed on the machine.

## Build and checks

With the .NET 10 SDK installed:

```bash
dotnet run --project DevKit3.csproj
dotnet run --project tests/ReleaseCatalog.Tests.csproj
dotnet run --project tests/ArduinoIntegration/Arduino.Tests.csproj -- --platform-checks
```

Run these checks manually when needed. They use prepared fixture packages and a harmless native shell process; they do not launch every third-party application. Real GUI and packaged server smoke tests are still needed for each package you publish.
