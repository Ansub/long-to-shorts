# long-to-shorts

A free Claude Code skill that turns one long video into about 10 finished Shorts. Give it a YouTube video,
a podcast or a webinar, and it gives back vertical Shorts ready to post on YouTube, Instagram and TikTok:

- **Picks the moments** that work on their own: each one opens on a hook and ends on the payoff
- **Reframes to vertical**: your face for talking-head parts, screen on top and face below for screen recordings
- **Tightens** each clip by cutting the long pauses, only at silent points
- **Captions** word by word, with spelling and grammar fixed and brand logos
- **Music** from a file or a YouTube link, ducked under your voice (optional)
- **Loudness** normalised to -14 LUFS
- **Titles and descriptions** for every Short, drafted from what you actually say

You approve the list of picks once. It makes all of them and shows you one review sheet at the end.

No editing app. It runs on macOS, Windows and Linux with ffmpeg, whisper.cpp and Python. No API keys, nothing uploaded.

## Install with one prompt

Open Claude Code and paste this:

```
Install the long-to-shorts skill for me:
1. Clone https://github.com/Ansub/long-to-shorts into ~/.claude/skills/long-to-shorts
   (if it's already there, git pull instead). Install Git for my OS first if missing.
2. Read its README.md and SKILL.md so you know what it does.
3. Detect my OS: run setup.ps1 in native PowerShell on Windows (with -ExecutionPolicy Bypass),
   or `bash ~/.claude/skills/long-to-shorts/setup.sh` on macOS/Linux. Install prerequisites for me;
   no WSL or manual Homebrew setup. The setup downloads a 3 GB speech model, so let it finish.
4. Refresh PATH and run scripts/check_setup.py with the private venv Python (Scripts/python.exe on
   Windows, bin/python elsewhere). Follow the OS commands in SKILL.md.
5. Tell me in a few lines what got installed, whether I need to restart Claude Code to see /long-to-shorts,
   and the exact prompt to make Shorts from my first video.
```

## Install by hand

1. Clone it into your Claude Code skills folder:
   ```bash
   git clone https://github.com/Ansub/long-to-shorts ~/.claude/skills/long-to-shorts
   ```
2. Run the one-time setup (installs the tools and downloads a 3 GB speech model, or reuses it if you
   already have [claude-shorts](https://github.com/Ansub/claude-shorts)):
   ```bash
   bash ~/.claude/skills/long-to-shorts/setup.sh
   ```
   On Windows, use PowerShell instead:
   ```powershell
   powershell -NoProfile -ExecutionPolicy Bypass -File "$HOME/.claude/skills/long-to-shorts/setup.ps1"
   ```
3. Open Claude Code and type:
   ```
   /long-to-shorts turn ~/Movies/my-video.mp4 into 10 shorts
   ```

Claude transcribes the video, shows you the picks (time range, first line, why it works), and after you say
yes it builds every Short without stopping. On an Apple Silicon Mac a 10-minute 4K video takes about
15-20 minutes for 10 Shorts; Windows and Linux PCs without a GPU take longer.

## Fonts

Captions use **Inter** and **Great Vibes** (both free). To use your own, drop a `.ttf` in
`~/.config/long-to-shorts/fonts/` and set `{"fontText": "...", "fontScript": "..."}` in
`~/.config/long-to-shorts/settings.json`, which can also change sizes and colours (keys in
`assets/caption-settings.json`).

## Good to know

- Works best on a finished, edited video. Raw footage with retakes works, but the picks will need more cleanup.
- Output is 1080x1920 at your video's own frame rate. A 4K source gives the sharpest vertical crops.
- Works on macOS, Windows 10/11 x64 (ARM64 needs Windows 11), and Linux (Debian/Ubuntu, Fedora or Arch). Setup may ask for your OS password.
- Copyrighted music from YouTube can get your Shorts claimed or muted. Use royalty-free tracks for posting.
- Add names Whisper keeps misspelling to `~/.config/long-to-shorts/glossary.txt` (`Correct = wrong, wrong`).
- Editing one raw phone clip into one Short instead? Use [claude-shorts](https://github.com/Ansub/claude-shorts).

Made by [Ansub](https://www.youtube.com/@ansubkhan). MIT licensed.
