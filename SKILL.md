---
name: long-to-shorts
description: >-
  Turn one long video (a YouTube video, podcast, webinar, course lesson or livestream, usually landscape) into
  about 10 finished vertical Shorts / Reels / TikToks in one batch, with no editing app. Picks the moments that
  stand alone and open on a hook, snaps every edge to silence, reframes face-to-camera shots to 9:16 and
  screen recordings to screen-on-top / face-below, writes spell-checked word-by-word captions with brand logos,
  optional ducked music, -14 LUFS, titles and a review sheet. Runs locally on ffmpeg, whisper.cpp and Python.
  Use whenever someone hands over a long or landscape video and says "turn this into shorts", "clip this",
  "make 10 shorts from my video", "repurpose my podcast for TikTok", "cut reels out of this webinar", "find the
  best moments", or types /long-to-shorts.
---

# long-to-shorts

One finished long video in, about 10 upload-ready 1080x1920 Shorts out. Everything runs on the user's Mac,
Windows PC or Linux machine: ffmpeg, whisper.cpp and Python (Pillow). No editing app, no API keys, nothing uploaded.

The user approves **once**: the list of picks. After that, run the whole batch without stopping, check your own
work, and show one review sheet at the end. Ten Shorts with a question each is ten interruptions; removing them
is the point of this skill. Keep every message to a few lines.

