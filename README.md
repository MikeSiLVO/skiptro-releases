<p align="center">
  <h1 align="center">Skiptro</h1>
  <p align="center">
    Automatic TV show intro detection for Kodi
    <br />
    <a href="https://github.com/MikeSiLVO/skiptro-releases/releases/latest"><strong>Download Latest Release</strong></a>
    <br /><br />
    If you find Skiptro useful, <a href="https://ko-fi.com/mikesilvo">buy me a coffee</a> ☕
  </p>
</p>

---

Skiptro uses audio fingerprinting and machine learning to detect TV show intros across episodes in a season, then exports skip markers that Kodi can use to automatically skip them during playback.

You run Skiptro on your PC or NAS and manage it from a **web UI** in your browser. Each release also includes a **CLI** for automation and a **tray app** that runs the server in the background.

## Screenshots

The show grid, with each show's intro coverage at a glance:

![Skiptro show grid](https://raw.githubusercontent.com/MikeSiLVO/skiptro-releases/main/screenshots/web-ui-shows.png)

A show's detail, with each season's detected intro, timing, and export status:

![Skiptro show detail](https://raw.githubusercontent.com/MikeSiLVO/skiptro-releases/main/screenshots/show-detail.png)

Works from your phone too:

<img src="https://raw.githubusercontent.com/MikeSiLVO/skiptro-releases/main/screenshots/mobile-shows.jpg" width="320" alt="Skiptro on a phone">

## Features

- Browser-based UI to manage libraries, scan, match shows to TMDB, and review results
- Detects intros by comparing audio across episodes in a season
- Exports `.skiptro.json` files for the companion Kodi add-on
- EDL export for Kodi's built-in auto-skip
- Theme audio export (MP3) for [tvtunes](https://github.com/latts9923/service.tvtunes) integration
- Optional ffmpeg chapter file export for users who prefer to embed chapters
- Per-show exclusion for skipping specific shows
- Remote access from a phone or another device, with an optional API key
- Docker support for headless/NAS deployments
- Self-contained, no runtime or dependencies required
- Needs at least 2 episodes per season to work

## Supported Platforms

| Platform | Architecture | FFmpeg Included |
|----------|-------------|-----------------|
| Windows | x64 | Yes |
| Linux | x64 | Yes |
| Linux | ARM64 (Raspberry Pi) | Yes |
| macOS | x64 (Intel) | Yes |
| macOS | ARM64 (Apple Silicon) | Yes |

All releases include FFmpeg 8.0 bundled, no additional dependencies required.

## Installation

1. Download the archive for your platform from [Releases](https://github.com/MikeSiLVO/skiptro-releases/releases/latest)
2. Extract to a directory of your choice
3. **macOS**: You may need to remove the quarantine flag:
   ```
   xattr -rd com.apple.quarantine Skiptro-Desktop skiptro
   ```

## What's Included

| File | Description |
|------|-------------|
| `skiptro` / `skiptro.exe` | The Skiptro app: runs the web UI (`skiptro serve`) and the CLI |
| `Skiptro-Desktop` / `Skiptro-Desktop.exe` | Tray app that runs the server in the background and opens the web UI |
| `ffmpeg/` | Bundled FFmpeg 8.0 |
| `intro_encoder.onnx` | ML model for intro verification |
| `intro_encoder.onnx.data` | ML model weights data |

## Web UI

Skiptro is managed from your browser.

```bash
# Start the server, then open http://localhost:8585
skiptro serve
```

On Windows you can instead launch `Skiptro-Desktop.exe`, which runs the server in the tray and opens the web UI for you (with an option to start with Windows).

From the web UI you can:

- Add and manage media libraries with a built-in folder browser
- Scan libraries and watch jobs run in a live queue
- Match shows to TMDB for proper titles and posters (optional)
- Review detected intros per season and export `.skiptro.json` files for Kodi
- Schedule scans and turn on a file watcher for new episodes
- Allow remote access and reach Skiptro from a phone or another device

To reach it from another device, turn on remote access in Settings and open `http://<server-ip>:8585`. There is no login by default; set an API key (Settings, or the `SKIPTRO_API_KEY` environment variable) to require one. For access from outside your LAN, use a VPN or an authenticated reverse proxy. A QR code in Settings fills the key in on a phone.

## CLI Quick Start

The CLI is ideal for automation, scheduled tasks, and headless servers.

```bash
# Scan a single season
skiptro scan "/path/to/TV Show/Season 1"

# Force rescan (ignore cached results)
skiptro scan "/path/to/TV Show/Season 1" --force

# Scan all configured libraries
skiptro scan --all

# Scan all libraries and export skip files for Kodi
skiptro scan --all --export

# Scan all and export both skip files and chapter files
skiptro scan --all --export --chapters

# Scan all and export EDL files for Kodi auto-skip
skiptro scan --all --export --edl

# Export .skiptro.json files from existing scan results
skiptro export

# Export ffmpeg chapter files (.skiptro.chapters)
skiptro export-chapters

# Export Kodi EDL files (.edl)
skiptro export-edl

# Export intro audio as MP3 for tvtunes (one per season in show folder)
skiptro export-themes

# Manage libraries
skiptro library add "/path/to/TV Shows"
skiptro libraries

# Exclude a show from scanning
skiptro show exclude 1 "Show Name"

# Check what's been detected for a specific file
skiptro lookup "/path/to/episode.mkv"
```

### Chapter Files

The `export-chapters` command writes `.skiptro.chapters` files in ffmpeg metadata format next to each video. If the video already has chapters, they are preserved and merged with the intro chapter.

These can be embedded into your video files to add chapter markers:

```bash
# MKV files, edits in-place, no remux needed (requires mkvtoolnix)
mkvpropedit episode.mkv -c episode.skiptro.chapters

# Any format, remuxes into a new file (no re-encoding)
ffmpeg -i episode.mkv -i episode.skiptro.chapters -map 0 -map_chapters 1 -codec copy output.mkv
```

### Scheduled Scanning

Set up a cron job or Windows Task Scheduler to keep your skip files updated:

```bash
# Cron (Linux/macOS), scan and export nightly at 3am
0 3 * * * /path/to/skiptro scan --all --export
```

```
# Windows Task Scheduler, run skiptro.exe scan --all --export
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
| `export-themes [path]` | Export intro audio as MP3 for tvtunes |
| `status` | Show scan statistics |
| `libraries` | List configured media libraries |
| `library add <path>` | Add a media library |
| `library remove <id>` | Remove a library and its data |
| `library enable/disable <id>` | Enable or disable a library |
| `show list <id>` | List shows in a library |
| `show exclude <id> <name>` | Exclude a show from scanning |
| `show include <id> <name>` | Re-include an excluded show |
| `lookup <file>` | Show detected segments for a file |
| `cleanup` | Remove detections for files that no longer exist (deleted/moved episodes). `--dry-run` to preview |

### CLI Options

| Option | Description |
|--------|-------------|
| `--force` | Force rescan everything (ignore cached results) |
| `--export` | Export after scan completes (use with `scan --all`) |
| `--chapters` | Also export ffmpeg chapter files (use with `--export`) |
| `--edl` | Also export Kodi EDL files (use with `--export`) |
| `--themes` | Also export theme MP3s for tvtunes (use with `--export`) |
| `--name <prefix>` | Theme filename prefix (default: `theme`) |
| `--overwrite` | Overwrite existing theme files |
| `--dry-run` | Scan without writing to database |
| `--rescan-no-intro` | Rescan seasons previously marked as having no intro |
| `--diagnostics` | Write a JSON diagnostic report to logs (use with `scan-files`) |
| `--audio-stream N` | Select audio stream index (0-based) for multi-language files |

## Docker

A Docker image is available for headless/NAS deployments (Unraid, Synology, etc.). The container serves the web UI on port 8585.

```bash
docker run -d \
  --name skiptro \
  -p 8585:8585 \
  -v skiptro-data:/root/.config/Skiptro \
  -v /path/to/tv:/media/tv \
  -e MEDIA_PATHS=/media/tv \
  mikesilvo/skiptro:latest
```

Then open `http://<host-ip>:8585`. The web UI has no login by default; set `SKIPTRO_API_KEY` to require one, and use a VPN or an authenticated reverse proxy for access from outside your LAN.

### Docker Compose

```yaml
services:
  skiptro:
    image: mikesilvo/skiptro:latest
    container_name: skiptro
    ports:
      - "8585:8585"
    environment:
      - MEDIA_PATHS=/media/tv
      # Optional: require this key for access from other devices
      # - SKIPTRO_API_KEY=choose-a-long-random-string
    volumes:
      - skiptro-data:/root/.config/Skiptro
      - /path/to/tv:/media/tv
      # Add more media mounts as needed:
      # - /path/to/more-tv:/media/tv2
    restart: unless-stopped

volumes:
  skiptro-data:
```

Media libraries listed in `MEDIA_PATHS` (comma-separated) are auto-registered on container startup. Configure scheduled scans and the file watcher from the web UI under Settings. You can also run one-shot commands:

```bash
# Manual scan
docker exec skiptro /app/skiptro scan --all --export

# Remove detections for deleted/moved episodes
docker exec skiptro /app/skiptro cleanup
```

> Upgrading from a 0.x image: the old `SCAN_SCHEDULE` and `SCAN_ARGS` variables no longer do anything. Publish port 8585 to reach the web UI, and set up scheduled scans there.

## Windows Performance

Windows Defender real-time protection can slow scans by **10-30x** by scanning every file read FFmpeg makes on large video files.

To fix, run this once in PowerShell as admin:

```powershell
# Replace with your actual Skiptro install path
Add-MpPreference -ExclusionProcess "C:\path\to\Skiptro\ffmpeg\ffmpeg.exe"
```

The web UI shows the exact command with the correct path under Settings > Performance.

This only excludes file reads made by the bundled FFmpeg binary, so your media files are still protected normally.

## Kodi Add-on

The companion **[service.skiptro](https://github.com/MikeSiLVO/service.skiptro)** Kodi add-on shows a skip button during playback when an intro is detected. See its [README](https://github.com/MikeSiLVO/service.skiptro/blob/main/README.md) for installation and details.

## License

See [THIRD_PARTY_LICENSES.txt](https://github.com/MikeSiLVO/skiptro-releases/releases/latest) included in each release for third-party license information.
