# VS Code Shell Command (`code .`)

How to open any directory in VS Code directly from your terminal.

---

## Problem

Running `code .` in the terminal throws `zsh: command not found: code`. VS Code is installed, but the shell command hasn't been registered yet. This happens on a fresh machine setup or after reinstalling VS Code.

## Fix

Open VS Code and run the shell command installer:

1. Open the Command Palette with `⌘⇧P`
2. Type `Shell Command`
3. Select **Shell Command: Install 'code' command in PATH**

VS Code installs a symlink at `/usr/local/bin/code`. Open a new terminal window and `code .` will work.

---

## Usage

```bash
# Open current directory in VS Code
code .

# Open a specific path
code /path/to/pbj-api

# Open a single file
code src/index.ts
```

---

## Troubleshooting

**Still not found after installing**

Open a new terminal window — the PATH change won't apply to sessions already open.

**Broke after a VS Code update**

VS Code updates occasionally remove the symlink. Re-run the installer from the Command Palette and it will be restored.

**Permission error during install**

If VS Code can't write to `/usr/local/bin`, it may prompt for your system password. This is expected — enter it and the install will complete.