If they give a single vertical phone clip to edit into one Short, this is the wrong tool: say so in one line
(the free `claude-shorts` skill does that: https://github.com/Ansub/claude-shorts) and stop.

## OS and paths (do this first)

Detect the OS from the session before running commands. Use **native PowerShell on Windows**, and bash on
macOS/Linux; never require WSL or zsh. Claude runs setup and installs prerequisites for the user. If an OS
password/UAC prompt appears, explain it briefly; do not ask them to install tools by hand.

- `$SK` = this skill's absolute folder; `$W` = a work folder in the session scratchpad or the OS temp directory
  (`[IO.Path]::GetTempPath()` on Windows, `${TMPDIR:-/tmp}` on macOS/Linux), with a subfolder per Short (`$W/<id>/`).
- `$PY` = `$HOME/.cache/long-to-shorts/venv/bin/python` on macOS/Linux, or
  `Join-Path $HOME '.cache\long-to-shorts\venv\Scripts\python.exe'` on Windows. Use this Python for **every** script.
- If the venv/tools are missing, run `bash "$SK/setup.sh"` on macOS/Linux, or
  `powershell.exe -NoProfile -ExecutionPolicy Bypass -File "$SK\setup.ps1"` on Windows. The one-time setup
  downloads a 3 GB speech model. Then run `scripts/check_setup.py` with `$PY` (Windows: `& $PY ...`).
- Windows setup changes the user PATH. In the current PowerShell session, refresh it after setup:
  `$env:PYTHONUTF8 = '1'; $env:Path = [Environment]::GetEnvironmentVariable('Path','User') + ';' + [Environment]::GetEnvironmentVariable('Path','Machine')`.
  On macOS/Linux add `export PATH="$HOME/.cache/long-to-shorts/bin:/opt/homebrew/bin:/usr/local/bin:/home/linuxbrew/.linuxbrew/bin:$PATH"`.
- Output `$OUT` = `<video's folder>/<video name> - shorts/` unless the user names a folder or
  `~/.config/long-to-shorts/profile.md` says otherwise. Read that file first if it exists. `~` means the user's home on all platforms; expand it before passing to tools.

The examples below use bash notation. **Translate them on Windows**:
`"$PY" "$SK/scripts/transcribe.py" <clip> "$W"` becomes `& $PY "$SK/scripts/transcribe.py" <clip> $W`;
`FPS=<fps> "$PY" ...` becomes `$env:FPS='<fps>'; & $PY ...`. Set variables with `$W = '...'`, use
`New-Item -ItemType Directory -Force` for folders, and quote every path (spaces and non-ASCII names are valid).
Use UTF-8 for all text/JSON files; in Windows PowerShell 5.1 use
`[IO.File]::WriteAllText($path, $text, (New-Object Text.UTF8Encoding($false)))` to avoid UTF-16/BOM output.
Fonts are rendered with Pillow, so no font path enters an ffmpeg filter. If adding a drawtext filter, escape
its drive-letter colon and filter metacharacters separately from shell quoting.

## 1. Probe and transcribe the whole video

```bash
ffprobe -v error -show_entries stream=codec_type,width,height,r_frame_rate -show_entries format=duration -of compact <video>
"$PY" "$SK/scripts/transcribe.py" <video> "$W"
```
Note the frame rate; it is `--fps` / `FPS` for every Short. Transcribing is roughly 1 minute per 10 minutes of video;
say so if the video is long. Read `$W/transcript.md` in full before picking anything.

## 2. Pick the Shorts

Default to 10, or the number they asked for. If the video only has 6 strong moments, pick 6 and say why:
a weak Short posted to hit a number does more harm than a missing one.

What makes a good pick, in order of weight:

1. **It stands alone.** Someone who never saw the long video gets it. No "as I said earlier", no "this one"
   pointing at something from three minutes ago, no setup that lives outside the clip.
2. **The first sentence is a hook.** A claim, a number, a surprise, a question, a "the mistake everyone makes".
   It does not have to be where a section starts: starting on the strongest line is the point. A Short that opens
   with "So, next," loses the viewer in the first second.
3. **It pays off.** It ends on the answer, the result, or the punchline, not mid-explanation.
4. **Length 30-90 s** (up to 3 min is allowed, rarely worth it). A complete idea beats a tight one; never cut
   the substance just to hit a length. A cold open that lands in 20 s is fine; padding it with setup talk is not.
5. **Variety.** Different topics across the batch. Avoid two Shorts making the same point.
6. **Screen-heavy moments are fine** (they become a split layout), as long as the screen part is readable at
   phone size: a terminal line, a result, a before/after. A dense settings page is not.

Boundaries: rough times are enough here. Picks never overlap. A clip may skip a weak sentence in the middle later
(step 4), but pick contiguous ranges here. Finished exports have long chunks (few pauses), so the right edge is
often inside one; read word times from `$W/transcript.json` when `transcript.md` is too coarse.

Write `$W/clips.json` (format in the docstring of `$SK/scripts/reframe.py`): `source`, empty `layouts`, and for
each pick `id` (`s01`...), `start`, `end`, `hook` (its first line, cleaned), `title` (a working title). Then snap
every edge to the silence between words:
```bash
"$PY" "$SK/scripts/edges.py" "$W" "$W/clips.json"
```
It prints each pick's new range with its first and last words. Check the first words are the hook and the last
words are the payoff, not the start of the next sentence ("... OpenAI. So"). A `WARN edge not silent` means the
words run together there: move that edge to the next sentence break and run it again.

Show the picks as one table: # | time range | length | first line | why it works (a few words). Then ask
**"Make these? Reply with numbers to drop or swap, and add a music file or YouTube link if you want music
(no music by default)."** This is the one approval. If they change picks, update `clips.json` and go on; no
second round of approval unless they ask for it.

## 3. Layouts (no questions, check yourself)

```bash
"$PY" "$SK/scripts/shots.py" "$W/clips.json" "$W"
```
It finds the framing changes in each pick (hard cuts and crossfades) and writes `$W/shots/<id>.jpg` (one frame
per shot, with a 10% grid) and `$W/shots/<id>_timeline.jpg` (a frame every 2 s). Look at both for every pick.
Expect a few extra shots where a card animates or the screen changes a lot: give them the same layout as their
neighbour. If the timeline shows a change still missed, split that shot in `clips.json` (copy the shot, set the
first one's `end` and the second one's `start`). Most long videos use 2-3 recurring framings, so define them once in `layouts`:

- **Face to camera** -> `{"type": "face", "cx": <face centre x>, "cy": <face centre y>}`. Read cx from the grid;
  people are rarely at exactly 0.5. `zoom` 1.1-1.3 tightens a wide shot.
- **Screen recording with a face-cam bubble** -> `{"type": "split", "screen": [x, y, w, h], "face": [x, y, w, h]}`.
  Screen box: the part that matters (the text being typed, the result), not the whole desktop; a full 16:9 desktop
  shrunk to phone width is unreadable. Face box: just inside the bubble, so its edges never show.
- **Screen with no face** -> `{"type": "face", ...}` centred on the important part of the screen (a 9:16 slice of
  it); the name only means "a single 9:16 window".
- **Already vertical, or a wide title card / motion graphic** -> `{"type": "full"}` (scaled to fit, black bars;
  any 9:16 slice of a wide card cuts its text).

Fill every shot's `layout` in `clips.json` (a shot `null` means you have not decided it yet; reframe refuses it).
If one pick needs a different screen box (the important text sits elsewhere), add a second layout such as
`demo_top` rather than compromising the shared one.

Then render stills for every pick and look at all of them:
```bash
"$PY" "$SK/scripts/reframe.py" "$W/clips.json" <id> "$W/<id>" --stills    # -> "$W/<id>/stills/shot_<k>.jpg"
```
Check: face not cut at the forehead or chin, eyes in the upper half of the face panel, screen text readable,
no bubble edges or black bars. Adjust the boxes and re-render until every still is right.

## 4. Build each Short

For each pick, in order, with no questions in between:

```bash
"$PY" "$SK/scripts/reframe.py" "$W/clips.json" <id> "$W/<id>"      # -> source.mov (1080x1920) + bands.json
"$PY" "$SK/scripts/transcribe.py" "$W/<id>/source.mov" "$W/<id>"
"$PY" "$SK/scripts/plan_cut.py" "$W/<id>" all --fps <fps>          # or a chunk list to drop a weak line
"$PY" "$SK/scripts/render_cut.py" "$W/<id>/source.mov" "$W/<id>"
"$PY" "$SK/scripts/words_on_cut.py" "$W/<id>" --fps <fps>
```
- `all` keeps every spoken chunk and only removes gaps of 0.5 s or more: a finished export has no retakes. Pass a
  chunk list instead when the opening line is weak or a sentence leans on the rest of the video.
- `plan_cut` must report 0 silence warnings. If an edge warns, the pick's boundary cuts a word: fix it in
  `clips.json`, rerun `edges.py`, and reframe again. `words_on_cut` drops words Whisper stretched over seconds
  of silence (it says so); they would otherwise flash on screen at the wrong moment.

Captions: write `$W/<id>/blocks.txt` by hand from `words_cut.json` (see "Caption blocks" below). Use their saved
fonts (`~/.config/long-to-shorts/settings.json`) or the defaults; don't ask about fonts during a batch. `reframe.py` wrote `$W/<id>/bands.json`, so every block sits in the band of the shot it starts in:
lower third on face shots, just above the seam with a dark backdrop on split shots, the top on full-frame shots.
Keep a block inside one shot (start a new block at a framing change) so it never sits in the wrong band.
```bash
FPS=<fps> "$PY" "$SK/scripts/captions.py" "$W/<id>" list
FPS=<fps> "$PY" "$SK/scripts/captions.py" "$W/<id>" qa
```
Look at `qa.jpg` for every Short: captions over a face, over the important screen text, cut off, or colliding mean
you change `blocks.txt` or the band (add `"captions": {"cy": ..., "safeTop": ..., "safeBot": ..., "backdropOpacity": ...}`
to that layout, then `reframe.py ... --bands-only`, which rewrites bands.json without re-encoding) and run qa
again. A change to shot boundaries or boxes is different: rerun the whole chain for that Short (reframe,
transcribe, plan_cut, render_cut, words_on_cut), since the cut is built from source.mov. The band is only 260 px tall: a block of three small lines
will not fit (captions.py warns), so split it. Then `"$PY" "$SK/scripts/captions.py" "$W/<id>" full "$W/<id>/captions.mov" <frames of cut.mov>`.

Music, if they gave one: fetch it once (`"$PY" "$SK/scripts/get_music.py" "<url>" "$W/music"`), then `"$PY" "$SK/scripts/music_bed.py" <file> "$W/<id>"` per Short. Use
`--start` to vary where each Short enters the track so ten Shorts do not all open on the same bar.

Export: `"$PY" "$SK/scripts/compose.py" "$W/<id>" "$OUT/<NN> - <title>.mp4"` (`--no-music` if none). Check frame
count (= frames of `cut.mov`) and 1080x1920 with `ffprobe`, and grab one frame to see the captions are burned in.

If the machine allows parallel work (subagents), Shorts are independent after step 3 and can be built side by
side; keep each one's files inside its own `$W/<id>/`.

## 5. Titles

For each Short: a title under 60 characters built from what is actually said in it (no claims the clip does not
make), plus a 2-3 line description with 3-5 hashtags. Write them all to `$OUT/titles.md`, one section per file,
and make sure each file name matches its final title (strip `<>:"/\\|?*`, control characters, `…` and emoji; trim trailing dots/spaces; avoid Windows reserved
names such as CON, NUL and COM1).

## 6. Review

```bash
"$PY" "$SK/scripts/review_sheet.py" "$OUT"     # -> $OUT/review.jpg, three frames from every Short
```
Look at it yourself first. Then report in a few lines: how many Shorts, total and per-Short length, anything you
dropped or could not fix, and the folder (macOS `open "$OUT"`, Windows `explorer.exe "$OUT"`, Linux `xdg-open "$OUT"`; skip on headless machines). Show `review.jpg`. Ask **"Any to redo?"** A redo is
one pick: change its range, layout or blocks and rebuild only that one.

## Caption blocks

`blocks.txt` format (see `$SK/examples/blocks.txt`), one line per caption line, a blank line between blocks:
```
style | display text (the CLEAN wording) | number of spoken words it covers | optional logo
```
- `s` small white words, `h` bold hero (names: products, models, people), `c` script hero (the emotional or
  payoff word: Sucks, Slow, Life). **One hero per block**, 1-3 lines, in spoken order, one block per sentence
  or clause.
- Display text is where spelling and grammar get fixed (check `$SK/assets/glossary.txt` and
  `~/.config/long-to-shorts/glossary.txt`: Whisper hears "chatGBD", "Grogbot", "cloud code"). Fix, never rewrite.
  The word count keeps it in sync with the audio: count spoken words exactly as they appear in `words_cut.json`,
  including ones the clean text drops. `captions.py list` must consume every word.
- Logos: `assets/logos/<name>.svg` (openai, claude, grok bundled). For another brand, download its SVG from
  `https://svgl.app` into `$W/<id>/logos/<name>.svg`. Use a logo only when the line names that product.
- Hero lines shrink to fit the band; small words never shrink (they become unreadable), so split long blocks.

## Rules that came from real runs

- Never cut inside a word; every cut point must be silent (plan_cut checks).
- New caption render = new look at the QA sheet. Don't show the user a render you haven't looked at.

- Long-form framing puts the head near the top of a 9:16 crop, so face-layout captions go in the lower third
  (y 1140-1440), not above the head as in phone clips. reframe.py sets this; don't move it back up.
- In split layouts, captions sit above the seam (y 690-940, ending 20 px above it so a hero line never lands on
  the head) on a 75% black backdrop, because white and blue words over a white board or a chat window were
  unreadable with less. Still pick a screen box whose bottom edge is
  empty-ish, so the backdrop hides nothing important.
- A face-cam bubble is often about 1.8:1; trimming it to the 1.125 face panel upscales it about 1.8x. Fine from a
  4K source, soft from 1080p: there, prefer a taller `top` (screen panel) so the face panel is smaller.
- Shot detection decodes the whole 4K file twice at low res (about 1 min for a 5-minute video), once; it is
  cached in `$W/scenes.json`. Heavily edited videos (crossfades everywhere) give many shots per pick; most share
  2-3 layouts, so labelling is quick.
- Never touch the source video; everything new goes in `$W` and `$OUT`.
- `edges.py` trusts Whisper's word times. When Whisper stretches a word into the next one, the snap can land on
  sound (it warns). Then set that edge by hand at a quiet point and do not rerun edges.py on it.
- A close-up face shot puts the mouth where the lower-third band is: give that layout its own lower band
  (`"captions": {"cy": 1470, "safeTop": 1380, "safeBot": 1560}`) and check qa.
- "No music" means no added music. Music baked into the source export stays under the voice.
