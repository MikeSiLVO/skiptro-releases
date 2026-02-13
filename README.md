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
- EDL export for Kodi's built-in auto-skip
- Optional ffmpeg chapter file export for users who prefer to embed chapters
- Per-show exclusion for skipping specific shows
- Docker support for headless/NAS deployments
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

# Scan all and export EDL files for Kodi auto-skip
skiptro-cli scan --all --export --edl

# Export .skiptro.json files from existing scan results
skiptro-cli export

# Export ffmpeg chapter files (.skiptro.chapters)
skiptro-cli export-chapters

# Export Kodi EDL files (.edl)
skiptro-cli export-edl

# Manage libraries
skiptro-cli library add "/path/to/TV Shows"
skiptro-cli libraries

# Exclude a show from scanning
skiptro-cli show exclude 1 "Show Name"

# Check what's been detected for a specific file
skiptro-cli lookup "/path/to/episode.mkv"
```

### Chapter Files

The `export-chapters` command writes `.skiptro.chapters` files in ffmpeg metadata format next to each video. If the video already has chapters, they are preserved and merged with the intro chapter.

These can be remuxed into your video files to add chapter markers:

```bash
ffmpeg -i episode.mkv -i episode.skiptro.chapters -map 0 -map_chapters 1 -codec copy output.mkv
```

No re-encoding is performed — only the container metadata is updated.

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
| `export-edl [path]` | Export Kodi EDL files for auto-skip (.edl) |
| `status` | Show scan statistics |
| `libraries` | List configured media libraries |
| `library add <path>` | Add a media library |
| `library remove <id>` | Remove a library and its data |
| `library enable/disable <id>` | Enable or disable a library |
| `show list <id>` | List shows in a library |
| `show exclude <id> <name>` | Exclude a show from scanning |
| `show include <id> <name>` | Re-include an excluded show |
| `lookup <file>` | Show detected segments for a file |

### CLI Options

| Option | Description |
|--------|-------------|
| `--force` | Force rescan everything (ignore cached results) |
| `--export` | Export after scan completes (use with `scan --all`) |
| `--chapters` | Also export ffmpeg chapter files (use with `--export`) |
| `--edl` | Also export Kodi EDL files (use with `--export`) |
| `--dry-run` | Scan without writing to database |
| `--rescan-no-intro` | Rescan seasons previously marked as having no intro |

## Docker

A Docker image is available for headless/NAS deployments (Unraid, Synology, etc.).

```bash
docker run -d \
  --name skiptro \
  -v skiptro-data:/app/data \
  -v /path/to/tv:/media/tv:ro \
  -e SCAN_SCHEDULE="0 3 * * *" \
  mikesilvo/skiptro:latest
```

By default, the container runs a cron job to scan all configured libraries on schedule. You can also run one-shot commands:

```bash
# Add a library
docker exec skiptro /app/skiptro-cli library add /media/tv

# Manual scan
docker exec skiptro /app/skiptro-cli scan --all --export
```

See `docker/docker-compose.yml` for a full example.

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
