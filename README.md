# Enhanced Zsh Setup — Complete Feature Guide

> **Quick start:** Run `./zsh-setup.sh install`, install a [Nerd Font](https://www.nerdfonts.com) in your terminal, then run `p10k configure`.

---

## Table of Contents

1. [Theme — Powerlevel10k](#1-theme--powerlevel10k)
2. [Tools](#2-tools)
   - [zoxide — Smart cd](#21-zoxide--smart-cd)
   - [bat — Better cat](#22-bat--better-cat)
   - [eza — Better ls](#23-eza--better-ls)
   - [delta — Better git diff](#24-delta--better-git-diff)
   - [thefuck — Autocorrect](#25-thefuck--autocorrect)
   - [tldr — Quick man pages](#26-tldr--quick-man-pages)
   - [fzf — Fuzzy Finder](#27-fzf--fuzzy-finder)
3. [Key Bindings](#3-key-bindings)
4. [Plugins](#4-plugins)
5. [Aliases](#5-aliases)
   - [Navigation](#51-navigation)
   - [File Listing (eza)](#52-file-listing-eza)
   - [Git](#53-git)
   - [Network](#54-network)
   - [Disk & Process](#55-disk--process)
   - [Config & Misc](#56-config--misc)
6. [Functions](#6-functions)
7. [History & Shell Options](#7-history--shell-options)
8. [Completion System](#8-completion-system)
9. [Customization](#9-customization)
10. [Troubleshooting](#10-troubleshooting)

---

## 1. Theme — Powerlevel10k

Powerlevel10k (p10k) is a feature-rich Zsh prompt with instant rendering, git awareness, and context-aware segments.

### What it shows

| Segment | Description |
|---|---|
| OS icon | Distro symbol (requires Nerd Font) |
| Directory | Current path, shortened smartly |
| Git branch & status | Branch name, ✔ clean / ✗ modified / ? untracked |
| Exit code | Red indicator when the last command failed |
| Execution time | How long the last command took (shown if > 2s) |
| Background jobs | Count of suspended jobs |
| Node / Python version | Shown when in a relevant project dir |
| Time | Current time (HH:MM) |

### Setup

```bash
# Run the interactive configuration wizard
p10k configure

# Reload theme after editing config
exec zsh

# Edit config manually
p10kconfig        # alias for: nano ~/.p10k.zsh
```

### Instant prompt

Powerlevel10k renders the prompt before Zsh finishes loading plugins, making the shell feel instant. If you see a warning about "instant prompt", move any code that prints output to *after* the p10k block in `~/.zshrc_custom`.

### Nerd Font (required)

Without a Nerd Font, icons render as boxes or question marks.

1. Download from [nerdfonts.com](https://www.nerdfonts.com) — recommended: **MesloLGS NF** or **JetBrainsMono Nerd Font**
2. Install the font on your system
3. Set it in your terminal emulator's font settings
4. Run `p10k configure` to verify icons render correctly

---

## 2. Tools

### 2.1 zoxide — Smart cd

zoxide tracks the directories you visit and lets you jump to any of them by typing a short fragment. It replaces `cd` via the `z` alias.

**How it learns:** Every `cd`/`z` command updates an internal frequency + recency score (frecency). The more you visit a directory, the more likely it is to match short queries.

```bash
# Basic usage
z proj              # jump to the best-matching dir containing "proj"
z dow               # jump to ~/Downloads
z code api          # match a path containing both "code" and "api"

# Interactive picker (fzf required)
zi                  # fuzzy-search all known dirs and pick one

# Force exact path (when zoxide picks wrong)
builtin cd ~/exact/path

# Show zoxide's database
zoxide query --list --score

# Remove a stale entry
zoxide remove /old/path
```

**Tips:**
- Give it a day of normal use — the more dirs you visit, the smarter it gets
- `zi` is the fastest way to jump when you can't remember the exact name
- `cd` is aliased to `z`, so your existing muscle memory works unchanged

---

### 2.2 bat — Better cat

`bat` replaces `cat` and `less` with syntax highlighting, line numbers, and git change markers.

```bash
# View a file (syntax highlighted, with line numbers)
cat file.py
cat config.yaml

# Paged viewing (like less, but colored)
less long_file.log

# Explicit bat usage with options
bat --style=plain file.py          # no line numbers or decorations
bat --language=json data.txt       # force a language
bat --line-range 10:30 file.py     # show only lines 10–30
bat -A file.txt                    # show non-printable characters

# Colorized man pages (configured automatically)
man curl
man git-commit
```

**Theme:** Set to `Monokai Extended`. Change it:
```bash
# List available themes
bat --list-themes

# Set permanently in ~/.zshrc_custom
export BAT_THEME="Dracula"
```

**Note:** On Ubuntu/Debian, bat is installed as `batcat`. The alias `bat='batcat'` is set automatically.

---

### 2.3 eza — Better ls

`eza` replaces `ls` with icons, colors, git status indicators, and tree views.

```bash
ls          # list with icons, dirs first
ll          # long list: permissions, size, date, git status
la          # long list including hidden files
lt          # tree view, 2 levels deep
lta         # tree view, 3 levels, all files including hidden
l           # compact 1-column list

# Direct eza options
eza --level=4                      # tree with 4 levels
eza --sort=size                    # sort by file size
eza --sort=modified                # sort by modified date
eza --no-icons                     # disable icons
eza --header                       # show column headers in long view
```

**Git status column** (shown in `ll` / `la`):

| Symbol | Meaning |
|---|---|
| `M` | Modified |
| `A` | Added (staged) |
| `D` | Deleted |
| `R` | Renamed |
| `?` | Untracked |
| `-` | Ignored |

---

### 2.4 delta — Better git diff

`delta` is a syntax-highlighting pager for `git diff`, `git log -p`, `git show`, and `git blame`. It is configured automatically in your global git config.

```bash
# delta is active automatically for all git commands
git diff
git diff HEAD~3
git log -p
git show abc1234
git stash show -p

# Side-by-side view (toggle in ~/.gitconfig)
git config --global delta.side-by-side true

# Navigate between changed files in diff output
# n → next file,  N → previous file  (when delta.navigate=true)
```

**Configuration** (auto-applied to `~/.gitconfig`):

| Setting | Value | Effect |
|---|---|---|
| `navigate` | `true` | `n`/`N` to jump between files |
| `line-numbers` | `true` | Show line numbers in both old and new |
| `light` | `false` | Dark-mode color theme |
| `syntax-theme` | `Monokai Extended` | Syntax highlight theme |

---

### 2.5 thefuck — Autocorrect

`thefuck` watches your last command, detects what went wrong, and suggests a corrected version. Type `f` (or `fuck`) to accept it.

```bash
# Examples of what it fixes
git comit -m "fix"
f                           # → git commit -m "fix"

apt install vim
f                           # → sudo apt install vim

git push
f                           # → git push --set-upstream origin <branch>

cd ~/Documnets
f                           # → cd ~/Documents

pyhton script.py
f                           # → python script.py

# Repeat correction (if first suggestion isn't right)
f                           # first suggestion
f                           # next suggestion
```

**Aliases:**
```bash
f       # short alias for 'fuck'
fuck    # full alias (also works)
```

---

### 2.6 tldr — Quick man pages

`tldr` shows community-written, practical examples for any command. Much faster than `man` when you just need a quick reminder of syntax.

```bash
tldr tar
tldr curl
tldr git checkout
tldr rsync
tldr docker
tldr ffmpeg
tldr find

# Update the local cache
tldr --update
```

**Example output for `tldr tar`:**
```
tar
Archiving utility.
- Create an archive and write it to a file:
    tar cf target.tar file1 file2 file3
- Extract a (compressed) archive into the current directory:
    tar xf source.tar[.gz|.bz2|.xz]
```

---

### 2.7 fzf — Fuzzy Finder

`fzf` is an interactive filter for any list. It powers the key bindings (Ctrl+R, Ctrl+F, Alt+C), tab completion previews, and the `fcd`/`fkill`/`fgit` functions. It can also be used directly.

```bash
# Pipe any list into fzf
ls | fzf
cat ~/.zsh_history | fzf
ps aux | fzf

# With preview
fzf --preview 'bat --color=always {}'
fzf --preview 'cat {}'

# Multi-select (Tab to mark, Enter to confirm)
ls | fzf -m

# Pass selected file to a command
vim $(fzf)
code $(fzf --preview 'bat --color=always {}')
```

**Built-in key bindings (configured automatically):**

| Key | Action |
|---|---|
| `Ctrl+R` | Fuzzy search shell history |
| `Ctrl+F` | Fuzzy file picker (inserts path at cursor) |
| `Alt+C` | Fuzzy cd into a directory |

**fzf inside completion:** After typing a partial command, press `Tab` to get an fzf-powered completion menu with preview.

```bash
cd **<Tab>            # fuzzy directory picker
vim **<Tab>           # fuzzy file picker
kill **<Tab>          # fuzzy process picker
ssh **<Tab>           # fuzzy host picker
```

---

## 3. Key Bindings

| Key | Action | Notes |
|---|---|---|
| `Ctrl+R` | Fuzzy history search | Powered by fzf; shows preview of full command |
| `Ctrl+F` | Fuzzy file picker | Inserts selected path at cursor position |
| `Alt+C` | Fuzzy cd | Directory tree preview via eza |
| `↑` | History search up | Filters by what you've already typed |
| `↓` | History search down | Same filter, opposite direction |
| `Ctrl+→` | Jump forward one word | Skip across arguments |
| `Ctrl+←` | Jump backward one word | Skip across arguments |
| `Ctrl+Backspace` | Delete word to the left | Much faster than holding Backspace |
| `Home` | Go to start of line | Standard editor behavior |
| `End` | Go to end of line | Standard editor behavior |
| `Delete` | Delete character to the right | Forward delete |

---

## 4. Plugins

| Plugin | What it does |
|---|---|
| `zsh-autosuggestions` | Ghost-text suggestions from history as you type; press `→` to accept |
| `zsh-syntax-highlighting` | Colors your command as you type — green = valid, red = not found |
| `zsh-history-substring-search` | Powers the ↑/↓ filtered history search |
| `zsh-autopair` | Auto-closes `(`, `[`, `{`, `"`, `'` and handles backspace deletion |
| `zsh-completions` | Adds hundreds of extra completion definitions |
| `fzf-tab` | Replaces the default completion menu with an fzf picker + preview |
| `forgit` | fzf-powered interactive git commands (see below) |
| `zsh-you-should-use` | Reminds you when a shorter alias exists for what you just typed |
| `zsh-better-npm-completion` | Smart tab-completion for npm scripts and package names |
| `zsh-nvm` | Lazy-loads nvm so it doesn't slow down shell startup |
| `git` | Oh My Zsh built-in: `ga`, `gco`, `gcm`, `gp`, `gl`, and ~150 more |

### forgit — interactive git

`forgit` provides fzf-powered versions of common git commands. Every command opens an interactive picker.

```bash
glo     # git log — browse commits, press Enter to show full diff
gad     # git add — interactively stage files (with diff preview)
gcf     # git checkout file — restore files from HEAD interactively
gsp     # git stash pop — pick a stash to pop
gbd     # git branch delete — pick branches to delete
gcb     # git checkout branch — fzf branch switcher
gdf     # git diff — pick files to diff interactively
grh     # git reset HEAD — unstage files interactively
```

### zsh-you-should-use

```bash
# If you type a full command that has an alias, you'll see:
$ git status
Found existing alias for "git status". You should use: "gst"
```

---

## 5. Aliases

### 5.1 Navigation

```bash
..          # cd ..
...         # cd ../..
....        # cd ../../..
~           # cd ~
-           # cd - (go back to previous directory)
```

### 5.2 File Listing (eza)

```bash
ls          # eza --icons --group-directories-first
ll          # long list + git status
la          # long list + hidden files + git status
lt          # tree, 2 levels
lta         # tree, 3 levels, all files
l           # compact 1-column list
```

### 5.3 Git

```bash
glog        # pretty graph log with hash, branch, author, relative time
gst         # git status -sb  (short format + branch)
gdiff       # git diff --stat  (show changed files + line counts only)
gwip        # git add -A && git commit -m "WIP: work in progress"
gundo       # git reset HEAD~1 --soft  (undo last commit, keep changes staged)
gclean      # delete all local branches merged into main/master/develop
```

**Oh My Zsh `git` plugin extras** (selection):

```bash
ga          # git add
gaa         # git add --all
gco         # git checkout
gcb         # git checkout -b  (new branch)
gcm         # git checkout main
gd          # git diff
gp          # git push
gl          # git pull
gm          # git merge
grb         # git rebase
gsta        # git stash
gstp        # git stash pop
```

### 5.4 Network

```bash
myip        # show public IP (via ifconfig.me)
localip     # show LAN/private IP
ports       # show listening ports (ss -tulpn)
ping        # ping -c 5  (always stops after 5 packets)
```

### 5.5 Disk & Process

```bash
df          # df -hT  (human-readable + filesystem type)
du          # du -sh  (summary of current dir)
dud         # du -d 1 -h  (size of each immediate subdirectory)
psg vim     # search running processes by name
```

### 5.6 Config & Misc

```bash
zshrc       # open ~/.zshrc in $EDITOR, then auto-reload
zshcustom   # open ~/.zshrc_custom in $EDITOR, then auto-reload
p10kconfig  # open ~/.p10k.zsh in $EDITOR
reload      # source ~/.zshrc  (reload without opening editor)

path        # print each $PATH entry on its own line
week        # print current ISO week number
timestamp   # print current datetime as 20260314_153000
pubkey      # print your SSH public key (id_ed25519 or id_rsa)
grep        # grep --color=auto

# Safety aliases (prompt before overwriting/deleting)
rm          # rm -i
cp          # cp -i
mv          # mv -i
mkdir       # mkdir -pv  (create parents, verbose)
```

---

## 6. Functions

### `mkcd` — Make directory and cd into it

```bash
mkcd ~/projects/new-thing
mkcd /tmp/test/nested/dir       # creates all parents automatically
```

---

### `extract` — Universal archive extractor

Detects the archive format and uses the correct extraction tool automatically.

```bash
extract archive.tar.gz
extract backup.tar.bz2
extract data.zip
extract package.7z
extract file.rar
extract compressed.xz
extract old.tar.Z
```

Supported formats: `.tar.gz`, `.tar.bz2`, `.tar.xz`, `.tar.zst`, `.tar`, `.gz`, `.bz2`, `.zip`, `.7z`, `.rar`, `.xz`, `.Z`

---

### `fcd` — Fuzzy directory jump

Opens an fzf picker over all directories from a starting path. Uses eza for directory preview.

```bash
fcd                 # search from current directory
fcd ~/code          # search from ~/code
fcd /               # search from filesystem root (slow on large systems)
```

---

### `fkill` — Fuzzy kill a process

Lists all running processes in fzf. Select one to kill it.

```bash
fkill               # kill -9 (SIGKILL) selected process
fkill 15            # kill -15 (SIGTERM) selected process
fkill 1             # send SIGHUP
```

---

### `fgit` — Fuzzy git branch switcher

Shows all local and remote branches in fzf with a commit log preview. Select one to check it out.

```bash
fgit                # pick a branch and checkout
```

---

### `up` — Go up N directories

```bash
up              # go up 1 level (same as cd ..)
up 3            # go up 3 levels (same as cd ../../..)
up 5            # go up 5 levels
```

---

### `backup` — Timestamped file backup

Creates a copy of a file with the current datetime appended.

```bash
backup nginx.conf
# → creates nginx.conf.backup.20260314_153000

backup ~/.zshrc
# → creates /home/user/.zshrc.backup.20260314_153001
```

---

### `serve` — Quick HTTP server

Starts Python's built-in HTTP server in the current directory.

```bash
serve               # http://localhost:8000
serve 3000          # http://localhost:3000
serve 9090          # http://localhost:9090
```

Useful for quickly sharing files on a local network or previewing static HTML files.

---

### `calc` — Quick calculator

Uses `bc -l` (arbitrary precision math). Supports decimals, exponents, and basic functions.

```bash
calc 1920 / 1080
calc 2^32
calc 150 * 1.08         # 162 (with VAT)
calc "sqrt(144)"
calc "scale=5; 1/3"     # 0.33333
```

---

### `weather` — Terminal weather

Uses [wttr.in](https://wttr.in) to show a one-line weather summary.

```bash
weather                 # weather at your current IP location
weather "Bangkok"
weather "New York"
weather "London"
```

---

### `sshkey` — Generate SSH key

Generates a new ed25519 SSH key and prints the public key.

```bash
sshkey                  # creates ~/.ssh/id_ed25519
sshkey work_github      # creates ~/.ssh/work_github
sshkey deploy_prod      # creates ~/.ssh/deploy_prod
```

After running, copy the printed public key to GitHub/GitLab/server `~/.ssh/authorized_keys`.

---

### `gi` — Fetch .gitignore template

Fetches a `.gitignore` template from [gitignore.io](https://www.toptal.com/developers/gitignore).

```bash
gi node
gi python
gi node,react,vscode
gi go,linux,jetbrains > .gitignore    # save to file
```

---

## 7. History & Shell Options

### History settings

| Setting | Value | Effect |
|---|---|---|
| `HISTSIZE` | 50,000 | Lines kept in memory |
| `SAVEHIST` | 50,000 | Lines saved to `~/.zsh_history` |
| `HIST_IGNORE_DUPS` | on | Don't save consecutive duplicate commands |
| `HIST_IGNORE_ALL_DUPS` | on | Remove older duplicate entries |
| `HIST_IGNORE_SPACE` | on | Commands starting with a space are not saved |
| `HIST_FIND_NO_DUPS` | on | Skip duplicates when searching |
| `SHARE_HISTORY` | on | History is shared across all open terminal sessions |

**Tip:** Prefix any command with a space to prevent it from being saved to history (useful for passwords or one-off commands you don't want to repeat).

```bash
 export AWS_SECRET=mysecret    # leading space = not saved
```

### Shell options

| Option | Effect |
|---|---|
| `AUTO_CD` | Type a directory name to cd into it without typing `cd` |
| `AUTO_PUSHD` | Every `cd` pushes to the directory stack; use `popd` to go back |
| `CORRECT` | Suggests spelling corrections for mistyped commands |
| `INTERACTIVE_COMMENTS` | Allows `# comments` in the interactive shell |
| `GLOB_DOTS` | Glob patterns like `*` match dotfiles without needing `.*` |

```bash
# AUTO_CD example
~/projects          # same as cd ~/projects

# AUTO_PUSHD example
cd ~/a
cd ~/b
cd ~/c
popd                # back to ~/b
popd                # back to ~/a

# List directory stack
dirs -v
```

---

## 8. Completion System

The completion system is cached (regenerated once per day) for fast startup, and enhanced with fzf-tab for interactive menus.

### Tab completion features

```bash
# Basic completion
git che<Tab>        # → git checkout

# fzf-tab: any completion opens fzf picker with preview
cd <Tab>            # fzf directory picker with eza preview
vim <Tab>           # fzf file picker with bat preview
kill <Tab>          # fzf process picker
ssh <Tab>           # fzf host picker (from ~/.ssh/config)

# Double-star glob completion
vim **<Tab>         # recursive file picker
cd **<Tab>          # recursive directory picker
```

### Completion behavior

- **Case-insensitive:** `cd proj` matches `Projects` and `projects`
- **Partial matching:** `cd pro` matches any directory starting with `pro`
- **Git-aware:** branch and file completions are sorted by relevance, not alphabetically

---

## 9. Customization

All custom configuration lives in `~/.zshrc_custom`. Edit it freely — it will never be overwritten by Oh My Zsh updates or re-running the setup script.

```bash
zshcustom       # open ~/.zshrc_custom in your editor + auto-reload
```

### Add your own aliases

```bash
# In ~/.zshrc_custom, add at the bottom:
alias k='kubectl'
alias tf='terraform'
alias dc='docker compose'
alias gs='git status'       # (already exists as gst, but you can override)
```

### Add your own functions

```bash
# In ~/.zshrc_custom:
gpr() {
    # Open a GitHub pull request for current branch
    local branch=$(git branch --show-current)
    local remote=$(git remote get-url origin | sed 's/git@github.com:/https:\/\/github.com\//' | sed 's/\.git//')
    open "${remote}/compare/${branch}?expand=1"
}

mkenv() {
    # Create and activate a Python virtual environment
    python3 -m venv "${1:-.venv}"
    source "${1:-.venv}/bin/activate"
}
```

### Change default editor

```bash
# In ~/.zshrc_custom:
export EDITOR='vim'         # or 'nano', 'code', 'nvim', etc.
export VISUAL="$EDITOR"
```

### Change p10k theme

Run the interactive wizard at any time:

```bash
p10k configure
```

Or edit `~/.p10k.zsh` directly to add/remove/reorder segments.

### Change bat theme

```bash
# List available themes
bat --list-themes | fzf --preview 'bat --theme={} --color=always ~/.zshrc'

# Set permanently in ~/.zshrc_custom
export BAT_THEME="Dracula"
```

---

## 10. Troubleshooting

### Icons show as boxes or question marks

You need a Nerd Font installed and selected in your terminal:

1. Download from [nerdfonts.com](https://www.nerdfonts.com) (recommended: MesloLGS NF)
2. Install on your system (double-click the `.ttf` file on most systems)
3. Open your terminal's preferences and change the font
4. Run `exec zsh` or open a new terminal

### Zsh is slow to start

1. Check which plugin is slow:
   ```bash
   zsh -i -c exit  # time a full startup
   zprof            # add 'zmodload zsh/zprof' at top of .zshrc to profile
   ```
2. `zsh-nvm` lazy-loads nvm by default — if you see nvm slowness, check it's not also loaded elsewhere
3. Compinit cache: if you see `compinit` warnings, run `rm -f ~/.zcompdump && exec zsh`

### p10k shows warnings about instant prompt

Move any `echo`, `print`, or `printf` statements in your `~/.zshrc_custom` to *below* the p10k block. Instant prompt cannot allow output before the prompt is drawn.

### thefuck / f command not found

`thefuck` requires Python 3 and pip. Install manually:

```bash
pip3 install thefuck --break-system-packages
# then reload
exec zsh
```

### zoxide not working / z command not found

Check that `~/.local/bin` is in your PATH:

```bash
echo $PATH | tr ':' '\n' | grep local
# if missing:
export PATH="$HOME/.local/bin:$PATH"
```

### Removing everything

```bash
./zsh-setup.sh remove
```

This removes `~/.oh-my-zsh`, `~/.zshrc_custom`, `~/.p10k.zsh`, restores your most recent `.zshrc` backup, and sets Bash as your default shell.

### Reinstalling a single plugin

```bash
# Example: reinstall fzf-tab
rm -rf ~/.oh-my-zsh/custom/plugins/fzf-tab
git clone --depth=1 https://github.com/Aloxaf/fzf-tab.git \
    ~/.oh-my-zsh/custom/plugins/fzf-tab
exec zsh
```

---

## Quick Reference Card

```
TOOLS          COMMAND       WHAT IT DOES
─────────────────────────────────────────────────────
zoxide         z proj        jump to best-matching "proj" dir
               zi            interactive fzf directory picker
bat            cat file.py   syntax-highlighted view
               less file     paged, colored
eza            ll            long list + git status
               lt            tree view
delta          git diff      auto-used, syntax highlighted
thefuck        f             fix last mistyped command
tldr           tldr tar      practical examples

KEY BINDINGS   SHORTCUT      ACTION
─────────────────────────────────────────────────────
               Ctrl+R        fuzzy history search
               Ctrl+F        fuzzy file picker
               Alt+C         fuzzy cd
               ↑ / ↓         filtered history (by typed text)
               Ctrl+→/←      jump word forward/backward

FUNCTIONS      USAGE         EXAMPLE
─────────────────────────────────────────────────────
mkcd           mkcd dir      mkdir + cd in one step
extract        extract f     auto-detect and extract archive
fcd            fcd           fuzzy directory jump (fzf)
fkill          fkill         fuzzy process kill
fgit           fgit          fuzzy git branch switch
up             up 3          go up 3 directories
backup         backup f      timestamped file copy
serve          serve 8080    quick HTTP server
calc           calc 2^10     terminal calculator
weather        weather NYC   one-line weather
sshkey         sshkey name   generate ed25519 key
gi             gi node       fetch .gitignore template

GIT ALIASES    ALIAS         EQUIVALENT
─────────────────────────────────────────────────────
               glog          pretty graph log
               gst           git status -sb
               gdiff         git diff --stat
               gwip          add -A + commit "WIP"
               gundo         reset HEAD~1 --soft
               gclean        delete merged branches
```

---

*Edit `~/.zshrc_custom` to personalize. Run `p10k configure` to set up your prompt. All tools are optional — if one isn't installed, its alias is silently skipped.*

Great question! Here's how it works:
You can install commands from any shell (bash or zsh) — the shell doesn't matter for installation. What matters is registering the completion afterward.
After installing a new tool, just run:
zshregister_completion <toolname>
For example:
zshregister_completion kubectl
register_completion helm
register_completion cargo
This auto-detects the correct completion method and registers it immediately.
If you want to refresh all tools at once:
zshcompinit_refresh

The typical workflow is:

Install the tool (e.g. sudo apt install kubectl or brew install helm)
Run register_completion kubectl in zsh
That's it — Tab completion works immediately in the same session


A few notes:

Tools installed via apt, npm, pip, etc. work fine from any shell
The zsh-nvm plugin handles Node version management automatically inside zsh
If you install something and Tab completion seems broken, run cleanup_completions to remove any stale/invalid completion files
The apti alias is a convenience wrapper — apti gh eza bat installs with flag completion support