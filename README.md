# long-to-shorts

Turn one long video into 10 Shorts, for free, with Claude Code.

Give it any long video (a YouTube video, a podcast, a webinar). It finds the best moments, makes each one
vertical, adds captions, and saves them as Shorts you can post on YouTube, Instagram or TikTok.

You don't need an editing app.

## How to install

1. Open Claude Code.
2. Copy this whole message, paste it in, and press Enter:

```
Install the long-to-shorts skill for me from https://github.com/Ansub/long-to-shorts.
Clone it into ~/.claude/skills/long-to-shorts, read its README.md and SKILL.md, run the setup
for my computer, check it works, and tell me how to use it.
```

3. Wait for it to finish. The first time takes a while because it downloads a large file (about 3 GB).

## How to use it

Type this in Claude Code, with the path to your video:

```
/long-to-shorts turn ~/Movies/my-video.mp4 into 10 shorts
```

Claude shows you the moments it picked. Say yes (or tell it which ones to change), and it makes all the Shorts.
When it's done, it opens the folder with your Shorts and their titles.

## Good to know

- Works on Mac, Windows and Linux.
- Your video never leaves your computer.
- 4K videos give the sharpest Shorts.
- Want to edit a single clip from your phone into one Short instead? Use
  [claude-shorts](https://github.com/Ansub/claude-shorts).

Made by [Ansub](https://www.youtube.com/@ansubkhan). Free to use (MIT license).
