# albumdown

Download entire YouTube Music albums as MP3s with one command.

## Requirements

- Python 3.10+
- ffmpeg

That's it. Dependencies are installed automatically on first run.

## Usage

```bash
cd ~/Documents/albumProject

# Basic — downloads to ~/Music/
./albumdown "https://music.youtube.com/playlist?list=PLxxxxx"

# Custom output directory
./albumdown "https://music.youtube.com/playlist?list=PLxxxxx" -o ~/Downloads

# Skip the confirmation prompt
./albumdown "https://music.youtube.com/playlist?list=PLxxxxx" --no-confirm

# Set MP3 quality (0=best, 10=worst, default: 0)
./albumdown "https://music.youtube.com/playlist?list=PLxxxxx" --quality 3
```

First run creates a `.venv/` and installs `yt-dlp`, `rich`, and `mutagen` automatically.

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

## Options

| Flag | Description | Default |
|------|-------------|---------|
| `-o, --output DIR` | Output directory | `~/Music` |
| `--quality N` | MP3 quality (0=best, 10=worst) | `0` |
| `--no-confirm` | Skip the yes/no prompt | off |
| `-h, --help` | Show help | — |

## Notes

- Files already downloaded are skipped (safe to re-run)
- Failed tracks are reported at the end
- Works with both `youtube.com` and `music.youtube.com` playlist URLs
- Artist names are cleaned automatically (removes "- Topic", "- Album" suffixes)
