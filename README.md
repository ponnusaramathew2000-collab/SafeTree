# SafeTree

Disk space analysis that tells you what you can delete. SafeTree scans a Windows drive in seconds, shows where the space went, and marks every file and folder as safe to delete, delete with care, do not delete, your own files or unknown, with the reason. It deletes only what you select, empties Windows' own junk with a one-click Deep clean, and refuses to touch anything Windows or an installed program needs.

It works like WizTree, plus the part WizTree leaves out: whether deleting something is safe, why, and the right way to reclaim the space when plain deletion is the wrong answer.

![SafeTree with demo data](assets/screenshot.png)

## Starting SafeTree

**Unblock the zip before extracting.** Right-click the downloaded `SafeTree_Portable.zip` → **Properties** → tick **Unblock** → **OK**, then extract. This removes the "downloaded from the internet" mark that makes SmartScreen and Smart App Control treat the files as untrusted. Do this first.

Then double-click **SafeTree.exe** and click **Yes** when Windows asks for administrator rights. Nothing is installed: Python, Qt and the Visual C++ runtime are all inside the exe, so it runs from the desktop, a USB stick or a network share. The first start takes a few seconds while it unpacks.

**If SmartScreen says "Windows protected your PC"**, choose **More info** → **Run anyway**. SafeTree.exe isn't code-signed, so SmartScreen doesn't recognise it yet.

