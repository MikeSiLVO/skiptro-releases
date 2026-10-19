# REST API

The web server (`skiptro serve`, the tray app, or the Docker image) has a JSON API under `/api`, on the same port as the web UI (default 8585). This page covers the part that is useful for automation: reading results and starting scans or exports.

The web UI uses more endpoints than are listed here. Those, and anything not on this page, may change in any release without notice.

## Access

Requests from the same machine are always allowed. Requests from another device need remote access turned on in Settings and, if an API key is set, the key in an `X-Api-Key` header or an `apikey` query parameter:

```
curl -H "X-Api-Key: <key>" http://<server-ip>:8585/api/status
```

A missing or wrong key returns `401`. See [Web UI](../README.md#web-ui) for setting up remote access and the key.

Responses are JSON with camelCase field names. Times are UTC in ISO 8601. Positions in media are in seconds. Confidence values run from 0 to 1, higher meaning more certain.

## Status

`GET /api/status`

```json
{
  "version": "1.3.2",
  "libraries": 15,
  "shows": 781,
  "seasons": 3257,
  "segments": 35949,
  "introEpisodes": 35949,
  "avgConfidence": 0.96,
  "exportedJsonEpisodes": 2826,
  "canRestart": true
}
```

| Field | Meaning |
|---|---|
| `version` | Skiptro version |
| `libraries`, `shows`, `seasons` | How many are known |
| `segments` | Detected segments of every type |
| `introEpisodes` | Episodes with a detected intro |
| `avgConfidence` | Average intro confidence, or `null` before the first detection |
| `exportedJsonEpisodes` | Episodes with a skip file written |
| `canRestart` | `false` in Docker, where the container manages restarts |

## Libraries

`GET /api/libraries` lists the configured libraries.

| Field | Meaning |
|---|---|
| `id` | Library id, used as a scan target |
| `name`, `path` | As entered when the library was added |
| `type`, `contentType`, `groupName` | Library kind and folder layout, as set in the web UI |
| `enabled` | Disabled libraries are skipped by a scan of `All` |
| `watchForChanges` | New files are scanned automatically |
| `lastScannedAt` | Time of the last scan, or `null` |

## Shows

`GET /api/shows` lists every show with its scan totals.

| Field | Meaning |
|---|---|
| `id` | Show id, used as a scan target and for the detail call |
| `title`, `sortTitle`, `year` | Show name and year |
| `tmdbId` | TMDB id, or `null` when the show is not matched |
| `matchSource` | `Pending`, `Auto`, `Manual`, `Unmatched` or `Ambiguous` |
| `posterPath` | Poster URL on this server, or `null` |
| `excluded` | Excluded shows are never scanned |
| `scannedSeasons`, `seasonsWithIntro`, `detectedEpisodes` | Scan totals |
| `avgConfidence` | Average across detected episodes, or `null` |
| `createdAt` | When the show was first found |
| `lastScannedAt` | Time of the last scan, or `null` |
| `folders` | Each library folder the show was found in: `id`, `libraryId`, `libraryName`, `folderPath`, `seasonCount`, `episodeCount`, and `hasIgnoreMarker` for a folder holding a `.skiptro.ignore` file |
| `signatureSeason`, `signatureConfidence` | The season and confidence of the stored intro signature, or `null` without one |

`GET /api/shows/{id}` returns the show with `overview`, `status` (from TMDB), `analysisWindowSeconds` (the show's search window, or `null` for the default) and a `seasons` list:

| Field | Meaning |
|---|---|
| `id` | Season id, used as a scan target and for the segments call |
| `season`, `seasonPath` | Season number and folder |
| `episodeCount`, `detectedEpisodes` | Episodes in the folder, and how many have an intro |
| `hasIntro` | Whether the season has an intro |
| `introConfidence`, `introDuration` | Season confidence and typical intro length, or `null` |
| `exportedJsonCount`, `exportedChaptersCount`, `exportedEdlCount`, `exportedThemesCount` | Output files written for the season |
| `scannedAt` | Time of the last scan |
| `userStatus` | `Unreviewed`, `Confirmed` or `Rejected` |
| `audioStreamCount`, `multiTrackEpisodeCount` | Audio tracks found, and episodes with more than one |
| `audioStreamOverride` | The audio track chosen for the season, or `null` for automatic |

## Detected intros

`GET /api/seasons/{id}/segments` lists the intros found in one season.

```json
[
  {
    "id": 25905,
    "filePath": "F:\\TV\\12 Monkeys\\Season 02\\12.Monkeys.S02E01.mkv",
    "season": 2,
    "episode": 1,
    "type": "Intro",
    "startSeconds": 356.5,
    "endSeconds": 374.66,
    "confidence": 1,
    "userStatus": "Unreviewed"
  }
]
```

| Field | Meaning |
|---|---|
| `id` | Segment id |
| `filePath` | The episode file |
| `season`, `episode` | Parsed from the file name, or `null` when it has none |
| `type` | `Intro` |
| `startSeconds`, `endSeconds` | Where the intro starts and ends |
| `confidence` | Detection confidence |
| `userStatus` | `Unreviewed`, `Confirmed` or `Rejected` |

These are the same values written to the [output files](output-formats.md).

## Jobs

Scans and exports run as jobs, one at a time. A job started while another runs waits in the queue.

`POST /api/jobs` starts a job and returns it.

```json
{ "type": "Scan", "target": "Season", "targetId": 2314, "force": false }
```

| Field | Values |
|---|---|
| `type` | `Scan` or `Export` |
| `target` | `All`, `Library`, `Show` or `Season` |
| `targetId` | The library, show or season id; omit for `All` |
| `force` | `true` rescans episodes that already have results |
| `export` | Export only, optional; see below |

`export` limits an export job. Every field is optional, and a missing format flag uses the setting from Settings.

```json
{
  "type": "Export", "target": "Show", "targetId": 616, "force": false,
  "export": { "seasonIds": [2314], "json": true, "chapters": false, "edl": true, "themes": false, "themeOverwrite": false }
}
```

An unknown `type` or `target` returns `400`; an unknown id returns `404`.

`GET /api/jobs` lists recent jobs, newest first.

| Field | Meaning |
|---|---|
| `id` | Job id |
| `type`, `target`, `targetId` | As sent |
| `description` | What the job does, as shown on the Activity page |
| `state` | `Queued`, `Running`, `Done`, `Failed` or `Cancelled` |
| `enqueuedAt`, `startedAt`, `finishedAt` | Times, `null` until reached |
| `error` | Failure message, or `null` |
| `summary` | For a finished scan: `showsScanned`, `showsUpToDate`, `showsNoEpisodes`, `seasonsDetected` and `introsFound`; otherwise `null` |

`POST /api/jobs/{id}/cancel` cancels a queued or running job. An unknown id returns `404`.

## Example: scan after a download

Scan the season a download landed in. With "Export automatically after each scan" on in Settings, the output files are written when the scan finishes.

```
curl -X POST -H "Content-Type: application/json" \
  -d '{"type":"Scan","target":"Season","targetId":2314,"force":false}' \
  http://localhost:8585/api/jobs
```

New files in a watched library are scanned without this. The call is for setups where the library is not watched or the files arrive on a share the watcher cannot see.
