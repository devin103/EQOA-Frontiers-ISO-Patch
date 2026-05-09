# EQOA Frontiers ISO Patch

A patcher for the *EverQuest Online Adventures: Frontiers* PS2 ISO. It modifies an unmodified retail ISO so DNAS authentication is bypassed and the iLinkID check is zeroed out, allowing the game to decrypt its content using the same key path it would have used at install time on a console that originally had a zeroed iLinkID.

## What this patcher does

Given an unmodified EQOA Frontiers ISO, it can produce a modified ISO with one or both of these patches applied:

- **DNAS bypass** — replaces the relevant DNAS2 stub functions in `UTIL.REL` so they return success without contacting the (long-defunct) Sony auth servers.
- **iLinkID fix** — patches the `CDVDMAN.IRX` byte-store inside `DNAS255.IMG` so the runtime iLinkID returned to the game is always zero. This matches the iLinkID that the game's encrypted payload was originally protected against, allowing decryption to succeed.

Before patching, the script verifies:

- The whole-ISO **SHA-256** matches a known-good fingerprint of the unmodified retail ISO.
- The main game ELF (`SLUS_207.44`) **XOR checksum** matches the expected build.

If either check fails, the script aborts without modifying the ISO. This prevents accidentally corrupting a different build.

## ⚠️ Before you start: back up your ISO

**This patcher modifies the ISO in place.** There is no undo. Before running the script, make a copy of your clean unmodified ISO and keep it somewhere safe — preferably on different storage from the working copy.

You will need that clean copy if any of these happen:

- Something goes wrong mid-patch (power loss, disk error)
- You apply the wrong patch and want to redo
- You initially apply only DNAS or only iLinkID, then later decide you want the other one too — see below

### About applying just one of the two patches

The script lets you pick: **DNAS only**, **iLinkID only**, or **Both**.

If you choose just one, the resulting ISO has only that patch applied. **You cannot run the script on it again to add the other patch later** — the integrity checks (SHA-256 + ELF checksum) only accept a clean unmodified ISO. A partially-patched ISO will be rejected with a checksum mismatch.

So if you applied DNAS-only and decide you want the iLinkID fix added too, you have to:

1. Discard the partially-patched ISO
2. Get a fresh copy of your clean backup
3. Run the script again and pick "Both" (or "iLinkID only" if you really only want that one)

This is by design — the integrity checks are what guarantee the script never lands patches on a build it doesn't understand. **Keep multiple backups of your clean ISO** so you can iterate without re-acquiring it.

## Compatibility

| Target | Status |
|---|---|
| **PCSX2 emulator** | ✅ Works |
| **Real PS2 hardware via burned DVD-R** | ✅ Should work (boot via FMCB → ESR) |
| **Real PS2 hardware via FreeMcBoot + OPL (USB/HDD)** | ❌ Doesn't currently work — OPL ships its own `CDVDMAN.IRX` that overrides the patched one in the ISO |

The OPL limitation is a known issue; OPL bundles its own CDVD modules and substitutes them at boot, so the iLinkID-zeroing patch in the disc ISO never executes on that boot path. A patched OPL build would be needed to fix this.

## Download

