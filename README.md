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

## Features

- Audio fingerprint matching across episodes in a season
- FFT-based correlation validation
- Silence and energy-based boundary snapping for precise intro end detection
- Optional ML verification for shows with long intros
- Exports `.skiptro.json` files for the Kodi service addon
- Self-contained binaries — no .NET runtime required

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
   xattr -rd com.apple.quarantine skiptro
   ```

## Quick Start

```bash
# Scan a single season
skiptro scan "/path/to/TV Show/Season 1"

# Force rescan (ignore cached results)
skiptro scan "/path/to/TV Show/Season 1" --force

# Scan with ML verification (for long intros)
skiptro scan "/path/to/TV Show/Season 1" --ml

# Scan all configured libraries
skiptro scan --all

# Scan all libraries and export skip files for Kodi
skiptro scan --all --export

# Export .skiptro.json files from existing scan results
skiptro export

# Check what's been detected for a specific file
skiptro lookup "/path/to/episode.mkv"
```

## How It Works

1. **Fingerprinting** — Compares audio fingerprints across episodes in a season to find the common intro segment
2. **Correlation** — Validates matches using FFT-based cross-correlation
3. **Boundary Snapping** — Refines the intro end point by detecting silence or energy drops
4. **ML Verification** *(optional)* — For shows with long intros (60s+), uses a trained neural network to verify and refine boundaries
5. **Export** — Writes `.skiptro.json` files next to each episode for the companion Kodi addon

## Requirements

- At least 2 episodes per season (fingerprinting needs episodes to compare)
- FFmpeg is bundled with all platform releases

## Kodi Addon

The companion **[service.skiptro](https://github.com/MikeSiLVO/service.skiptro)** Kodi addon displays a skip button when an intro is detected during playback.

### How it works

1. Skiptro CLI scans your library and exports `.skiptro.json` files next to each episode
2. The Kodi addon detects these files during playback
3. When playback enters an intro region, a skip button appears on screen
4. The button auto-dismisses after a configurable timeout (default 10 seconds)

### Addon installation

1. Download the latest `service.skiptro` zip from [service.skiptro releases](https://github.com/MikeSiLVO/service.skiptro/releases)
2. In Kodi: Settings > Add-ons > Install from zip file
3. Select the downloaded zip

### Skip data format

Skiptro writes a `.skiptro.json` file next to each video file:

```json
{
  "intro": { "start": 0, "end": 87.5 }
}
```

## Commands

| Command | Description |
|---------|-------------|
| `scan <path>` | Scan a directory for TV episodes and detect intros |
| `scan --all` | Scan all configured libraries |
| `scan-files <files...>` | Scan specific episode files (min 2) |
| `export [path]` | Export .skiptro.json files for Kodi |
| `status` | Show scan statistics |
| `libraries` | List configured media libraries |
| `lookup <file>` | Show detected segments for a file |

## Options

| Option | Description |
|--------|-------------|
| `--force` | Force rescan everything (ignore cached results) |
| `--ml` | Enable ML verification for improved accuracy |
| `--export` | Export after scan completes (use with `scan --all`) |
| `--dry-run` | Scan without writing to database |
| `--rescan-no-intro` | Rescan seasons previously marked as having no intro |

## License

See [THIRD_PARTY_LICENSES.txt](https://github.com/MikeSiLVO/skiptro-releases/releases/latest) included in each release for third-party license information.
