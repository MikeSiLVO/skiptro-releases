<p align="center">
  <h1 align="center">Skiptro</h1>
  <p align="center">
    Automatic TV show intro detection for Kodi
    <br />
    <a href="https://github.com/MikeSiLVO/skiptro-releases/releases/latest"><strong>Download Latest Release</strong></a>
  </p>
</p>

---

Skiptro uses audio fingerprinting and machine learning to detect TV show intros across episodes in a season, then exports skip markers that Kodi can use to automatically skip them during playback.

Each release includes both a **Desktop GUI app** and a **CLI** for automation.

## Features

- Detects intros by comparing audio across episodes in a season
- Exports `.skiptro.json` files for the companion Kodi add-on
- Optional ffmpeg chapter file export for users who prefer to embed chapters
- Self-contained — no runtime or dependencies required
- Needs at least 2 episodes per season to work

## Supported Platforms

| Platform | Architecture | FFmpeg Included |
|----------|-------------|-----------------|
| Windows | x64 | Yes |
| Linux | x64 | Yes |
| Linux | ARM64 (Raspberry Pi) | Yes |
| macOS | x64 (Intel) | Yes |
| macOS | ARM64 (Apple Silicon) | Yes |

All releases include FFmpeg 8.0 bundled — no additional dependencies required.

## Installation

1. Download the archive for your platform from [Releases](https://github.com/MikeSiLVO/skiptro-releases/releases/latest)
2. Extract to a directory of your choice
3. **macOS**: You may need to remove the quarantine flag:
   ```
   xattr -rd com.apple.quarantine Skiptro skiptro-cli
   ```

## What's Included

| File | Description |
|------|-------------|
| `Skiptro` / `Skiptro.exe` | Desktop GUI app — configure libraries, scan, and view results |
| `skiptro-cli` / `skiptro-cli.exe` | Command-line tool — for automation and scheduled tasks |
| `ffmpeg/` | Bundled FFmpeg 8.0 |
| `intro_encoder.onnx` | ML model for intro verification |

## Desktop App

Launch `Skiptro` to open the GUI. From the desktop app you can:

- Add and manage media libraries
- Scan libraries for intros
- View detection results
- Export `.skiptro.json` files for Kodi

## CLI Quick Start

The CLI is ideal for automation, scheduled tasks, and headless servers.

```bash
# Scan a single season
skiptro-cli scan "/path/to/TV Show/Season 1"

# Force rescan (ignore cached results)
skiptro-cli scan "/path/to/TV Show/Season 1" --force

# Scan all configured libraries
skiptro-cli scan --all

# Scan all libraries and export skip files for Kodi
skiptro-cli scan --all --export

# Scan all and export both skip files and chapter files
skiptro-cli scan --all --export --chapters

# Export .skiptro.json files from existing scan results
skiptro-cli export

# Export ffmpeg chapter files (.skiptro.chapters)
skiptro-cli export-chapters

# Check what's been detected for a specific file
skiptro-cli lookup "/path/to/episode.mkv"
```

### Chapter Files

The `export-chapters` command writes `.skiptro.chapters` files in ffmpeg metadata format next to each video. These can be remuxed into your video files to add chapter markers:

```bash
ffmpeg -i episode.mkv -i episode.skiptro.chapters -map_metadata 1 -codec copy output.mkv
```

This creates three chapters: pre-intro, intro, and episode — allowing you to skip the intro using chapter navigation.

### Scheduled Scanning

Set up a cron job or Windows Task Scheduler to keep your skip files updated:

```bash
# Cron (Linux/macOS) — scan and export nightly at 3am
0 3 * * * /path/to/skiptro-cli scan --all --export
```

```
# Windows Task Scheduler — run skiptro-cli.exe scan --all --export
```

### CLI Commands

| Command | Description |
|---------|-------------|
| `scan <path>` | Scan a directory for TV episodes and detect intros |
| `scan --all` | Scan all configured libraries |
| `scan-files <files...>` | Scan specific episode files (min 2) |
| `export [path]` | Export .skiptro.json files for Kodi |
| `export-chapters [path]` | Export ffmpeg chapter files (.skiptro.chapters) |
| `status` | Show scan statistics |
| `libraries` | List configured media libraries |
| `lookup <file>` | Show detected segments for a file |

### CLI Options

| Option | Description |
|--------|-------------|
| `--force` | Force rescan everything (ignore cached results) |
| `--export` | Export after scan completes (use with `scan --all`) |
| `--chapters` | Also export ffmpeg chapter files (use with `--export`) |
| `--dry-run` | Scan without writing to database |
| `--rescan-no-intro` | Rescan seasons previously marked as having no intro |

## Windows Performance

Windows Defender real-time protection can slow scans by **10-30x** by scanning every file read FFmpeg makes on large video files.

To fix, run this once in PowerShell as admin:

```powershell
# Replace with your actual Skiptro install path
Add-MpPreference -ExclusionProcess "C:\path\to\Skiptro\ffmpeg\ffmpeg.exe"
```

The Desktop app shows the exact command with the correct path under Settings > Performance.

This only excludes file reads made by the bundled FFmpeg binary — your media files are still protected normally.

## Kodi Add-on

The companion **[service.skiptro](https://github.com/MikeSiLVO/service.skiptro)** Kodi add-on shows a skip button during playback when an intro is detected. See its [README](https://github.com/MikeSiLVO/service.skiptro/blob/main/README.md) for installation and details.

## License

See [THIRD_PARTY_LICENSES.txt](https://github.com/MikeSiLVO/skiptro-releases/releases/latest) included in each release for third-party license information.
