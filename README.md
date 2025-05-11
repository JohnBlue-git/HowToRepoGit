## What is `repo` 
The `repo` tool is a Google-built wrapper for Git, used heavily in Android OS development.

### Install Git (this is pre-requirement)
```bash
sudo apt update
sudo apt install git
```

### Configure Git
Run these commands to set your Git identity (necessary for commits, repo sync, etc.):
```bash
git config --global user.name "Your Full Name"
git config --global user.email "your.email@example.com"
```
Confirm your settings:
```bash
git config --list
```

### Install `repo` on Linux/macOS:
```bash
# Create a directory for the repo binary
mkdir -p ~/bin
export PATH=~/bin:$PATH

# Download the repo script
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo

# Make it executable
chmod a+x ~/bin/repo
```
> **Tip:** Add `export PATH=~/bin:$PATH` to your `~/.bashrc`, `~/.zshrc`, or shell config file to persist it.

### Example: Initializing a repo-based project (Android)
```bash
# Create a working directory
mkdir android-project && cd android-project

# Initialize a repo-based manifest
repo init -u https://android.googlesource.com/platform/manifest

# Sync the source (downloads a LOT of data)
repo sync
```

## Common usage of repo

### 🔹 `repo init`
Initializes a repo client in your working directory.
```bash
repo init -u <manifest-url> [-b <branch>] [-m <manifest.xml>]
        [--depth=1] [--no-clone-bundle] [--no-tags] [--repo-url=<repo-tool-url>] [--mirror]
```
* `-u <manifest-url>`, `--manifest-url <manifest-url>`
  **(Required)** URL of the manifest Git repository (e.g., AOSP: `https://android.googlesource.com/platform/manifest`).
* `-b <branch>`, `--manifest-branch <branch>`
  Manifest branch to use (e.g., `android-14.0.0_r1`).
* `-m <manifest.xml>`, `--manifest-name <manifest.xml>`
  Use a specific manifest file other than the default `default.xml`.
* `--depth=1`
  Perform a shallow clone to reduce initial download size.
* `--no-clone-bundle`
  Skip the optimized Git bundle that might cause issues in some environments.
* `--no-tags`
  Don’t fetch Git tags—helps reduce sync time and bandwidth usage.
* `--repo-url=<repo-tool-url>`
  Specify a custom location to download the `repo` tool (e.g., internal mirror or alternative server).
* `--mirror`
  Create a mirror of all repositories (used to set up a local source cache).

### 🔹 `repo sync`
Downloads and syncs all the repositories specified in the manifest.
```bash
repo sync [-jN] [--fail-fast] [--force-sync] [--fetch-submodules] [--no-clone-bundle]
          [-c] [-d] [-f]
```
* `-j4` – Use 4 parallel threads for syncing (default is 4; more threads = faster sync)
* `--fail-fast` – Stop immediately if any project fails to sync
* `--force-sync` – Discard local changes and re-sync all repositories
* `--fetch-submodules` – Also fetch Git submodules defined in the manifests
* `--no-clone-bundle` – Avoid using pre-packaged Git bundle (may improve reliability)
* `-c`, `--current-branch` – Only fetch the current manifest branch for each project (faster and lighter)
* `-d`, `--detach` – Sync and checkout the detached HEAD state instead of tracking the manifest branch
* `-f`, `--force-broken` – Force syncing even if a previous sync failed (use with caution)

It is similar to `repo forall -c 'git pull --rebase'`, but `repo sync` ensures repo structure matches the manifest exactly.

| Use Case                                                                  | Recommended Command                  |
|---------------------------------------------------------------------------|--------------------------------------|
| Clean, accurate sync per manifest                                         | `repo sync`                          |
| You’ve already synced and want to update local branches without full sync | `repo forall -c 'git pull --rebase'` |
| Recovering from a broken/inconsistent state                               | `repo sync -f`                       |


### 🔹 `repo status`
Shows Git status of all projects (good for checking local changes):
```bash
repo status
```

