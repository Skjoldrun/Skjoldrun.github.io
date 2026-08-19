---
layout: page
title: Linux - opencode sudo permissions
parent: Linux
---

# opencode sudo permissions

When an AI coding assistant needs admin rights on Linux, the tempting shortcut is to launch it with `sudo`. That grants the whole agent root powers, not just the few privileged commands it actually needs. This article shows how to give opencode selective sudo access through its `permission` config so routine admin tasks (package installs, service management, log reads) run automatically while everything else stays locked down.

## The problem: running the agent as root

Starting opencode with `sudo opencode` works, but it escalates the *entire* process. Every file read, every write, every command then runs as root:

```
sudo opencode
  |
  +-- opencode process (root)
        |
        +-- file reads          -> root, can read anything
        +-- edits               -> root, can overwrite anything
        +-- bash commands       -> root, no sudo needed
        +-- background tools    -> root
```

That is far more than the few commands you actually want to delegate. One malformed `rm` or a stray write to `/etc` and the agent silently mutates the system without ever asking.

## The fix: `permission.bash` rules

opencode's config supports a `permission` block that maps bash command patterns to actions. For each command the agent wants to run, opencode picks the **last matching rule**:

| Action | Effect |
|---|---|
| `allow` | Runs the command automatically, no prompt |
| `ask` | Asks the user before running |
| `deny` | Blocks the command outright |

Because the last match wins, you order the rules broad-to-narrow: general patterns first, the exceptions last.

## A working example

The following global config (at `~/.config/opencode/opencode.json`) grants automatic sudo for common admin tasks and blocks user management plus shutdown:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "permission": {
    "bash": {
      "sudo pacman *": "allow",
      "sudo systemctl *": "allow",
      "sudo journalctl *": "allow",
      "sudo rm *": "allow",
      "sudo chmod *": "allow",
      "sudo chown *": "allow",
      "sudo cp *": "allow",
      "sudo mv *": "allow",
      "sudo mkdir *": "allow",
      "grep *": "allow",
      "rg *": "allow",
      "cat *": "allow",
      "head *": "allow",
      "tail *": "allow",
      "ls *": "allow",
      "diff *": "allow",
      "which *": "allow",
      "type *": "allow",
      "ps *": "allow",
      "uptime *": "allow",
      "df *": "allow",
      "find *": "ask",
      "sudo usermod *": "deny",
      "sudo passwd *": "deny",
      "sudo shutdown *": "deny",
      "sudo reboot *": "deny",
      "sudo *": "ask",
      "*": "ask"
    }
  }
}
```

What each allowed command unlocks:

| Pattern | What the agent may now do |
|---|---|
| `sudo pacman *` | Install, update and remove packages (Arch / CachyOS) |
| `sudo systemctl *` | Start, stop and enable systemd services |
| `sudo journalctl *` | Read system and service logs for diagnostics |
| `sudo rm *` | Delete files in system areas |
| `sudo chmod *`, `sudo chown *` | Change permissions and ownership |
| `sudo cp *`, `sudo mv *` | Copy and move system files (e.g. config backups) |
| `sudo mkdir *` | Create directories outside the home folder |

### Read-only commands

Read-only CLI tools like `grep` or `ls` are safe to auto-allow: they never modify the filesystem. With the `"*": "ask"` catch-all in place they would otherwise prompt on every run, which gets noisy fast.

| Pattern | Notes |
|---|---|
| `grep *`, `rg *` | Text search (ripgrep is the faster alternative) |
| `cat *`, `head *`, `tail *` | Show file contents |
| `ls *` | List directories |
| `diff *` | Compare files |
| `which *`, `type *` | Locate programs |
| `ps *`, `uptime *`, `df *` | System status (processes, load, disk) |

Two caveats:

- **`find` is not read-only.** It supports `-delete` and `-exec`, both of which can modify or delete files. Keep it on `ask` (or `deny`) unless you are sure the agent will only use read flags like `-name` or `-iname`.
- **Shell redirection still writes.** A pattern like `cat *` also matches `cat file > /etc/foo`, because the permission check matches the parsed command before any redirect is applied. For sensitive destinations that matters, but in practice the risk is small for the commands above.

Anything else starting with `sudo` falls through to `ask` and is presented to you, and plain (non-sudo) commands default to `ask` too, so the agent can still ask for one-off privileges it doesn't have yet.

> The rule set is opinionated: `rm`, `chmod` and friends are genuinely dangerous. Only allow them if you trust the agent to operate within your system and you keep a config backup (the example includes a `.bak` file) handy.

## How the interaction feels

With these rules in place, a typical workflow is seamless:

```
User:  "Install nginx and enable it"
Agent: sudo pacman -S nginx          -> allowed, runs silently
       sudo systemctl enable nginx   -> allowed, runs silently
Agent: sudo nginx -t                 -> not matched by any allow rule
                                          -> opencode asks the user
```

The agent no longer needs to be started as root, and your user account only ever escalates for the specific commands you approved.

## Pitfalls

- **Restart required.** Config is loaded once at startup and is not hot-reloaded. After editing `opencode.json`, quit and restart opencode.
- **Pattern matching is literal.** `"sudo pacman *"` requires a space after `pacman`; a bare `sudo pacman` without arguments does not match the allow rule and lands on `ask`. If you want the bare form too, add `"sudo pacman": "allow"`.
- **Order matters.** The last matching pattern wins, so list broad rules first and specific deny rules last. A `"sudo *": "deny"` placed *after* your allow rules would silently block everything above it.
- **`sudo -n` is not handled specially.** If sudo is configured to require a password, the `allow` rule runs the command but the password prompt still appears in the shell. For fully unattended runs, configure passwordless sudo for those commands in `/etc/sudoers.d/` yourself.

## Running unattended with auto mode

The permission rules above control *what* the agent may run. For fully unattended runs, opencode additionally offers auto mode, which auto-approves every permission request that is not explicitly denied:

```bash
# Interactive TUI with auto-approve
opencode --auto

# Headless batch run until the task finishes
opencode run --auto "Refactor this module"
```

In the TUI you can also toggle it at runtime via the command palette (**Enable auto-approve permissions**); a muted `auto` indicator appears next to the current agent.

Caveats before you leave the agent alone:

- **`deny` rules still apply.** Auto mode only upgrades `ask` to `allow`; the `usermod`, `passwd`, `shutdown` and `reboot` blocks from the example remain enforced.
- **`question` prompts still block.** If the agent asks you something via its question tool, that is not a permission request, so auto mode cannot override it - the run stops until someone answers.
- Prefer `opencode run --auto` over `--auto` in the TUI for headless batch jobs, since the TUI expects an interactive session.

## Global vs. project scope

The example uses the global config at `~/.config/opencode/opencode.json`, which applies in every project. For a stricter setup, put a smaller `permission` block in a project-level `opencode.json` instead - project config deep-merges over global, so you can tighten or relax rules per repository.

## Sources

- opencode permissions reference (auto mode, patterns, defaults): https://opencode.ai/docs/permissions
- opencode configuration reference: https://opencode.ai/docs/config
- opencode JSON schema (authoritative shapes): https://opencode.ai/config.json