Pre-built executables for each platform are available on the [Releases page](https://github.com/devin103/EQOA-Frontiers-ISO-Patch/releases). No Python installation required — pick the binary matching your OS.

| Platform | Filename |
|---|---|
| Linux x86_64 | `eqoa-frontiers-iso-patch-linux-x86_64` |
| Windows x86_64 | `eqoa-frontiers-iso-patch-windows-x86_64.exe` |
| macOS Apple Silicon | `eqoa-frontiers-iso-patch-macos-arm64` |

If you have a Python 3.6+ installation, you can also run the source script directly without a binary download — see [Run from source](#run-from-source) below.

## Usage

### Quick start

The patcher takes one argument: the path to your unmodified EQOA Frontiers ISO. **It modifies the ISO in place — back up first.**

#### Windows

Three ways to run it:

**1. Drag-and-drop (simplest):** drag your ISO file onto `eqoa-frontiers-iso-patch-windows-x86_64.exe` in Explorer.

**2. Command Prompt or PowerShell:**
```
eqoa-frontiers-iso-patch-windows-x86_64.exe "C:\path\to\EQOA Frontiers.iso"
```

**3. Double-click then read the prompt:** double-clicking with no argument prints usage. The window stays open until you press a key, so you can see the message.

> **First run:** Windows SmartScreen may show "Windows protected your PC" — click **More info → Run anyway**. The binary isn't code-signed, which is normal for open-source releases.

#### Linux

```bash
# Mark executable (downloads strip the +x bit)
chmod +x eqoa-frontiers-iso-patch-linux-x86_64

# Run it
./eqoa-frontiers-iso-patch-linux-x86_64 /path/to/EQOA-Frontiers.iso
```

#### macOS (Apple Silicon)

```bash
# Mark executable
chmod +x eqoa-frontiers-iso-patch-macos-arm64

# First-time only: strip Gatekeeper quarantine attribute
xattr -d com.apple.quarantine eqoa-frontiers-iso-patch-macos-arm64

# Run it
./eqoa-frontiers-iso-patch-macos-arm64 /path/to/EQOA-Frontiers.iso
```

> The `xattr` step is required because the binary isn't signed by an Apple Developer ID. Alternative: right-click the binary in Finder → **Open** → confirm in the dialog.

If you have an Intel Mac, install Rosetta 2 first:
```bash
softwareupdate --install-rosetta --agree-to-license
```

### Run from source

If you prefer not to download a binary, the script runs directly with Python 3.6 or newer (no third-party dependencies):

```bash
# Linux / macOS
python3 "EQOA Frontiers ISO Patch.py" /path/to/EQOA-Frontiers.iso

# Windows
python "EQOA Frontiers ISO Patch.py" "C:\path\to\EQOA Frontiers.iso"
```

The double quotes around the script name are required on every platform because of the spaces in the filename.

### Choosing which patches to apply

After integrity checks pass, the script asks which patches to apply:

```
Which patches do you want to apply?
  [1] DNAS bypass only       (modifies UTIL.REL)
  [2] iLinkID fix only       (modifies DNAS255.IMG)
  [3] Both                   (default)
```

Type `1`, `2`, `3`, or just press Enter to accept the default (Both). Most users will want option 3.

> **Reminder:** if you pick 1 or 2, you can't later run the script again on the same ISO to add the other patch. The integrity checks won't accept a partially-patched ISO. To add the missing patch, start from a fresh copy of your clean unmodified backup.

For automation/scripting, set the `EQOA_PATCH_MODE` environment variable to `dnas`, `ilink`, or `both` to skip the prompt:
```bash
EQOA_PATCH_MODE=both ./eqoa-frontiers-iso-patch-linux-x86_64 my-iso.iso
```

In non-interactive contexts (output redirected to a file, CI, etc.), the script defaults to applying both patches automatically.

## What you should see

A successful run looks roughly like this:

```
[*] Opening EQOA-Frontiers.iso  (4,295,098,368 bytes, 4096 MB)
    Detected ISO9660 image
[*] Verifying ISO SHA-256...
    SHA-256:  f1db24a5...  ✓ (unmodified)
[*] Verifying main game ELF checksum
    File:     /SLUS_207.44  (400,296 bytes)
    Expected: 0xEEEE1FCC
    Actual:   0xEEEE1FCC  ✓ (unmodified game build)

Which patches do you want to apply?
  [1] DNAS bypass only       (modifies UTIL.REL)
  [2] iLinkID fix only       (modifies DNAS255.IMG)
  [3] Both                   (default)

Enter 1, 2, or 3 (or just press Enter for both): 3

[*] Locating MODULES/DNAS255.IMG ...
    [+] Patched at 0x000235CC
[*] Searching ISO for UTIL.REL files ...
    sceDNAS2InitNoHDD            ✓
    sceDNAS2AuthInstall          ✓
    ... (8 more functions) ...
[+] Done.

Press any key to exit...
```

If any integrity check fails, the script prints which one and exits without modifying the file.

## Troubleshooting

| Error | Cause | Fix |
|---|---|---|
| `ISO SHA-256 mismatch` | Your ISO has been modified, is a different region/revision, is partially-patched from a previous run, or is a re-mastered version | This patcher only supports the specific unmodified retail build. If it's partially patched, restore from your clean backup |
| `Game ELF checksum mismatch` | Main game ELF differs from the expected build | Same as above |
| `cannot be opened because the developer cannot be verified` (macOS) | Gatekeeper quarantine | Run `xattr -d com.apple.quarantine ./binary-name` |
| `Permission denied` (Linux/macOS) | Forgot `chmod +x` | `chmod +x ./binary-name` |
| `'python' is not recognized` (Windows) | Python not on PATH | Reinstall Python with "Add to PATH" checked, or use the precompiled .exe |
| `python: command not found` (Linux/macOS) | Modern systems use `python3`, not `python` | Use `python3` instead |
| Want to add the missing patch after applying just one | Partially-patched ISO is rejected by integrity checks | Restore from clean backup, run again, pick "Both" |

## Building from source

The releases are built automatically by GitHub Actions on every tag push. If you want to build a binary yourself:

```bash
pip install pyinstaller
pyinstaller --onefile --console --noupx --name eqoa-frontiers-iso-patch "EQOA Frontiers ISO Patch.py"
```

The output binary appears in `dist/`.

## Disclaimer

This tool is intended for preserving access to legally-owned copies of EQOA Frontiers after the official servers shut down. Distributing patched ISOs or game files is your responsibility — this tool only modifies a copy you provide.

## License

[Add license info here — MIT, GPL, etc.]
