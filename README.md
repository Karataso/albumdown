# albumdown

Download entire YouTube Music albums as MP3s with one command — or hand it an
**artist** and grab the whole discography, release by release.

## Installation

Clone the repo and `cd` into it:

```bash
git clone https://github.com/Karataso/albumdown.git
cd albumdown
```

**Linux:**
```bash
./albumdown "https://music.youtube.com/playlist?list=PLxxxxx"
```

**Windows:**
```bash
python albumdown "https://music.youtube.com/playlist?list=PLxxxxx"
```

**macOS:**
```bash
./albumdown "https://music.youtube.com/playlist?list=PLxxxxx"
```
> *Should work — same as Linux — but hasn't been tested on a real Mac.*

That's it. First run automatically installs dependencies into a local `.venv/` — nothing global gets touched.

> **Note:** ffmpeg must be installed (used for MP3 conversion).
> - Arch: `sudo pacman -S ffmpeg`
> - Debian/Ubuntu: `sudo apt install ffmpeg`
> - Windows: [ffmpeg.org/download](https://ffmpeg.org/download.html)

## Usage

```bash
# Basic — downloads to ~/Music/
./albumdown "https://music.youtube.com/playlist?list=PLxxxxx"

# Custom output directory
./albumdown "https://music.youtube.com/playlist?list=PLxxxxx" -o ~/Downloads

# Skip the confirmation prompt
./albumdown "https://music.youtube.com/playlist?list=PLxxxxx" --no-confirm

# Set MP3 quality (0=best, 10=worst, default: 0)
./albumdown "https://music.youtube.com/playlist?list=PLxxxxx" --quality 3

# Full artist — pick albums from their discography
./albumdown "https://music.youtube.com/channel/UCxxxxx"
./albumdown "https://music.youtube.com/@artisthandle"
./albumdown "@artisthandle"

# Whole discography, no questions asked
./albumdown "@artisthandle" --all
```

## Full artist mode

Give albumdown an **artist page** instead of a playlist and it reads the entire
discography straight from YouTube Music, then lets you choose what to grab:

```
🎤 The National — full discography
   13 albums • 16 singles & EPs • 13 shown

     #   Type      Year   Release
    01   Album     2023   Laugh Track
    02   Album     2023   First Two Pages of Frankenstein
    03   Album     2019   I Am Easy to Find
    …

Which releases? (e.g. 1-4, 7, all · Enter = all · q = quit) (all):
```

Whatever you pick gets downloaded one release at a time into the same
`Artist/Album/` layout, following the same rules as single-album mode: existing
folders are reused (and their capitalization adopted), files already on disk are
skipped, and cover art, lyrics and ID3 tags are handled per album. A release
that can't be read is reported and skipped — the rest of the discography keeps
going.

- **Albums only by default.** Add `--include-singles` to also offer EPs and singles.
- **`--all`** downloads everything in scope without the picker or the confirmation — handy for scripts.
- Album names and years come from the artist page itself, and each release is fetched in a *single* metadata request instead of one per track, so a 15-album discography doesn't hammer YouTube before the first download even starts.
- Album and single links come from the artist page's "Albums" and "Singles & EPs" shelves (including their "see all" pages, so nothing past the first few is missed).
- Channels that aren't artists — a vlog channel, a fan page — are detected and reported instead of downloading random playlists.

## What it does

1. You give it a YouTube Music (or YouTube) playlist URL
2. It fetches the tracklist and shows you what's in it
3. You confirm
4. It downloads every track as MP3 with embedded metadata into:
   ```
   ~/Music/
   └── Artist Name/
       └── Album Name/
           ├── 01. First Song.mp3
           ├── 02. Second Song.mp3
           └── ...
   ```

Each MP3 file has ID3 tags embedded: title, artist, album, and track number.

With an **artist URL** the flow is the same, repeated per release: albumdown
lists everything on the artist's page, you pick, and it walks through them one
album at a time — same folder layout, same tags, same skip-if-present rules.

Track numbers come from the playlist order (YouTube Music stores albums as playlists — playlist position *is* the track number), and the folder/artist/album names come from each song's own metadata tags, so names are clean (e.g. `Lámina Once` instead of the raw playlist title `Album - Lámina Once`).

## Options

| Flag | Description | Default |
|------|-------------|---------|
| `-o, --output DIR` | Output directory | `~/Music` |
| `--quality N` | MP3 quality (0=best, 10=worst) | `0` |
| `--no-confirm` | Skip the yes/no prompt | off |
| `--all` | Artist mode: download every release, no prompts | off |
| `--include-singles` | Artist mode: offer singles & EPs too | off |
| `--safe` | Slower, gentler downloads (see below) | off |
| `--cookies FILE` | Use a cookies.txt for authenticated downloads | none |
| `--lyrics` | Fetch and archive lyrics (see below) | off |
| `--no-art` | Skip embedding album cover art | off |
| `-h, --help` | Show help | — |

## Cover Art

On by default. The **album's own cover** (from the playlist) is downloaded once and embedded into every MP3 — one clean, consistent cover for the whole album instead of screwey per-video thumbnails with text overlays. Per-video thumbnails are only used as a last resort if the album has no art.

Disable with `--no-art`. Works as a backfill too: run it on an already-downloaded album and it only adds art to files missing it.

## Lyrics

```bash
./albumdown "URL" --lyrics
```

Fetches lyrics from [LRCLIB](https://lrclib.net) (free, no API key) and archives them two ways:

- `01. Song Title.lrc` — sidecar file with synced timestamps (read by mpv, foobar2000, most Android players)
- Lyrics embedded in the MP3's ID3 tags (shows in any player's lyrics view)

Matching is duration- and artist-aware to avoid attaching wrong-song lyrics. You can also run it on an album you've already downloaded — it backfills missing lyrics without re-downloading anything.

> Why not Musixmatch? Their free API tier only returns 30% lyric snippets; full lyrics require a paid license. LRCLIB is free and complete.

### Players that show the lyrics

- **Quod Libet** (`quodlibet`) — enable the *Display lyrics* plugin: shows embedded lyrics in a panel next to the song
- **Strawberry** — shows embedded plain lyrics
- **mpv + [mpv-lrc](https://github.com/guidocella/mpv-lrc)** — displays the `.lrc` files as a synced overlay that follows playback

## 403s and retries

YouTube sometimes throws a 403 — or a "Sign in to confirm you're not a bot" — at a single track. Failures are sorted into three kinds, each with its own policy:

| What happened | What albumdown does |
| --- | --- |
| **Blocked** (403, bot check, 429) | Waits 30s → 60s → 120s and retries that track. If it clears, the run carries on and the remaining tracks pace themselves slower (10s apart instead of 1.5s, and a longer gap between albums in artist mode) |
| **Gone** (private, deleted, geo-blocked) | Fails the track immediately — no point waiting — and moves on |
| **Network** (timeouts, connection resets, 5xx) | Short backoff: 2s → 4s → … capped at 30s, six tries |

A track only counts as downloaded once its MP3 is actually on disk, so the summary never claims success for a file that doesn't exist. Whatever still failed is listed at the end **with the reason** — just re-run the same command to retry only those, because everything already downloaded is skipped.

`--safe` is the pre-emptive version of this: it slows everything down before YouTube gets a chance to complain.

## Safe Mode

If you're getting "Sign in to confirm you're not a bot" or 403 errors, YouTube is flagging your requests. Use `--safe`:

```bash
./albumdown "URL" --safe
```

This adds aggressive delays:
- 3 seconds between metadata requests
- Random 5–15 second pause before each download
- Random 6–12 second wait between tracks (with a visible countdown)
- Random 6–12 second wait between albums in artist mode
- Fewer retries (fewer requests = less suspicion)

Slower, but much less likely to trip bot detection.

## Cookies (authenticated downloads)

If bot detection persists, you can pass YouTube cookies:

```bash
./albumdown "URL" --cookies cookies.txt
```

> ⚠️ **Warning:** Cookies tie downloads to a Google account. YouTube can temporarily restrict accounts that download heavily. Use a **throwaway account**, never your main one. Export cookies from a private/incognito window using a cookies.txt extension, then close the window.

## Notes

- Files already downloaded are skipped (safe to re-run)
- Failed tracks are reported at the end with a reason — re-running retries just those (see [403s and retries](#403s-and-retries))
- Works with both `youtube.com` and `music.youtube.com` playlist URLs, plus artist pages (channel URLs and `@handles`, `music.youtube.com` or `youtube.com`)
- In artist mode, releases that can't be read are reported at the end while the rest keep downloading
- Artist names are cleaned automatically (removes "- Topic", "- Album" suffixes)
- Artist name capitalization is unified — YouTube's catalog mixes spellings ("El Cuarteto De Nos" vs "...de Nos"), so existing case-variant folders are merged and the tags follow, keeping players from splitting one artist into two
- Linux and Windows are supported (tested)
- macOS should work but is untested