### 🔹 `repo start`
Start a new branch for development on all/specified repos:
```bash
repo start <branch-name> [specific project defined in manifest ...]
```
When command `repo start <local branch-name>`, it is the same as `repo forall -c "git checkout -b <local branch-name>"`

### 🔹 `repo forall`
Run a shell command across all projects:
```bash
repo forall -c '<command>'

# To checkout
repo forall -c 'git checkout main'
# (Not official)
repo checkout main
repo checkout <feature branch>

# To checkout remote branch locally !!!
repo forall -c 'git fetch && git checkout <remote>/<remote branch name> -b <local branch name>'
#               git fetch
# Downloads the latest refs (branches, tags) from the remote (usually `origin`) without modifying your working directory.
# Does not merge anything into your current branch
# Does not switch your branch or modify files
#                            git checkout <remote>/<remote branch name> -b <local branch name>
# * Creates a **new local branch** named `android-14.0.0_r1`
# * Starts it **from the fetched remote branch**

# To delete
repo forall -c 'git branch -D <local branch name>'
```

### 🔹 `repo branches`
Lists all active local branches across projects.
```bash
repo branches
```

### 🔹 `repo diff`
Shows combined diff across all projects.
```bash
repo diff
```

### 🔹 `repo manifest`
To show or generate the `manifest.xml` in your workspace.
```bash
repo manifest [--xml-name=<filename>] [--git-url=<url>] [--no-tags]
        [-v] [-r] [-o <directory>]
```
* `--xml-name=<filename>` – Generate the manifest and output it to a specific file name (default: `default.xml`).
* `--git-url=<url>` – Customize the base Git URL for repositories.
* `-v` – Show the **version** of the manifest file currently in use.
* `-r` – Show the **repo manifest revision** (the Git commit ID). 
* `-o <directory>` – Output the generated manifest to a specific directory.

### 🔹 `repo list`
The `repo list` command is for inspecting the set of repositories that are part of the current manifest, along with their current statuses, branches, and more.
```bash
repo list [--short] [--long] [--status] [--untracked]
    [-v] [-p]
    [<sepecific project> ...]
```
* `--short` – Display a **shorter output**, showing only the project names.
* `--long` – Customize the base Git URL for repositories.
* `--status` – Show the **current status** (similar to `git status`) for each project.
* `--untracked` – Show **untracked projects** that are not part of the manifest.
* `-v` – Show the **current branch** for each project.
* `-p` – Display only the **project path** (without additional details like branch).

## My example manifest

### Here's an example of a minimal `repo` manifest
default.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <remote name="github"
          fetch="https://github.com/JohnBlue-git/" />

  <default remote="github"
           revision="main"
           sync-j="4" />

  <project name="HowToBoostAsio"
           path="boost-asio">
    <copyfile src="README.md" dest="README-boost-asio.md" />
    </project>

  <project name="BoostAsioWebApi-VersionOne"
           path="web-api">
    <copyfile src="README.md" dest="README-web-api.md" />
    </project>

</manifest>
```
| Tag / Attribute    | Description                                                              |
| ------------------ | ------------------------------------------------------------------------ |
| `<remote>`         | Defines a Git host (`fetch` URL prefix) — here it's your GitHub profile. |
| `<default>`        | Sets defaults for revision (`main`) and parallel sync.                   |
| `<project>` `name` | GitHub repo name (the path after `github.com/JohnBlue-git/`).            |
| `<project>` `path` | Local folder name for the repo on your disk (you can change it).         |

### How to use
Create repo to place `dafault.xml`
```console
git init
git branch -M main (optional)
git add default.xml
git commit -m "Add manifest"
```
Clone from elsewhere
```bash
# Init repo
repo init -u https://github.com/JohnBlue-git/HowToRepoGit.git -b main -m default.xml
# Or we can test locally:
repo init -u . -b main -m default.xml

# Run repo sync
repo sync
```
Then we will see
```console
├── boost-asio/
│   ...
│   └── README.md  # original file in repo
├── web-api/
│   ...
│   └── README.md  # original file in repo
├── README-boost-asio.md     # copied here by <copyfile>
├── README-web-api.md        # copied here by <copyfile>
└── .repo/              # internal repo metadata
```
