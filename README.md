# albumdown

Download entire YouTube Music albums as MP3s with one command.

## Installation

Clone the repo and `cd` into it:

```bash
git clone https://github.com/Karataso/albumdown.git
cd albumdown
```

**Linux / macOS:**
```bash
./albumdown "https://music.youtube.com/playlist?list=PLxxxxx"
```

**Windows:**
```bash
python albumdown "https://music.youtube.com/playlist?list=PLxxxxx"
```

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
```

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
- Linux, macOS, and Windows are supported
