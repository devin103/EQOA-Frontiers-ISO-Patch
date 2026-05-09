# EQOA Frontiers ISO Patch

A patcher for the *EverQuest Online Adventures: Frontiers* PS2 ISO. It modifies an unmodified retail ISO so DNAS authentication is bypassed and the iLinkID check is zeroed out, allowing the game to decrypt its content using the same key path it would have used at install time on a console that originally had a zeroed iLinkID.

## What this patcher does

Given an unmodified EQOA Frontiers ISO, it produces a modified ISO that:

- **Bypasses DNAS authentication** — replaces the relevant DNAS2 stub functions in `UTIL.REL` so they return success without contacting the (long-defunct) Sony auth servers.
- **Zeros out the iLinkID retrieval** — patches the `CDVDMAN.IRX` byte-store inside `DNAS255.IMG` so the runtime iLinkID returned to the game is always zero. This matches the iLinkID that the game's encrypted payload was originally protected against, allowing decryption to succeed.

Before patching, the script verifies:

- The whole-ISO **SHA-256** matches a known-good fingerprint of the unmodified retail ISO.
- The main game ELF (`SLUS_207.44`) **XOR checksum** matches the expected build.

If either check fails, the script aborts without modifying the ISO. This prevents accidentally corrupting a different build.

## Compatibility

| Target | Status |
|---|---|
| **PCSX2 emulator** | ✅ Works |
| **Real PS2 hardware via burned DVD-R** | ✅ Assumed this should work (boot via FMCB → ESR) |
| **Real PS2 hardware via FreeMcBoot + OPL (USB/HDD)** | ❌ Doesn't currently work — OPL ships its own `CDVDMAN.IRX` that overrides the patched one in the ISO. DNAS patching itself should still work, however and is beneficial to avoid pnach/cheats for that alone. |

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

The patcher takes one argument: the path to your unmodified EQOA Frontiers ISO. It modifies the ISO **in place**, so make a backup first.

#### Windows

Three ways to run it:

**1. Drag-and-drop (simplest):** drag your ISO file onto `eqoa-frontiers-iso-patch-windows-x86_64.exe` in Explorer.

**2. Command Prompt or PowerShell:**