**If Smart App Control blocks SafeTree.exe** (Windows 11), there is no "run anyway" option for unsigned programs. Either have IT sign the exe once (see below) or run SafeTree from source with Python (see [Running from source](#running-from-source)).

- Needs Windows 10 (version 1809 or later) or Windows 11, 64-bit.
- **Administrator rights** give you the fast scan and Deep clean. Without them SafeTree still works, but it walks the folders instead (slower) and can't read some system folders. The title shows **[Run as administrator]** as a reminder, in amber when SafeTree is *not* running as administrator, and a button restarts it with the right permissions.
- Try the interface without scanning anything: `SafeTree.exe --demo`
- Some antivirus products quarantine unsigned PyInstaller programs. If that happens, allow SafeTree.exe or run it from source.

## Code-signing SafeTree.exe (permanent fix, for IT)

SafeTree.exe is not code-signed, so Smart App Control can block it on a fresh download even after unblocking. Signing it once with your organisation's trusted code-signing certificate makes it run everywhere without a block:

```
signtool sign /fd SHA256 /tr http://timestamp.digicert.com /td SHA256 /a SafeTree.exe
```

To rebuild the exe from source first, run **build_exe.bat** (it creates `dist\SafeTree.exe`).

## Colours

The same five colours are used in the tree, the space map, the summary bar and every dialog. Each colour always comes with its name, and "Do not delete" is also hatched, so it still reads in print, on a projector and for colour-blind users.

| Colour | Category | Meaning |
|---|---|---|
| Green | Safe to delete | Temporary or regenerated data: temp folders, caches, crash dumps, update downloads. Windows and apps are not harmed. |
| Amber | Delete with care | Removable, with a side effect: you lose a rollback option, have to re-download something, reset an app or lose history. The side effect is always shown. |
| Red, hatched | Do not delete | Windows or an installed program needs it. Many of these show the official way to shrink them instead, such as DISM for WinSxS, `powercfg /h off` for hiberfil.sys or System Protection for restore points. |
| Blue | Your files - safe | Your own data: Documents, Downloads, Pictures, and any folder Windows didn't create that holds no programs (for example `C:\Echo Show` with a device image). Deleting it can't harm Windows; it only removes that data. Kept apart from green so junk and data never get mixed. |
| Grey | Unknown - check first | Not created by Windows but contains programs (`.exe` or `.dll` files near the top), so it's probably software installed outside Program Files. Deleting it won't harm Windows but will break that program. |

## The window

- **Summary bar** (top): the whole drive split by category, plus free space.
- **Show** drop-down (above the list): *Everything*, *Can be deleted safely* (green + blue), *Safe to delete (system junk)*, *Your files - safe*, *Delete with care*, *Unknown - check first* or *Cannot be deleted*. The list and the space map then show only matching items, largest first, with the matching size in its own column.
- **List** (left): sizes like WizTree, plus a *Safe to free* column showing how much of each folder is green.
- **Cleanup** tab: every removable location found, grouped (temporary files, browser caches, Windows Update cache, crash dumps, your files and so on).
- **Windows tools** tab: large items that must be reclaimed through Windows itself, with the exact steps, a **Copy command** button and a button that opens the right Windows tool (Disk Cleanup, Storage settings, System Protection, Installed apps and so on).
- **Selected item** tab: the category, the rule that decided it, why, what the folder contains by category, and which actions are allowed.
- **Space map** (bottom): a treemap coloured by safety. Click a block to select it, double-click to zoom in, and use **Up one level** / **Whole scan** to zoom out.
- **Right-click** any item: *Move to Recycle Bin*, *Delete permanently*, *Open in Explorer*, *Copy path*, *Show in space map* and *Why this category?*
- **Keys:** **Delete** deletes the selected rows after confirmation; **Shift+Delete** deletes them permanently. Ctrl-click and Shift-click select several rows.

## Deleting files

1. Tick items in the **Cleanup** tab (*Tick all green* ticks only green system junk, never your files), or select rows in the list.
2. Press **Delete selected (size)**. Nothing is deleted yet.
3. Confirm. System junk is deleted **permanently** by default, because moving junk to the Recycle Bin frees no space. Your own files, amber and grey items go to the **Recycle Bin** by default. You can switch either way in the confirmation. Amber and grey items also need an "I've reviewed these" tick before the button unlocks.
4. The result window says what was freed and what wasn't, and why:
   - **Empty Recycle Bin now** frees the space taken by anything that went to the Recycle Bin.
   - **Delete the rest at next restart** handles files a program had open: Windows deletes them while it starts.
   - Files Windows protects are listed with the reason.

The list and the summary update straight away; no rescan is needed.

## Deep clean

Clears Windows' own junk in one go, including places a normal delete can't reach. It needs administrator rights. Each step shows how much it will free before you start and its result afterwards, with a full log. After a deep clean SafeTree scans the drive again so the view is up to date.

**Recommended (ticked by default)**

| Step | What it does |
|---|---|
| Temporary files | Empties `C:\Windows\Temp` and every user's `%TEMP%`. Files used in the last 24 hours are kept. |
| Windows Update download cache | Pauses the update services (Windows Update, BITS, Delivery Optimization), empties `SoftwareDistribution\Download` and the Delivery Optimization cache, then starts the services again. |
| Browser, app and graphics caches | Chrome, Edge, Firefox, Brave, Opera, Teams, Discord, Slack, VS Code, Spotify, GPU shader caches and thumbnails. |
| Developer tool caches | npm, Yarn, pip, Poetry, Go, NuGet and JetBrains. |
| Crash dumps and error reports | `MEMORY.DMP`, Minidump, LiveKernelReports, CrashDumps and Windows Error Reporting. |
| Windows Disk Cleanup (system files) | Runs `cleanmgr /sagerun` with SafeTree's own profile: Windows Update Cleanup, old driver packages, Defender leftovers, setup logs and more. |
| Component store cleanup (WinSxS) | `DISM /Online /Cleanup-Image /StartComponentCleanup` |

**More space (not ticked; each shows its side effect and asks again before running)**

| Step | Side effect |
|---|---|
| Empty the Recycle Bin (all drives) | Deleted files can't be restored. |
| Previous Windows installation and upgrade leftovers (`Windows.old`, `$WINDOWS.~BT`) | You can't go back to the previous version of Windows. |
| Remove superseded updates permanently (`DISM /ResetBase`) | Installed updates can't be uninstalled. |
| Older restore points (keep the newest) | Only the newest restore point remains. |
| Turn off hibernation (deletes `hiberfil.sys`) | Hibernate and Fast Startup are off. `powercfg /h on` turns them back on. |

How Deep clean stays safe:

- The file-deleting steps use the same rules as the rest of SafeTree: only "Safe to delete" locations, folder *contents* only (the folders themselves stay), files in use are skipped, and junctions or links are removed as links without touching what they point to.
- If Windows is waiting for a restart to finish installing updates, the Windows Update and DISM steps are skipped until you restart.
- The Disk Cleanup profile leaves out *Windows ESD installation files* (needed for Reset this PC), *User file versions* (File History is your data) and *Language Pack*.
- **Not touched on purpose:** event logs, Prefetch, recent-files history and the Windows Search index. They are audit and forensic records or get rebuilt at once, and they free little space.

## How SafeTree decides

**Rules.** About 370 rules cover Windows, Program Files, ProgramData, user profiles, browsers (Chrome, Edge, Brave, Vivaldi, Firefox, Opera), chat and media apps, Steam and Epic, GPU shader caches and developer tools (npm, pip, NuGet, Gradle, Maven, conda, WSL, Docker, Hugging Face, Ollama and more). Each rule says which category it gives, why, and, where it applies, the right way to reclaim the space. All rules live in `safetree/rules.py` with the reason for each.

- A rule covers its folder and everything inside it until a more specific rule takes over.
- Generic rules such as `**\node_modules` or `*.dmp` are never allowed to override a protected area.
- Folders listed in Windows' **Installed apps** (read from the registry) are marked "Installed program: *name*" and protected, with a button to uninstall the program properly instead.
- A folder Windows didn't create is blue (your files) if it holds no programs, and grey (unknown) if it holds `.exe` or `.dll` files near the top.
- Memory dumps outside app-data folders are amber, not green, because they may be evidence.
- SafeTree's own running files (unpacked into `%TEMP%`) are marked "Do not delete" while it runs.

**Scanning.** Run as administrator on an NTFS drive, SafeTree reads the Master File Table directly from `\\.\C:`, the way WizTree does, so a full system drive takes seconds. The volume is opened read-only. Hard-linked files (WinSxS and System32 share most of theirs) are counted once. In every other case (no admin rights, FAT32, exFAT, ReFS, a network share or a single folder) it walks the folders instead. Junctions such as `Documents and Settings` are never followed, so nothing is counted twice.

## Resizing the window

- Drag any edge or corner, or maximize. SafeTree opens at a size that fits your screen (maximized on small laptop screens) and remembers the size, position and divider positions you leave it at.
- The window can be made as small as about 630 × 570. As it gets narrower the toolbar compacts step by step: shorter button labels, smaller logos, then the *[Run as administrator]* reminder moves under the name, and finally the logos and *Refresh* hide. The reminder never disappears.
- The two dividers (between the list and the side panel, and above the space map) can be dragged; they highlight when you hover over them. Drag the lower one all the way down to hide the space map and give the list the full height.
- The *Name* column widens and narrows with the list until you drag it yourself; then it keeps your width. The summary bar and the space-map legend wrap onto a second line instead of overlapping, and long status messages are shortened with "..." (hover for the full text).
- The confirmation, result and Deep clean windows are resizable too, and they never open larger than the screen.

## Your own rules

Copy `custom_rules.example.json` to `%APPDATA%\SafeTree\custom_rules.json` and edit it. Restart SafeTree after editing. Custom rules win ties with the built-in ones.

```json
[
  {
    "pattern": "{SYS}\\Cases",
    "category": "personal",
    "title": "Forensic case data",
    "group": "Case data",
    "reason": "Evidence images and exports. Never delete without checking the case is closed and archived.",
    "contents_only": true
  }
]
```

The file is a list, so add more rules inside the same `[ ]`, separated by commas.

| Field | Meaning |
|---|---|
| `pattern` | The path the rule covers (required). See the placeholders below. |
| `category` | `safe`, `caution`, `keep`, `personal` or `unknown` (required). |
| `title`, `group`, `reason`, `how`, `cmd` | What SafeTree shows: name, Cleanup-tab group, why, how to reclaim the space, and a command to copy. |
| `contents_only` | `true` = empty the folder but keep the folder itself. |
| `deletable` | `false` = never offer to delete it, whatever the category. |
| `applies_in` | List of categories: the rule only applies where the surrounding area already has one of them (stops a generic rule from overriding protected areas). |
| `target` | `file` or `dir` = match only files or only folders. |

Patterns understand `{SYS}` (system drive), `{WIN}`, `{PF}`, `{PF86}`, `{PD}` (ProgramData), `{USERS}`, `{U}` (any user profile), `{L}` / `{R}` / `{LL}` (that profile's AppData Local / Roaming / LocalLow), `{ANY}` (any drive), `*` and `?` (wildcards within one folder name) and `**` (any number of folders).

## Text report and disk images

SafeTree can print a report without opening a window, from a live drive or from a raw (dd) image of an NTFS volume. Open a Command Prompt **as administrator**:

```
SafeTree.exe --report C:\                                 # live drive (MFT scan)
SafeTree.exe --report --image disk.dd                     # raw/dd image of an NTFS volume
SafeTree.exe --report --image full.dd --offset 1048576    # partition inside a full-disk image
```

| Option | Meaning |
|---|---|
| `path` | Drive or folder to scan (also works without `--report`: the window opens and scans it). |
| `--demo` | Show made-up data instead of scanning. |
| `--report` | Print a text report; no window. |
| `--image FILE` | Read a raw NTFS image instead of a live drive (report mode). |
| `--image-root C:\` | The drive letter the image represents, so the rules match (default `C:\`). |
| `--offset BYTES` | Where the partition starts inside a full-disk image (default 0). |
| `--top N` | How many cleanup items the report lists (default 25). |

Images must be raw (dd). E01 and other container formats aren't supported. Reading an image never changes it, which makes this the way to look at evidence instead of a live machine.

## What SafeTree changes on your PC

- **No network.** SafeTree never connects to the internet: no telemetry, no update checks, no cloud lookups.
- **Nothing is installed.** No services, scheduled tasks or startup entries. While it runs, the exe unpacks itself into `%TEMP%\_MEI…` and removes that folder when it closes.
- **Window size and divider positions** are saved in the registry under `HKCU\Software\SafeTree\SafeTree`.
- **Custom rules** are read from `%APPDATA%\SafeTree\custom_rules.json`, only if you create it.
- **Deep clean** saves its Disk Cleanup profile as `StateFlags0731` values under `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\VolumeCaches`, and briefly stops and restarts the Windows Update, BITS and Delivery Optimization services.
- **Delete the rest at next restart** uses Windows' own pending-delete list (`MoveFileEx`), which Windows clears when it deletes the files.
- **Scanning only reads.** The fast scan opens the volume read-only. Only deleting changes the drive, and only after you confirm. (On a live machine, Windows itself may update folder last-access times when any program lists them; use `--report --image` to examine evidence instead.)

## Safety guards

- "Do not delete" items can't be deleted from SafeTree at all, and neither can a folder that contains one.
- A fixed list of locations is refused whatever the rules say: drive roots, `Windows`, `System32`, `SysWOW64`, `WinSxS`, `Program Files`, `ProgramData`, `Users`, each profile folder and its `AppData`.
- Temp and cache folders are *emptied*, never removed, so apps don't break looking for them. Files in use are skipped and listed afterwards.
- Amber and grey items need an explicit "I've reviewed these" tick before the delete button unlocks. Your own files (blue) only need the normal confirmation.
- *Tick all green* in the Cleanup tab ticks only green system junk, never your files.
- Windows warns before anything too big for the Recycle Bin is deleted permanently.
- SafeTree can't delete its own running files.

## Running from source

1. Install Python 3.10 or newer from python.org (tick **Add python.exe to PATH**).
2. In the SafeTree folder, run `pip install -r requirements.txt`
3. Right-click **SafeTree (run as admin).bat** → **Run as administrator**. (From a Command Prompt: `python -m safetree`, with the same options as SafeTree.exe.)

## What was tested, and what wasn't

- **Rule engine:** 77 Windows locations checked, plus the delete guards, installed-program detection and folders Windows didn't create.
- **MFT parser:** compared against real NTFS volumes created with `mkntfs` (512 B, 4 KB and 64 KB clusters, hard links, a file with 61 hard-link names, sparse and fragmented files, Unicode names, deleted files, 10,000+ records), on Linux and again through Windows file I/O.
- **Windows code paths**, run with a Windows build of Python under Wine: admin detection, the drive list, the Recycle Bin (including a locked file in the same batch), permanent deletion of paths longer than 260 characters, `%TEMP%` handling, the fallback from MFT scan to folder walk, Deep clean, delete at next restart, window resizing, and the finished `SafeTree.exe` itself (window, scan, text report).
- **Not testable without real Windows hardware:** reading `\\.\C:` directly on a live system drive, and the UAC prompt. Scan once and try a Recycle Bin delete on something unimportant before cleaning your system drive.

Known limits: alternate data streams aren't counted; in folder-walk mode "on disk" is estimated from the cluster size; OneDrive online-only files correctly show 0 bytes on disk.

## Tests

```
python -m unittest discover -s tests
```

The MFT test runs only on Linux as root with `mkntfs` and `ntfs-3g` installed; it is skipped elsewhere.

## Files

```
SafeTree.exe                 the program (portable zip)
custom_rules.example.json    starting point for your own rules
safetree/
  scan_mft.py                raw NTFS $MFT reader (fast scan)
  scan_walk.py               folder walk (fallback)
  rules.py                   the safety rules
  classifier.py              applies the rules, delete guards
  deleter.py                 Recycle Bin / permanent delete, tree update
  deepclean.py               Deep clean steps
  winapi.py                  Windows calls (admin, drives, Recycle Bin, services, tools)
  treemap_layout.py          space map layout
  gui/                       PySide6 interface
  cli.py, demo.py            text report, demo data
tests/
build_exe.bat                builds dist\SafeTree.exe
SafeTree (run as admin).bat  runs from source
```

## Change log

**1.0** - first release.

- Fast MFT scan of NTFS drives, with a folder-walk fallback for everything else.
- Five safety categories decided by about 370 rules, with the reason and the right way to reclaim space for each.
- **Show** filter for what can and can't be deleted.
- Cleanup tab and a delete flow with Recycle Bin or permanent deletion, *Empty Recycle Bin now* and *Delete the rest at next restart*.
- Deep clean: 12 steps, including Disk Cleanup, DISM, restore points and hibernation.
- Windows tools tab, space map coloured by safety, resizable window.
- Text report from live drives and raw NTFS images; custom rules.
