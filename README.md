# Spicetify Fix — User Mode

> If you use a modified Windows image where every shell runs elevated, and
> Spicetify's installer refuses to run because of it, this tiny launcher gets
> the official installer into a non-elevated context.

## The problem

Some modified Windows builds launch terminals as `Administrator` all the time.
Spicetify's installer expects a normal user context and exits when it detects
elevation:

```powershell
iwr -useb https://raw.githubusercontent.com/spicetify/cli/main/install.ps1 | iex
```

## What this does

`SpicetifyFix` is a 500-line WinForms launcher with one button:

- If the current process **is elevated**, it starts PowerShell through
  `runas /trustlevel:0x20000` (a restricted "basic user" token) and runs the
  official Spicetify installer there.
- If it is **not elevated**, it starts PowerShell directly.
- The window stays open (`-NoExit`) so you can read the result.

It does not download, modify or bundle Spicetify itself, and it does not loop
or relaunch itself.

## Repository contents

```text
Spicetify-Fix-UserMode/
├── README.md
└── src/
    └── SpicetifyFix.cs   # the whole app
```

There is **no prebuilt EXE and no `run_spicetify.ps1` in the repository.**
The app creates `run_spicetify.ps1` next to the EXE on first use, containing
the official installer command and `pause`.

## Build

Compile with the .NET Framework compiler that ships with Windows:

```powershell
& "$env:WINDIR\Microsoft.NET\Framework64\v4.0.30319\csc.exe" `
  /target:winexe /out:SpicetifyFix.exe `
  /reference:System.Windows.Forms.dll /reference:System.Drawing.dll `
  src\SpicetifyFix.cs
```

The result is a small framework-dependent EXE that runs on any Windows 10/11
with .NET Framework 4.x (preinstalled).

## Use

1. Put `SpicetifyFix.exe` in any writable folder.
2. Double-click it and press **Install Spicetify**.
3. A PowerShell window opens and runs the official installer. Keep it open
   until it finishes.

### Manual alternative

You do not need this tool. Open a normal (non-elevated) PowerShell and run:

```powershell
iwr -useb https://raw.githubusercontent.com/spicetify/cli/main/install.ps1 | iex
```

If your shell always starts elevated, `runas /trustlevel:0x20000 powershell`
gives you a restricted token for the same account. Note that this is **not** a
separate user account or a security boundary — it is the same profile with a
filtered token.

## Requirements

- Windows 10 / 11
- .NET Framework 4.x
- Spotify installed
- Internet access for the official Spicetify installer

## Notes and limits

- The app only automates one command; it does not patch Spotify or install
  themes/marketplace by itself.
- Do not run the installer as a different administrator account if Spotify is
  installed per-user — Spicetify installs into the current profile.
- The official installer prints its own prompts and errors; read them.

## Credits

DEV: Mr-Glint. Spicetify is a separate open-source project:
<https://github.com/spicetify/cli>.

## License

No license file is included. The source is published for reference; contact
the author before redistributing.

---

<div align="center">
  <sub>Built by <a href="https://github.com/M-GOLEM"><b>Mr.GOLEM</b></a> ·
  watch the work on <a href="https://www.youtube.com/@golemsec">YouTube — GOLEM SEC</a></sub>
</div>
