# DevKit3 manual

DevKit3 manages your software projects on Windows and Linux. For each project it knows which tools to install and start (VSCode, MariaDB, Apache, ...), which tools to put on `PATH` (PHP, Git, Node, ...), which environment variables to set and which git repositories to clone. Open a project and everything is ready; move to another machine and you get the same setup by entering a URL and a few tokens.

On Linux, install Git, a desktop terminal, `xdg-utils`, and `secret-tool` with an unlocked Secret Service keyring. Local credentials use the desktop keyring; Windows uses DPAPI. Project tokens still use the encrypted shared vault. Use Linux tool packages and replace Windows-only tools in projects you open on Linux. Choose an unprivileged web port such as `8080` when necessary.

The macOS terminal integration opens Terminal.app with the selected working directory and tool environment. It uses the configured shell, falling back to zsh or bash. This terminal support does not imply full macOS application support: the local secret store still needs a macOS Keychain implementation.

- [1. Concepts](#1-concepts)
- [2. First start: the config repository](#2-first-start-the-config-repository)
- [3. Creating access tokens](#3-creating-access-tokens)
- [4. Projects and groups](#4-projects-and-groups)
- [4.1 Standalone Launch](#41-standalone-launch)
- [5. Tools](#5-tools)
- [6. Variables](#6-variables)
- [7. Repositories](#7-repositories)
- [8. How git authentication works](#8-how-git-authentication-works)
- [9. Opening and stopping projects](#9-opening-and-stopping-projects)
- [10. Setting up another machine](#10-setting-up-another-machine)
- [11. Where DevKit3 keeps its data](#11-where-devkit3-keeps-its-data)
- [12. Troubleshooting](#12-troubleshooting)
- [13. Git cheatsheet](#13-git-cheatsheet)

---

## 1. Concepts

DevKit3 splits everything into what is **shared** between your machines and what stays **on this machine**.

| Shared (in the config repository) | On this machine only |
|---|---|
| Groups and projects | Where each project folder is |
| Tools of each project, their versions and settings | Installed tools |
| Variable names, repository URLs and auth variable names | The token of the config repository itself |
| Variable **values** and tokens, **encrypted with your tokens password** | The remembered tokens password (optional) |
| | Running processes |

The shared part lives in a git repository of your own (the **config repository**) as two files:

- `devkit3-projects.json`: groups, projects, tools, variable names, repositories. No secrets.
- `devkit3-tokens.json`: every variable value and repository token, each one encrypted with a password you choose (see [section 6.1](#61-the-tokens-file-and-its-password)).

So a new machine needs only three things: the config repository URL, its token, and the tokens password.

---

## 2. First start: the config repository

On first start DevKit3 asks **Where is the project list stored?** (also available later with **Settings** in the toolbar).

### Option A: Git repository (recommended)

The project list is synced between your machines.

1. Create an **empty, private** repository on GitHub, GitLab, Gitea, Bitbucket, ... for example `devkit3-config`. Nothing needs to be in it.
2. Create an access token for it with **write** access (see [section 3](#3-creating-access-tokens)). Write access is needed because DevKit3 commits and pushes every change you make to the project list.
3. In DevKit3 fill in:

| Field | What to enter |
|---|---|
| Repository URL | The **HTTPS** URL, e.g. `https://github.com/me/devkit3-config.git`. SSH URLs (`git@...`) do not use tokens and are not supported here. |
| Token variable name | Keep `DEVKIT3_CONFIG_TOKEN`. It is only the name under which the token is saved. This token stays on this machine (encrypted for your Windows account): it is needed to download the tokens file itself. |
| User | Leave empty for GitHub. See the table in [section 3](#3-creating-access-tokens) for other services. |
| Token | The token you just created. Leave empty if you entered it before. |

4. Click **Save**. DevKit3 installs its own portable Git if needed, clones the repository and shows the project list.

### Option B: This machine only

The project list stays on this machine (in a local git repository without remote). You can connect it to a remote repository later: open **Settings**, choose **Git repository** and enter the URL and token. If that repository already holds projects, the two lists are merged (see [section 4.1](#41-working-on-several-machines)).

### Choosing another project file

Without arguments, DevKit3 uses `devkit3-projects.json`. To use another project list in the **same configuration repository**, launch it with `--project-file=my-project.json` or `--project-file my-project.json`:

```powershell
# Windows
.\DevKit3.exe --project-file=my-project.json
```

```sh
# Linux/macOS executable, when built for that operating system
./DevKit3 --project-file=my-project.json

# Running the .NET application directly, on any supported OS
dotnet DevKit3.dll --project-file=my-project.json
```

The value is a `.json` filename in the repository root, not a path relative to the terminal. Quote filenames containing spaces, such as `--project-file="my projects.json"`. Paths, reserved filenames and `devkit3-tokens.json` are rejected consistently on all platforms. A missing file starts with an empty project list and is created on the first save; existing lists are not renamed or copied. The selected filename appears in Settings and the status bar, and is used for loading, saving, committing and project conflict resolution.

The option applies to that launch; supply it again (or add it to the shortcut/launcher) to return to that list. Repository settings, local tool installations, tracked processes and `devkit3-tokens.json` remain shared. Cleanup checks all project JSON files in the repository, including lists other than the one currently selected, and shows each reference's filename. If another file has conflicting changes on both machines during sync, sync stops without replacing local commits so that conflict can be resolved first.

### 2.1 Updating DevKit3

On Windows x64, open **Settings → Check for update**. DevKit3 checks the [DevKit3 release](https://github.com/minhnguyenerp/devkit3/releases/tag/DevKit3) for a newer `DevKit3-win-x64-<version>.exe` and shows the current version, new version and download size before asking whether to download and install. Same or older versions are not installed. Save any Settings changes before updating.

After approval, the installer downloads into a unique folder under `%TEMP%\DevKit3Updates`. Progress and **Cancel update** are shown in Settings. DevKit3 validates the download size and release SHA-256 checksum, starts the installer as a separate process, then exits so you can complete the normal installation wizard. If downloading, validation, or launching the installer fails (including canceled Windows elevation), DevKit3 stays open and shows the error.

On the next application start, temporary installers and partial downloads are cleaned in the background. An active download is protected by a file lock. If the installer is still holding its file, cleanup retries and leaves it for a later start if necessary. Other temporary files, installed tools, projects and service data are kept. This application update check is independent of each project tool's Auto Update setting.

---

## 3. Creating access tokens

DevKit3 uses **personal access tokens** over HTTPS, both for the config repository and for project repositories. A token is like a password limited to some repositories and permissions; you can revoke it at any time.

Which permission do you need?

| Used for | Permission |
|---|---|
| Config repository | Read **and write** (DevKit3 pushes the project list) |
| Project repository you only clone/pull | Read |
| Project repository you push to (VSCode, terminal) | Read and write |

### GitHub: fine-grained token (recommended)

1. Open **Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token** (direct link: `https://github.com/settings/personal-access-tokens/new`).
2. **Token name**: something you will recognise, e.g. `DevKit3 config` or `DevKit3 ACME repos`.
3. **Expiration**: pick a date. You will enter a new token when it expires (see [section 7.4](#74-when-a-token-expires)).
4. **Resource owner**: your account, or the organization owning the repositories.
5. **Repository access**: **Only select repositories**, then pick the repositories this token is for.
6. **Permissions → Repository permissions → Contents**: **Read and write** (or **Read-only** for repositories you never push to).
7. Click **Generate token** and copy the value (`github_pat_...`). GitHub shows it **only once**.

Note: a fine-grained token only works for the repositories selected in step 5. When you add a new repository to a project and want to reuse this token, edit the token on GitHub and add the repository.

### GitHub: classic token

**Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token**, tick the **repo** scope. The value starts with `ghp_`. A classic token works for every repository you can access, which is convenient but less safe.

### GitLab

**Preferences (your avatar) → Access tokens → Add new token**. Tick **read_repository**, and **write_repository** if you push. Project access tokens (**Project → Settings → Access tokens**) also work.

### Gitea / Forgejo

**Settings → Applications → Generate new token**. Under **repository** choose **Read and write** (or **Read**).

### Bitbucket Cloud

Open the repository → **Repository settings → Access tokens → Create access token**. Tick **Repositories: Read**, and **Write** if you push.

### What to put in the "User" field

| Service | User |
|---|---|
| GitHub | Leave empty (DevKit3 sends `x-access-token`) |
| GitLab | Your GitLab username |
| Gitea / Forgejo | Your username |
| Bitbucket repository access token | `x-token-auth` |

---

## 4. Projects and groups

The left side of the main window is a tree of **groups** and **projects**. A green dot means something of that project is running.

- **+ Group** creates a group. Select a group to **Rename** or **Delete group** it; deleting a group moves its projects to **(No group)**.
- **+ Project** creates a project (inside the selected group). New projects start with VSCode (primary) and Git.
- Select a project and click **Edit** to change it, **Delete** to remove it from the list. Deleting never removes the project folder or its data on disk.
- In **Edit → General → Show project on**, use the **Windows**, **macOS**, and **Linux** checkboxes to choose which operating systems show the project. All three are enabled by default, including for existing projects. These choices sync with the project. Enable **Show hidden projects** above the project list to find a hidden project and change its settings again. Hiding a project does not stop running tools or delete its files; these checkboxes only control list visibility.

Every change is saved to `devkit3-projects.json`, committed and pushed to the config repository. **Sync** does both directions: it gets the changes made on your other machines and pushes the ones made here. DevKit3 also syncs when it starts.

### 4.1 Working on several machines

DevKit3 merges the project list **by group and by project**, not line by line:

- Changes to **different** projects or groups on two machines are combined automatically. For example, you add *Shop* on your laptop and edit *API* on your desktop: after syncing, both machines have both changes.
- When the **same** project or group was changed on both machines since the last sync (or deleted on one and edited on the other), DevKit3 shows **Changed on two machines**. For each item, pick **Keep this machine's version** or **Keep the other machine's version**, then click **Apply and sync**. Everything else is still merged automatically.
- **Cancel** in that window leaves everything as it is: your changes stay on this machine and are not pushed. The question comes back at the next save or **Sync**.
- If a save could not be pushed (no network, expired token), the change is kept on this machine and pushed by the next save or **Sync**.

The history of the config repository shows one commit per change, with the machine name on merged ones, so you can always see who changed what.

### General tab

| Field | Meaning |
|---|---|
| Project name | Shown in the tree. |
| Group | Group of the project. |
| Project folder on this machine | Where the project lives **on this machine**. Not shared. Empty means `~/DevKit3/Projects/<project name>`. Repositories are cloned inside it and relative paths in tool settings start from it. |

---

### 4.1 Standalone Launch

Use **Projects** and **Launch** in the top toolbar to switch views. **Show hidden projects** is also in this toolbar.

Launch lists every tool DevKit3 supports on the current operating system. Search by name, then double-click an application or click **Launch** to run it without creating a project. SDKs and command-line tools can be selected as helpers.

Choose the application's version, optional working directory (defaults to your home folder), tool-specific settings, extra PATH folders and environment variables. Relative paths use the working directory. Add helper tools with **+ Helper tool**: **InPath** contributes PATH and environment; **Launch** also starts the helper before the main application. Missing portable versions are installed automatically.

**Launch** and **Terminal** automatically save configuration. These profiles, including manually entered environment values as plain text, are stored in this machine's local.json and are not synced with the project repository. Tool settings use the same keys documented in section 5.

Every tool has **+ Add variable**, with name, masked value and description fields like the Project editor. Values are encrypted in the same `devkit3-tokens.json` vault used by Projects and synced to other machines. The same variable name reuses the same saved value across Projects and Launch tools. Leave a value empty to keep its saved value. Launch and Terminal apply the selected variables to the application, helpers and terminal; local.json stores only their names and descriptions.

**Stop** stops the selected application's tools and helpers. Running tools reuse their current process; stop them and launch again to apply configuration changes. **Stop all** in the main toolbar stops tools across both Projects and Launch.

### 4.2 Cleanup

Open **Cleanup** in the toolbar to see installed portable tool versions, their sizes, and why each version is kept. Select an **Unused** version and click **Clean selected version** to remove it. Usage is checked again before deletion. References from all projects (including hidden projects), saved Launch configurations, Auto Update's latest installed version, and active tools protect installations. Only the Git installation bound to the current DevKit3 session is protected for configuration and repository operations; other Git versions can be removed when unreferenced. Project references distinguish the selected version, Auto Update's selected fallback, and its current installed version. System installations are not managed here.

**Clean downloads** removes cached packages and partial downloads; it keeps the offline version catalog and installed tools. Packages are downloaded again if needed. Cleanup keeps project folders, settings and per-project service data. Scanning and deleting run in the background; use **Refresh** to update the list after other changes.

New launches record the exact versions of the application and its PATH dependencies, so running projects protect the versions they actually launched with even if the project is edited later. Older process records without dependency snapshots protect their own recorded tool version and versions referenced by the current project/Launch configuration; missing dependency metadata does not block every installed version.

---

## 5. Tools

The **Tools** tab lists the tools of the project. For each tool choose:

| Field | Meaning |
|---|---|
| Tool and version | Portable tools are downloaded and installed by DevKit3 the first time a project needs them, under `~/DevKit3/Data/apps`. Projects using the same version share one installation. |
| Auto Update | Off by default: keep the selected version. Turn it on to check for and use the latest available version when opening the project; missing portable versions are installed automatically. The selected version remains the fallback and is used again if you turn Auto Update off. System tools use the latest locally installed version. Stop and reopen running tools to apply an update. |
| Launch / InPath | **Launch**: started when the project opens. **InPath**: only added to `PATH` (and its variables set), so you can call it from a terminal. |
| Primary | The main tool, started last after the services (usually VSCode). Only one per project, Launch mode only. |
| Settings | Tool options, one `key=value` per line. An empty box shows an example. |
| Add to PATH | Extra folders put in front of `PATH`, one per line. Relative paths start from the project folder. |
| Environment variables | `KEY=VALUE` lines set for this project. These are **shared** in the project file: do not put passwords here, use the Variables tab. |

### Available tools

DevKit3 includes all 69 application integrations from DevKit2. Additional editors, SDKs, command-line utilities and desktop applications use **Launch** / **InPath** and the optional `open=` and `workingDirectory=` settings. See the [full tool catalog](tool-catalog.md) for release IDs, package layouts, dependencies and service settings. A tool without a published package has no downloadable versions yet.

| Tool | Modes | Settings | What it sets |
|---|---|---|---|
| VSCode | Launch, InPath | `open=` file or folder to open, relative to the project folder (default: the project folder) | Each project gets its own VSCode instance, so its terminals and Source Control see the project environment. Extensions are shared per VSCode version. |
| Arduino IDE | Launch, InPath | `open=` sketch or folder (optional), `workingDirectory=` relative to the project folder (default: the project folder) | Opens Arduino IDE with the project's environment. Set it as Primary when Arduino is the project's main editor. DevKit2 setting names `StartupFile` and `WorkingDirectory` are also accepted. |
| Visual Studio | Launch, InPath | `open=` solution, project or folder (optional), `workingDirectory=` relative to the project folder (default: the project folder) | Uses a locally installed Windows IDE. Lists detected editions and Preview installations; does not download or uninstall Visual Studio. The selected IDE directory is added to `PATH`. DevKit2 setting names `StartupFile` and `WorkingDirectory` are also accepted. |
| Git | InPath | – | Portable Git on Windows; detected system Git on Linux. Adds the selected executable's directory to `PATH`. |
| PHP | InPath | – | `php` on `PATH`, `PHPRC` |
| MariaDB | Launch, InPath | `port=` (default 3306) | One server **per project** with its own data. `MYSQL_TCP_PORT` so the `mariadb` client connects to it. |
| Apache | Launch, InPath | `port=` (default 80), `webRoot=` relative to the project folder | One server **per project**. Loads PHP when the project also uses PHP. |
| Node.js | InPath | – | `node`, `npm` on `PATH`; `npm install -g` goes next to node |
| .NET SDK | InPath | – | `dotnet` on `PATH`, `DOTNET_ROOT`, its own NuGet cache |

Every project also gets `DEVKIT3_PROJECT_ID`, `DEVKIT3_PROJECT_NAME` and `DEVKIT3_PROJECT_ROOT`.

### Visual Studio

Install Visual Studio using Visual Studio Installer, then click **Sync** in DevKit3 to rescan. In **Edit → Tools**, add **Visual Studio** and select the installed version and edition. Choose **Launch** and optionally **Primary** to use it as the main editor. Example settings:

```text
open=MySolution.sln
workingDirectory=.
```

The version list is discovered from this machine, without a fixed list of release years. Build Tools-only installations and incomplete or unlaunchable IDEs are excluded. Normal selections are preserved across patch updates. Another machine needs a matching installed edition; DevKit3 will report a missing installation rather than switch silently.

Visual Studio discovery uses Microsoft's `vswhere`, included with Visual Studio Installer. Without it, the list is empty. The IDE directory on `PATH` makes `devenv` available; this does not initialize a Visual Studio Developer Command Prompt or add every compiler toolchain.

### Running services

Nginx, Caddy, PostgreSQL and SphinxSearch are also available as per-project services. Nginx/Caddy support `port=`, `webRoot=`, optional `config=`, and `phpPort=` when the project includes PHP. PostgreSQL uses `port=5432` and optional `dataDirectory=`; SphinxSearch uses `port=9312` and optional `config=`. Service data is kept in each project's instance directory by default. Select **Launch** to start a service when opening the project.

Etcd runs as a local single-member server per project or Launch instance. Settings: `port=2379` (client), `peerPort=2380`, `name=default`, `logLevel=info`, optional `dataDirectory=` and `instanceDirectory=`. Both listeners bind to `127.0.0.1`. Data defaults to the instance's `data` folder; logs go to `logs/etcd.log`. Terminal sets `ETCDCTL_ENDPOINTS` for `etcdctl`. Simultaneous instances need distinct client and peer ports. Keep the member name and peer port unchanged when reusing data; existing membership changes require `etcdctl member update`. Stop targets the recorded PID and start time. Windows Stop forcibly terminates only that process: upstream Etcd 3.7.3 has no graceful interrupt handler on Windows. Linux/macOS Stop sends SIGTERM and waits up to 30 seconds for a clean shutdown; on failure it reports an error and retains tracking rather than force killing the server.

Consul runs as a local single-node server per project or Launch instance. Settings: `mode=server`, `port=8500` (HTTP/UI), `dnsPort=8600`, `serverPort=8300` (RPC), `lanPort=8301`, `wanPort=8302`, and `logLevel=info`. Use distinct values for all five ports across simultaneous instances. All listeners bind to localhost; HTTPS and gRPC listeners are disabled. UI is at `http://127.0.0.1:8500/ui/` (adjust the HTTP port). Persistent server data lives in `data`, generated configuration in `consul.json`, and rotating logs in `logs` inside the instance directory. Windows server instances select BoltDB to avoid the new-directory WAL sync failure. Optional `dataDirectory=` and `instanceDirectory=` override paths. `mode=dev` uses separate `dev-data` and temporary in-memory KV storage; it loses KV data on restart. Terminal sets `CONSUL_HTTP_ADDR` for this instance. Stop verifies the recorded PID/start time and node identity, requests graceful leave via the local agent API, and waits for exit. If shutdown fails, tracking is retained and an error is reported without force killing.

Two projects can run MariaDB (or Apache) at the same time as long as they use different ports. Always set `port=` explicitly: DevKit3 refuses to save two projects that set the same port, and when starting it reports a port that is already in use.

### Customising MariaDB and Apache

DevKit3 writes `my.ini` (MariaDB) and `httpd.conf` (Apache) again on every start. To add your own options create, in the instance folder of the project (`~/DevKit3/Data/instances/<project id>/mariadb` or `.../apache`):

- `custom.ini` for MariaDB, e.g. `[mariadbd]` then `max_connections=500`
- `custom.conf` for Apache, e.g. extra `<VirtualHost>` blocks

The MariaDB data, error log and Apache logs are in the same folders.

---

## 6. Variables

Use the **Variables** tab for anything the project needs from the environment that depends on the machine or must stay private: passwords, API keys, account names.

| Field | Meaning |
|---|---|
| Name | Variable name, e.g. `SECRET_PASSWORD`. Letters, digits and `_`, not starting with a digit. **Shared** with your other machines. |
| Value | The value, encrypted in `devkit3-tokens.json` and synced to your other machines. Leave empty to keep the saved value; the box then shows *Saved (leave empty to keep)*. |
| Description | Hint shown when DevKit3 asks for the value. |

Things to know:

- **One value per name.** Values are stored by name, not by project. Every project that uses `SECRET_PASSWORD` gets the same value. Use different names if projects need different values (e.g. `SHOP_DB_PASSWORD`, `API_DB_PASSWORD`).
- **Asked when missing.** If a variable has no value yet, **Open** and **Terminal** first show *missing values* and ask for it.
- **Windows environment fallback.** If no value is saved but Windows already has an environment variable with that name, DevKit3 uses it.
- **Only inside the project.** Values are set for the processes DevKit3 starts (tools, VSCode, Terminal). They are never written to the Windows environment.

### 6.1 The tokens file and its password

All variable values and repository tokens are kept in `devkit3-tokens.json`, next to `devkit3-projects.json` in the config repository, and synced with it. Each value is encrypted with **AES-256-GCM**, using a key derived from your **tokens password** (PBKDF2-SHA256, 600 000 iterations). The names are readable, the values are not.

- **Creating the password.** The first time DevKit3 needs to save a value, it asks you to **create the tokens password** (at least 8 characters). Use the same password on all your machines.
- **Unlocking.** On another machine, or after the password was forgotten, DevKit3 asks for the password when it needs a value (opening a project, Terminal, editing a project).
- **Remember on this machine.** Ticked by default: the password is saved for your account (Windows DPAPI or the Linux desktop keyring) so DevKit3 does not ask again. **Settings → Forget the tokens password** removes it.
- **It cannot be recovered.** Without the password the values cannot be read. If it is lost, delete `devkit3-tokens.json` from the config repository and enter the values again.
- **Choose a strong password**, especially if the config repository is shared: anyone with access to the repository can try to guess it offline.
- **Upgrading from an older DevKit3.** Values that older versions saved only on this machine are moved into the tokens file the first time it is unlocked.
- **Two machines changing values.** Values changed on different machines are merged by name. If the same value was changed on both, the machine that syncs last wins. If two machines each created their own tokens file before syncing (different passwords), the one already in the repository is kept and DevKit3 lists the values to enter again.

---

## 7. Repositories

The **Repositories** tab lists the git repositories cloned into the project folder.

| Field | Meaning |
|---|---|
| Folder | Sub-folder of the project folder, e.g. `src`, `theme`, `apps\api`. `.` clones into the project folder itself. When combining a root repository with independent child repositories, ignore those child folders in the root repository. Do not use absolute paths: they are shared and may not exist on other machines. |
| URL | HTTPS URL of the repository. |
| Branch | Branch to clone. Empty means the repository's default branch. |
| Auth variable | **Name** of the variable holding the token, e.g. `GH_TOKEN_ACME`. Empty for public repositories you never push to. |
| User | See [the table in section 3](#what-to-put-in-the-user-field). Empty means `x-access-token` (GitHub). |
| Token | The token, encrypted in `devkit3-tokens.json` under the auth variable name and synced. Leave empty to keep the saved token. |

After you save a new project, DevKit3 creates its folder, clones missing configured repositories with `git clone --depth 1`, and runs `git submodule update --init --recursive --depth 1` only immediately after each new clone. The selected branch is used for a new clone; leaving it empty uses the remote's default branch. Repositories without submodules need no special configuration. Creating the project does not install or launch its tools.

These shallow checkouts include the current code with minimal history. Submodules still use the exact commits recorded by the parent. For history-dependent work or builds, fetch more history using `git fetch --deepen=100` or `git fetch --unshallow` in the affected repository/submodule; fetch other branches or older tags explicitly when needed. Existing checkouts are not converted to shallow repositories. The configuration repository retains its full history because project-list sync needs common ancestors for merging changes and deletions.

Opening a project checks for missing clones. Existing checkouts (including repositories using a `.git` file) skip submodule updates, and are not pulled, reset or switched to another branch. The destination of a new clone must be empty or missing. A root repository is prepared before independent child repositories. During a new clone's setup, submodules are updated to the commits recorded by their containing repositories, without `--force`. If submodule setup fails after cloning, the saved project and files remain; fix the cause and run `git submodule update --init --recursive` manually in that repository. Reopening the project will keep skipping submodule setup once the repository exists. Private submodules need access through the configured token or your existing Git authentication.

Repository preparation shows the current operation (checking, cloning, configuring authentication, or initializing submodules), destination folder, elapsed time, Git PID, and its most recent output. Git's transfer progress is streamed as it arrives. After 30 seconds without output, the status explicitly says Git or a helper is waiting; elapsed time alone is not evidence of download progress. Errors include Git's diagnostic output with credentials redacted. **Cancel** stops the active Git process and its child helpers. Existing checkouts explicitly report that cloning and submodule updates were skipped.

### 7.1 Using the same token in many repositories and projects

Tokens are saved **by auth variable name**. Use the same name everywhere the same token applies, and you enter it **only once** (it is synced to your other machines):

| Project | Folder | URL | Auth variable |
|---|---|---|---|
| Shop ABC | `src` | `https://github.com/acme/shop.git` | `GH_TOKEN_ACME` |
| Shop ABC | `theme` | `https://github.com/acme/theme.git` | `GH_TOKEN_ACME` |
| API Gateway | `src` | `https://github.com/acme/api.git` | `GH_TOKEN_ACME` |

Use **different names** for different tokens: another account, another server (`GITEA_TOKEN`), or a fine-grained token limited to other repositories. If two repositories share a name, they share the token.

### 7.2 Adding a repository to a new project with an existing token

1. In the new project, **Repositories → + Add repository**.
2. Enter the folder and URL.
3. Type the **same auth variable name** as before. The token box shows *Saved (leave empty to keep)*: the token is already saved.
4. Leave the token empty and click **Save**. A new project is prepared immediately; use **▶ Open** when you want to launch its tools.

If the token box shows *Token* instead, the name does not match any saved token: check the spelling (names are case-sensitive) or enter the token.

With a GitHub fine-grained token, also add the new repository to the token on GitHub ([section 3](#github-fine-grained-token-recommended)), or cloning fails with an authentication error.

### 7.3 Changing the token

Edit any project that uses the auth variable, type the new token in the **Token** box of one of its repositories and click **Save**. Every repository and project using that name now uses the new token.

### 7.4 When a token expires

Git reports `Authentication failed` (or HTTP 401/403). Create a new token ([section 3](#3-creating-access-tokens)) and enter it as in [section 7.3](#73-changing-the-token). For the config repository, open **Settings** and enter the new token there.

---

## 8. How git authentication works

For each repository DevKit3 adds a credential helper to its `.git\config`. When git needs to authenticate (`clone`, `pull`, `push`, `fetch`), the helper reads the token from the environment variable named by the auth variable. The token itself is never written to `.git\config`, and the Windows credential manager is turned off for that repository so no login window pops up.

The project environment contains the token, so authentication works **automatically, without any password prompt**:

- in VSCode started with **▶ Open**: its terminals and the Source Control view (commit, push, pull, sync);
- in a terminal started with **Terminal**;
- when DevKit3 clones the repositories.

Terminals and tools launched by DevKit3 also inherit URL-scoped Git credential helpers through the process environment. Recursive Git commands, including `git submodule update --remote --recursive`, receive these helpers automatically; no manual `git -c credential.helper=...` arguments are needed. A single configured token/user for a server also applies to submodules under other paths on that same server. Credentials are not forwarded to a different server unless that server is separately configured in the project. If the project defines different credentials for different repositories on the same server, helpers remain scoped to those repository URLs rather than guessing which token an unrelated submodule should use. Existing inherited Git configuration is preserved, and helper definitions contain variable names rather than token values.

After changing tokens or updating DevKit3, open a new project **Terminal**, or close and reopen the editor through DevKit3. Already-running terminals and editors retain their old environment.

It does **not** work when git runs outside DevKit3 (a normal command prompt, VSCode started from the Start menu, GitHub Desktop, ...). There the variable does not exist, and git fails with an authentication error instead of asking for a password. Start your tools from DevKit3, or, if you really need it, set the variable (e.g. `GH_TOKEN_ACME`) as a Windows user environment variable. Be aware that a Windows environment variable is stored unencrypted.

---

## 9. Opening and stopping projects

| Button | What it does |
|---|---|
| ▶ Open | Asks for missing values, installs missing tools, clones missing repositories, then starts the Launch tools: services (MariaDB, Apache) first, the primary tool (VSCode) last. Tools already running are not started twice. |
| ■ Stop | Stops the tools and services of the selected project. |
| ■ Stop all | Stops everything DevKit3 started, in every project. |
| Terminal | Command prompt in the project folder with the project `PATH` and variables. Open the project once first so its tools are installed. |
| Folder | Opens the project folder in Explorer. |

Closing VSCode or switching to another project leaves the services (databases, web servers) running, so several projects can be worked on at once. Use **Stop** or **Stop all** to shut them down. DevKit3 remembers what it started, even after it is closed and reopened.

The progress bar at the bottom shows downloads and installation; **Cancel** stops the current operation.

---

## 10. Setting up another machine

1. Install and start DevKit3.
2. In **Settings**, choose **Git repository** and enter the config repository URL and its token (the same token, or a new one created for this machine).
3. The project list appears.
4. Select a project, set **Project folder on this machine** in **Edit → General** if you do not want the default folder, and click **▶ Open**.
5. DevKit3 asks for the **tokens password** (tick *Remember on this machine*), then installs the tools and clones the repositories with the tokens saved on your other machines.

---

## 11. Where DevKit3 keeps its data

Projects default to `~/DevKit3/Projects`. App data is under `~/DevKit3/Data` (`~` means your user home directory on Windows and Linux):

| Path | Contents |
|---|---|
| `config\` | Clone of the config repository: `devkit3-projects.json` and `devkit3-tokens.json` |
| `local.json` | Config repository URL, project folders of this machine, running processes |
| `secrets.json` | The config repository token and the remembered tokens password, encrypted with DPAPI for your Windows account |
| `apps\<tool>\<version>\` | Installed tools |
| `instances\project-<project name>-<project id>\<tool>\` | Per-project data: MariaDB database, Apache logs, VSCode settings, `custom.ini` / `custom.conf` |
| `downloads\` | Download cache; safe to delete |

Project instances use the current project name and stable id directly. DevKit3 does not scan or migrate older folders. If you rename a project, its instance path changes with the name. Standalone tools keep their `instances\launch-<tool>\` folders.

Set the `DEVKIT3_HOME` environment variable to use another folder (for example a portable installation on another drive).

`secrets.json` cannot be decrypted on another machine or by another Windows account. Copying it is useless: on a new machine enter the config repository token and the tokens password instead.

---

## 12. Troubleshooting

**`Authentication failed` / HTTP 401 or 403 when cloning, pulling or pushing**
- The token expired or was revoked: enter a new one ([section 7.4](#74-when-a-token-expires)).
- The token has no access to that repository (GitHub fine-grained token without that repository, or read-only token used to push).
- Wrong **User** for the service ([section 3](#what-to-put-in-the-user-field)).
- Git was run outside DevKit3 ([section 8](#8-how-git-authentication-works)).

**`... exists, is not empty and is not a git repository`**
The repository folder already contains files. Move them away or choose another folder.

**`port 3306 is already in use`**
Another program (or another project) listens on that port. Pick another `port=` in the tool settings.

**`... returned a web page instead of the file`**
Check that the tool's release tag and `<ToolId>-<version>.zip` asset exist in `minhnguyenerp/devkit3`. Click **Sync** to refresh the version list, then retry. Package layout is described in [Tool packages](tool-packages.md).

**`Missing values: ...`**
Enter the values when asked, or in **Edit → Variables / Repositories**.

**"Wrong password" when unlocking tokens**
Use the password chosen when the tokens file was created, on any machine. It cannot be recovered ([section 6.1](#61-the-tokens-file-and-its-password)).

**"Values to enter again" after a sync**
Another machine had created its own tokens file with a different password. The one in the repository is kept: unlock it with its password and enter the listed values again.

**"Changed on two machines" window**
The same project or group was edited here and on another machine. Choose which version to keep ([section 4.1](#41-working-on-several-machines)).

**`Sync cancelled: ... changed on this machine and on another one`**
You closed the "Changed on two machines" window with **Cancel**. Nothing was lost: click **Sync** to choose again.

**A change made here does not appear on another machine**
The push failed (no network or expired token); the error was shown when saving. Fix the cause (for the token, see [section 7.4](#74-when-a-token-expires)) and click **Sync** on this machine, then on the other one.

**After reinstalling Windows DevKit3 asks for the config token and the tokens password again**
The copies saved for your Windows account (DPAPI) cannot be read after a reinstall. Enter the config repository token in **Settings** and the tokens password when asked; all other values come back from the tokens file.

---

## 13. Git cheatsheet

Run these commands in the repository folder. With private repositories, open **Terminal** from the project in DevKit3 so Git receives its token configuration. The commands also work in an editor terminal when the editor was started by DevKit3. Replace example branch names, paths and URLs with your own.

### 13.1 Check what changed

| Command | Purpose |
|---|---|
| `git status` | Current branch, modified files, staged files and conflicts. Start here. |
| `git status -sb` | Compact branch and file status. |
| `git diff` | Changes not yet staged. |
| `git diff --staged` | Staged changes that will enter the next commit. |
| `git diff -- src/app.cs` | Changes in one file. |
| `git log --oneline -15` | Last 15 commits available locally. |
| `git log --oneline --graph --decorate --all -30` | Recent history with branches and tags. |
| `git show HEAD` | Most recent commit and its changes. |
| `git blame -- src/app.cs` | Which commit last changed each line of a file. |

Press **q** to leave Git's pager when a log or diff fills the terminal.

### 13.2 Clone a repository

```bash
# Minimal history, like DevKit3's project clones
git clone --depth 1 https://server.example/team/repo.git

# Select a branch when cloning
git clone --depth 1 --branch main https://server.example/team/repo.git

# Clone the repository and its submodules with minimal history
git clone --depth 1 --recurse-submodules --shallow-submodules https://server.example/team/repo.git
```

Choose one clone command. A normal `git clone URL` downloads the full history for the branches it fetches.

### 13.3 Everyday edit, commit and push

```bash
git status
git diff
git add src/app.cs
git diff --staged
git commit -m "Describe the change"
git push
```

| Command | Purpose |
|---|---|
| `git add --all` | Stage all additions, changes and deletions in this repository. Review `git status` first. |
| `git add -p` | Choose individual changes to stage interactively. |
| `git restore --staged -- src/app.cs` | Unstage a file; keep its working changes. |
| `git push -u origin feature/my-change` | First push of a new branch; set its upstream for later `git push` and `git pull`. |

### 13.4 Branches and updating code

| Command | Purpose |
|---|---|
| `git branch` | List local branches; `*` marks the current one. |
| `git branch -a` | List local and known remote branches. |
| `git switch main` | Switch to an existing local branch. |
| `git switch -c feature/my-change` | Create and switch to a new branch from the current commit. |
| `git fetch origin` | Download remote updates without changing checked-out files. |
| `git pull --ff-only` | Fetch and update the current branch only if no merge is needed. If histories diverged, stop and inspect them. |
| `git merge origin/main` | Merge the fetched `main` branch into the current branch. May require conflict resolution. |
| `git branch -d feature/my-change` | Delete a merged local branch after switching away from it. |

For a fresh feature branch, switch to `main`, run `git pull --ff-only`, then `git switch -c feature/my-change`.

### 13.5 Save unfinished work temporarily

```bash
git stash push -u -m "Work in progress"
git stash list
git stash pop
```

Run the first command before switching tasks, and `git stash pop` when ready to restore the work. `-u` includes untracked files, but not ignored files. Restoring a stash can produce conflicts. `git stash apply` restores without removing the stash entry.

### 13.6 Resolve a merge conflict

1. Run `git status` to find conflicting files.
2. Edit those files, choose the intended content and remove conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`).
3. Stage resolved files with `git add path/to/file`.
4. For a merge, run `git merge --continue`. To cancel that merge instead, use `git merge --abort`.

If the operation was a rebase, use `git rebase --continue` or `git rebase --abort` instead. Check `git status` before deciding which command applies.

### 13.7 Submodules: the two different kinds of update

| Command | Purpose |
|---|---|
| `git submodule status --recursive` | Show submodule commits, including nested submodules. |
| `git submodule update --init --recursive --depth 1` | Initialize missing submodules and check out the exact commits recorded by their parents, with minimal history. |
| `git submodule update --remote --recursive --depth 1` | Fetch and check out the configured remote branch tips of submodules, with minimal history. This can change the commits recorded by the parent working tree. |
| `git submodule sync --recursive` | Update local submodule URL settings after URLs change in `.gitmodules`. Does not fetch code. |
| `git submodule foreach --recursive 'git status -sb'` | Inspect the working state of every initialized submodule. |

After pulling a parent repository, use the update command **without `--remote`** to reproduce the versions that parent expects. Use **`--remote`** when you intentionally want newer submodule code. A submodule update commonly leaves its checkout at a detached HEAD; switch to a branch inside that submodule before developing new changes.

After intentionally updating a submodule, record its new commit in the parent repository:

```bash
git status
git add Applications/Global
git commit -m "Update Global submodule"
git push
```

If you edited files inside a submodule, commit and push those changes **inside the submodule first**, then commit its new pointer in the parent. For nested submodules, work from the deepest repository outward. A parent commit does not include uncommitted files inside a submodule.

### 13.8 Shallow clones: get more history or another branch

| Command | Purpose |
|---|---|
| `git rev-parse --is-shallow-repository` | `true` means local history is incomplete. |
| `git fetch --deepen=100 origin` | Download up to 100 more levels of history. |
| `git fetch --unshallow origin` | Convert a shallow clone to full history available from that remote. Use only when the repository is shallow. |
| `git fetch origin tag v1.2.3` | Fetch a particular tag. |
| `git fetch origin feature/my-change:refs/remotes/origin/feature/my-change` | Fetch a branch omitted by the original single-branch clone. |
| `git switch -c feature/my-change --track origin/feature/my-change` | Create a local branch tracking the fetched remote branch. |

Run history commands inside the repository that needs them; a parent repository and each submodule have separate history. No reclone is needed.

### 13.9 Undo a mistake

| Command | Purpose |
|---|---|
| `git restore --staged -- src/app.cs` | Undo staging while keeping file edits. |
| `git restore -- src/app.cs` | **Discard unstaged edits** in that tracked file, restoring its staged version. Those discarded edits are not saved by this command. |
| `git revert HEAD` | Create a new commit reversing the latest commit. Useful for changes already shared; may require conflict resolution. |
| `git reflog -15` | Inspect recent local HEAD movements to locate a previous commit. |

To inspect/recover a commit found in the reflog, create a branch at its hash: `git switch -c recovery/my-work abc1234`. This recovers committed work, not arbitrary unsaved file edits.

### 13.10 Remotes and identity

| Command | Purpose |
|---|---|
| `git remote -v` | Show fetch/push URLs. |
| `git remote get-url origin` | Show the main remote URL. |
| `git config user.name "Your Name"` | Set the commit author name for this repository. |
| `git config user.email "you@example.com"` | Set the commit author email for this repository. |
| `git --version` | Show the Git version used by this terminal. |

Identity settings control commit attribution, not server authentication. Use DevKit3's **Auth variable** and token settings for authentication. For more commands, see the [official Git reference](https://git-scm.com/docs).
