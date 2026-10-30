# DevKit3

**Your projects. The right tools. Ready to work.**

DevKit3 is a desktop app for Windows and Linux that brings your development projects and tools together in one place. Choose what each project needs, then open it with the right software versions and services ready to use.

Working on an older PHP application and a new Node.js project? Keep a separate setup for each and switch between them without repeatedly configuring your tools.

## What You Can Do

- **Organize your projects.** Keep your work in one place and arrange related projects into groups.
- **Choose the right versions.** Use different versions of PHP, Node.js, .NET, and other tools for different projects.
- **Let DevKit3 install missing tools.** Select the tools your project needs and let the app handle their installation.
- **Open VSCode ready to work.** Start your editor with the tools and settings selected for that project.
- **Run local websites and databases.** Start Apache and MariaDB for your projects, with separate settings and database data.
- **Set up your source code.** Add your project's Git repositories so DevKit3 can clone them for you.
- **Keep project settings together.** Manage environment variables and access tokens alongside each project, with password-protected storage for sensitive values.
- **Take your setup to another machine.** Sync your project list and settings, or choose to keep everything on one computer.
- **Control running projects.** Open projects, check what is running, and stop their tools and services when you are finished.

## Supported Tools

DevKit3 includes integrations for 69 tools, including:

| Category | Tools |
| --- | --- |
| Editors and IDEs | VSCode, VSCodium, Visual Studio, Arduino IDE, Code::Blocks, CodeLite, Geany, Kate, Apache NetBeans, Qt Creator, Zed, Spyder, Thonny, Processing, Notepad++, ILSpy |
| Languages and SDKs | .NET SDK, PHP, Node.js, Java JDK, Python, Go, Rust (GNU), Rust (MSVC), Flutter SDK, Nim, Lua, Zig |
| C/C++ build tools | MSVC, WinLibs (MinGW-w64), LLVM, CMake |
| Databases and search | MariaDB, PostgreSQL, Sphinx Search, SQLiteStudio, HeidiSQL |
| Web servers | Apache, Nginx, Caddy |
| APIs and networking | Bruno, Pororoca, curl, Bombardier, PuTTY, SuperPuTTY, Consul, etcd |
| Git and command-line utilities | Git, GitHub CLI, Forgejo/Gitea CLI (tea), fd, fzf, ripgrep, Command Prompt, Composer |
| Design, graphics and games | Blender, FreeCAD, Godot, Godot .NET, Inkscape, Krita, PixiEditor, Shotcut, SolveSpace |
| Desktop utilities | Bitwarden, ShareX, SumatraPDF, WinMerge |

Available versions depend on the packages provided for each tool. Visual Studio is detected from the installations on your computer.

## Save Your Project List to Git

Store your project list in a dedicated Git repository to back it up and use the same setup on another computer.

1. Create an **empty, private repository**, for example `devkit3-config`, on your Git hosting service.
2. Create an access token with **read and write** access to that repository. DevKit3 needs write access to push changes. See [Creating access tokens](docs/manual.md#3-creating-access-tokens) for provider-specific instructions.
3. Open **Settings** in DevKit3 and select **Git repository (synced between machines)**.
4. Enter the connection details:

   | Field | Value |
   | --- | --- |
   | Repository URL | The HTTPS clone URL, such as `https://github.com/me/devkit3-config.git` |
   | Token variable name | Keep `DEVKIT3_CONFIG_TOKEN`, or choose a name for this repository's token |
   | User | Leave empty for GitHub; use the username required by your Git provider otherwise |
   | Token | Paste the access token; leave empty to keep a token already saved on this machine |

5. Click **Save**. DevKit3 connects the repository and syncs your project list. An existing local list is merged with projects already in the remote repository.

Changes saved to your projects and groups are committed and synced automatically. Use **Sync** in the toolbar to pull changes from other machines and push pending local changes, including after reconnecting to the network.

The repository stores project definitions in `devkit3-projects.json` and password-encrypted variable values and project tokens in `devkit3-tokens.json`. Project source code stays in its own repositories; installed tools, project folder paths and standalone Launch configurations stay on each machine.

On another computer, enter the same repository URL and an access token in **Settings**. Unlock the encrypted values with the same tokens password when prompted, then choose the local folder for each project. The config repository's access token is stored locally on each machine.

## A Typical Workflow

1. Create a project and choose its folder.
2. Select the tools and versions you want to use.
3. Add any repositories, variables, or service settings your project needs.
4. Open the project and start working.

DevKit3 takes care of preparing your selected tools so you can spend more time on your code.

## Platform

DevKit3 supports **Windows and Linux**. Available tools depend on your operating system and the published packages. Windows-only software, including Visual Studio and MSVC, requires Windows.

For Linux setup and requirements, see the [Linux guide](docs/linux.md).

## Help and Feedback

- Read the [User Manual](docs/manual.md) for setup and usage instructions.
- [Report a bug or suggest a feature](https://github.com/minhnguyenerp/devkit3/issues).
