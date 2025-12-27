---
related:
  - "[[PC]]"
created: 2025-12-27
---
# Windows Reference

[[references]]

## Disable showing group thumbnail in taskbar

```
Settings > System > Multi-tasking > Snap windows > Show my snapped windows when I hover over taskbar apps, in Task VIew, and when I press Alt+Tab
```

## Disable Animation Effect

```
Settings > Accessibility > Visual effects > Animation effects
```

## Disable F1 Opening Help

> [Source Link](https://www.winhelponline.com/blog/disable-f1-key-help-windows-10/)

### Lockdown helppane.exe

`HelpPane.exe` is the file that launches the Bing search help and support page in Microsoft Edge.
Locking down `helppane.exe` can prevent from opening help when F1 key is pressed.

1. Open an admin Command Prompt window
2. Run the command bellow

```cmd
taskkill /f /im HelpPane.exe
takeown /f %WinDir%\HelpPane.exe
icacls %WinDir%\HelpPane.exe /deny Everyone:(X)
```

With the above permission entries, when F1 is pressed, `explorer.exe` will get Access Denied error when launching `helppane.exe` and silently fail.
This the Bing search page won’t be launched. 

### Unlock helppane.exe (Revert)

To revert the `helppane.exe` permissions to defaults, run these commands from admin Command Prompt:

```cmd
icacls %WinDir%\HelpPane.exe /remove:d Everyone
icacls %WinDir%\HelpPane.exe /setowner "NT Service\TrustedInstaller"
```

If the 2nd command fails to execute and throws the error:

```
C:\WINDOWS\HelpPane.exe: Access is denied.
Successfully processed 0 files; Failed processing 1 files
```

Then, use the Permissions GUI to change ownership to NT Service\TrustedInstaller.

## Prevent installed fonts disappearing after Windows restart

1. Open `%appdata%\..\Local\Microsoft\Windows\Fonts`
  1. The directory should contain all installed fonts
2. Press `Ctrl + A` to select all font files
3. R-Click to open context menu
4. Press `Install for all users`
