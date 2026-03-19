---
layout: page
title: Terminal mod - Starship & OneDark Theme
parent: Other
---

# Starship terminal setup on Windows

This guide covers installing [Starship](https://starship.rs/) on Windows, setting up Nerd Fonts, and integrating everything into Windows Terminal and VS Code with the OneDark theme.

## Install a Nerd Font

Starship uses symbols that require a patched font. Install a Nerd Font of your choice (e.g. **FiraCode Nerd Font** or **CaskaydiaCove Nerd Font**):

1. Download from [nerdfonts.com](https://www.nerdfonts.com/font-downloads)
2. Extract the archive, select all `.ttf` files, right-click → **Install for all users**

Alternatively, use `winget`:

```powershell
winget install --id=Nerdfont.FiraCode -e
```

Or install via **Chocolatey**:

```powershell
choco install nerd-fonts-firacode
```

## Install Starship

Install Starship via `winget`:

```powershell
winget install --id Starship.Starship
```

Or via **Chocolatey**:

```powershell
choco install starship
```

## Install PowerShell 7+

This guide targets **PowerShell 7+** (cross-platform, `pwsh.exe`), not the legacy Windows PowerShell 5.1. Install it via `winget`:

```powershell
winget install --id Microsoft.PowerShell -e
```

After installation, `pwsh` should be available in your PATH. Verify with:

```powershell
pwsh --version
```

## Configure your shell to use Starship

### PowerShell 7+

Open your PowerShell 7 profile. Note that `pwsh` uses a separate profile path from Windows PowerShell 5.1:

```powershell
pwsh -Command "if (!(Test-Path $PROFILE)) { New-Item -Path $PROFILE -Type File -Force }; notepad $PROFILE"
```

Add the following line to the profile:

```powershell
Invoke-Expression (&starship init powershell)
```

### CMD

For CMD integration, install [Clink](https://chrisant996.github.io/clink/) and add the following to a `starship.lua` file in your Clink scripts directory (`clink info` shows the path):

```lua
load(io.popen('starship init cmd'):read("*a"))()
```

## Starship configuration

Starship is configured via `~/.config/starship.toml`. Create the file if it doesn't exist:

```powershell
mkdir -Force ~/.config
New-Item -Path ~/.config/starship.toml -Type File
```

A minimal configuration to get started:

```toml
# ~/.config/starship.toml

format = """
$directory\
$git_branch\
$git_status\
$dotnet\
$nodejs\
$python\
$docker_context\
$line_break\
$character"""

[character]
success_symbol = "[❯](bold green)"
error_symbol = "[❯](bold red)"

[directory]
truncation_length = 5
truncation_symbol = "…/"

[git_branch]
symbol = " "

[git_status]
format = '([\[$all_status$ahead_behind\]]($style) )'
```

Browse the full configuration options at [starship.rs/config](https://starship.rs/config/).

You can also use one of the bundled presets:

```powershell
starship preset nerd-font-symbols -o ~/.config/starship.toml
```

## Windows Terminal – OneDark theme and Nerd Font

Open Windows Terminal settings (`Ctrl+,`) → **Open JSON file**, and add the OneDark color scheme to the `schemes` array:

```json
{
    "name": "OneDark",
    "background": "#282C34",
    "foreground": "#ABB2BF",
    "cursorColor": "#528BFF",
    "selectionBackground": "#3E4451",
    "black": "#282C34",
    "red": "#E06C75",
    "green": "#98C379",
    "yellow": "#E5C07B",
    "blue": "#61AFEF",
    "purple": "#C678DD",
    "cyan": "#56B6C2",
    "white": "#ABB2BF",
    "brightBlack": "#5C6370",
    "brightRed": "#E06C75",
    "brightGreen": "#98C379",
    "brightYellow": "#E5C07B",
    "brightBlue": "#61AFEF",
    "brightPurple": "#C678DD",
    "brightCyan": "#56B6C2",
    "brightWhite": "#FFFFFF"
}
```

Then apply the scheme and Nerd Font to your profile in the `profiles.defaults` or a specific profile:

```json
{
    "defaults": {
        "colorScheme": "OneDark",
        "font": {
            "face": "FiraCode Nerd Font",
            "size": 11
        },
        "opacity": 95,
        "useAcrylic": false
    }
}
```

Make sure **PowerShell 7** is set as the default profile in Windows Terminal. Under `profiles.list`, ensure the `pwsh.exe` entry exists (it is usually auto-detected). Then set it as default via `defaultProfile` in the JSON root:

```json
{
    "defaultProfile": "{GUID-of-PowerShell-7-profile}"
}
```

You can find the GUID in the `profiles.list` array for the entry named "PowerShell" with `"source": "Windows.Terminal.PowershellCore"`. Alternatively, set this via the Terminal UI under **Settings → Startup → Default profile**.

## VS Code – Starship integration with OneDark

### Terminal font

In VS Code settings (`Ctrl+,`), configure the integrated terminal to use the Nerd Font. Add to your `settings.json`:

```json
{
    "terminal.integrated.fontFamily": "'FiraCode Nerd Font', monospace",
    "terminal.integrated.fontSize": 14
}
```

This ensures Starship glyphs render correctly in the VS Code terminal.

### OneDark theme

Install the **One Dark Pro** extension from the marketplace:

1. Open Extensions (`Ctrl+Shift+X`)
2. Search for **One Dark Pro**
3. Click **Install**
4. Set it as active: `Ctrl+K Ctrl+T` → select **One Dark Pro**

### Terminal color scheme (OneDark)

To apply OneDark colors specifically to the integrated terminal, add to `settings.json`:

```json
{
    "workbench.colorCustomizations": {
        "terminal.background": "#282C34",
        "terminal.foreground": "#ABB2BF",
        "terminalCursor.foreground": "#528BFF",
        "terminal.ansiBlack": "#282C34",
        "terminal.ansiRed": "#E06C75",
        "terminal.ansiGreen": "#98C379",
        "terminal.ansiYellow": "#E5C07B",
        "terminal.ansiBlue": "#61AFEF",
        "terminal.ansiMagenta": "#C678DD",
        "terminal.ansiCyan": "#56B6C2",
        "terminal.ansiWhite": "#ABB2BF",
        "terminal.ansiBrightBlack": "#5C6370",
        "terminal.ansiBrightRed": "#E06C75",
        "terminal.ansiBrightGreen": "#98C379",
        "terminal.ansiBrightYellow": "#E5C07B",
        "terminal.ansiBrightBlue": "#61AFEF",
        "terminal.ansiBrightMagenta": "#C678DD",
        "terminal.ansiBrightCyan": "#56B6C2",
        "terminal.ansiBrightWhite": "#FFFFFF"
    }
}
```

> **Note:** If you're using the One Dark Pro theme, the terminal colors are already included. The above is useful if you want to override or use a different editor theme while keeping OneDark in the terminal.

### Default shell

Ensure VS Code uses **PowerShell 7** as the default terminal profile so Starship initializes correctly. The profile name for `pwsh` is `"PowerShell"` (not `"Windows PowerShell"`, which refers to the legacy 5.1):

```json
{
    "terminal.integrated.defaultProfile.windows": "PowerShell"
}
```

Verify the correct shell is used by running `$PSVersionTable.PSVersion` in the VS Code terminal — it should show version **7.x** or higher.
