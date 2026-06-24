# Dev Container

This devcontainer uses Docker to create a reproducible Ubuntu-based development environment for Salesforce development. It is designed primarily for Apple Silicon (M-series) Macs but uses Ubuntu Linux for broad compatibility.

## Dockerfile

Built from `ubuntu:24.04`, the Dockerfile sets up the full toolchain in layers:

1. **System packages** — installs core utilities (`curl`, `git`, `jq`, `wget`, `unzip`, `xz-utils`, `gnupg`, `lsb-release`, `sudo`, `zsh`) and **OpenJDK 17** (headless) required by Salesforce CLI for Apex compilation and language server features.
2. **Locale** — configures `en_US.UTF-8` to prevent encoding issues in CLI output.
3. **Node.js 20** — installed via the NodeSource APT repository, providing the runtime for the Salesforce CLI and project npm tooling.
4. **GitHub CLI (`gh`)** — installed from the official GitHub APT repository for PR and repo management from the terminal.
5. **Salesforce CLI (`sf`)** — installed globally via `npm install -g @salesforce/cli` as root so it is available system-wide.
6. **Non-root `vscode` user** — Ubuntu 24.04 ships a built-in `ubuntu` user at UID/GID 1000, which would conflict with the standard dev-container convention. The Dockerfile removes that user first (`userdel -r ubuntu`), then creates a `vscode` user at UID/GID 1000 with passwordless `sudo` access and `zsh` as the default shell.
7. **Oh My Zsh** — installed for the `vscode` user to provide a productive shell experience.
8. **Salesforce Code Analyzer plugin** — `@salesforce/plugin-code-analyzer` is installed as the `vscode` user so it lands in the user-scoped plugin directory (`~/.local/share/sf/plugins`).

The working directory is set to `/workspaces/qldo-jax-rv` to match the bind-mount path used by the dev container.

## devcontainer.json

Configures how VS Code / Cursor attaches to and uses the container:

| Setting                              | Value                     | Purpose                                                                                                                                                             |
| ------------------------------------ | ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `build.dockerfile`                   | `Dockerfile`              | Points to the Dockerfile above; `context` is the repo root so project files are available during the build.                                                         |
| `workspaceMount` / `workspaceFolder` | `/workspaces/qldo-jax-rv` | Bind-mounts the local repo into the container at a stable path.                                                                                                     |
| `forwardPorts`                       | `[1717]`                  | Exposes the port that `sf` uses for local org OAuth callback during `sf org login web`.                                                                             |
| `postCreateCommand`                  | `npm install && ...`      | Installs npm dependencies and prints versions of Node, `sf`, its core plugins, Java, and `gh` to confirm the environment is healthy after the container is created. |
| `remoteUser`                         | `vscode`                  | Runs VS Code Server and all terminal sessions as the non-root `vscode` user.                                                                                        |
| **Extensions**                       | See list below            | Pre-installs a curated set of extensions inside the container.                                                                                                      |
| **Settings**                         | See list below            | Applies workspace-level editor settings inside the container.                                                                                                       |

### Pre-installed Extensions

- `salesforce.salesforcedx-vscode` — Salesforce core pack (Apex, SOQL, LWC, Aura, etc.)
- `salesforce.sfdx-code-analyzer-vscode` — Code Analyzer results viewer
- `GitHub.vscode-pull-request-github` — GitHub PRs and issues
- `eamodio.gitlens` — Advanced Git history and blame
- `esbenp.prettier-vscode` — Code formatting
- `dbaeumer.vscode-eslint` — JavaScript/LWC linting
- `streetsidesoftware.code-spell-checker` — Spell checking

### Applied Editor Settings

- Default terminal profile set to `zsh`
- Apex Java home pointed at `/usr/lib/jvm/java-17-openjdk-aarch64` (ARM64)
- Format on save enabled with Prettier as the default formatter for all files and Apex
