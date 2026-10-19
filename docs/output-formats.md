# Output Formats

Skiptro writes its results as files next to your media. Each format is optional and can be turned on in Settings or with the matching command-line flag. Times are in seconds unless a format says otherwise.

## Skip file (`.skiptro.json`)

One file per episode, named after the video: `Episode.mkv` gets `Episode.skiptro.json` in the same folder. This is what the companion Kodi add-on reads.

```json
{
  "intro": {
    "start": 356.5,
    "end": 374.7
  },
  "skiptro": {
    "confidence": 1,
    "version": "1.3.2"
  }
}
```

| Field | Meaning |
|---|---|
| `intro.start` | Where the intro starts, rounded to 0.1s |
| `intro.end` | Where the intro ends, rounded to 0.1s. Missing when no end could be found; treat the intro as having no known end |
| `skiptro.confidence` | 0 to 1. Higher means more episodes in the season agreed on this intro |
| `skiptro.version` | Skiptro version that wrote the file |

Only episodes with a detected intro get a file. A rescan rewrites the file in place. Fields may be added in later versions, so ignore any you do not recognize.

## Kodi EDL (`.edl`)

One file per episode, named after the video with the extension swapped: `Episode.mkv` gets `Episode.edl`. Kodi reads it without any add-on.

```
356.5	374.7	3
```

One line: start, end, and action `3`, tab-separated. Action `3` marks a commercial break, which Kodi skips or offers to skip depending on its own playback settings.

An existing `.edl` next to the video is overwritten.

## Chapter file (`.skiptro.chapters`)

One file per episode in ffmpeg metadata format: `Episode.mkv` gets `Episode.skiptro.chapters`. Times are in milliseconds.

```
;FFMETADATA1
[CHAPTER]
TIMEBASE=1/1000
START=0
END=356505
title=Episode
[CHAPTER]
TIMEBASE=1/1000
START=356505
END=374663
title=Intro
[CHAPTER]
TIMEBASE=1/1000
START=374663
END=2580000
title=Episode
```

The intro becomes a chapter titled `Intro`. If the video already has chapters, they are kept and split around the intro; otherwise the rest of the episode is covered by chapters titled `Episode`. The file does nothing on its own; see [Chapter Files](../README.md#chapter-files) for how to embed it.

## Theme audio (`theme-sNN.mp3`)

One MP3 per season holding the intro audio, for [tvtunes](https://github.com/latts9923/service.tvtunes). It is written to the show folder, `theme-s01.mp3` for season 1, or to the episode folder when episodes are not in season folders.

The `theme` prefix can be changed in Settings or with `--name <prefix>`. An existing file is kept unless "Overwrite existing theme files" is on in Settings or `--overwrite` is given.
