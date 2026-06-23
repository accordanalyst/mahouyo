# ✩ Mahou Amp ✩

I built this because I wanted a Winamp-style player on my GitHub that was reminscent of fun, easier times and had at least the essence of the magical-girl aesthetic. It comes in two versions: a static image you can drop straight into a README, and a fully working interactive player that actually plays music. Still surprising to me.

![Mahou Amp preview](./mahou-amp-player.svg)

---

## What's in here

| File | What it does |
|---|---|
| `mahou-amp-player.svg` | The static version. This is what shows up if you drop it into a README — GitHub doesn't let Markdown run JavaScript, so this is just a picture of the player. |
| `mahou_amp_player.html` | The real one. Buttons work, playlist scrolls, music actually plays. Open it in a browser, or host it somewhere and link to it. |

**Why two files?** GitHub READMEs can only show static stuff — images, not live scripts. So the SVG is what people actually *see* in the README, and the HTML file is where the real functionality lives. I just can't make the interactive one run embedded directly inside the README itself — that's a GitHub limitation, not something I can code around.

---

## What it does

- Pastel gradient shell, twinkly star decorations, the whole vibe
- Scrolling "now playing" marquee in the little LCD screen
- A 10-bar equalizer that animates while something's playing and freezes when it's paused
- A spinning sparkle disc, same deal — only spins while playing
- Click any track in the playlist to jump straight to it
- The playlist actually scrolls now (mouse wheel or click-and-drag) since I've got more tracks than fit on screen at once — there's a little pink scrollbar thumb on the side so you can see where you are
- Seek bar and volume slider you can actually drag
- It plays real songs. I wired it up to YouTube under the hood, so when you hit play, you're hearing the actual track, not a fake animation pretending to be one

---

## Quick heads up about the track times

Funny story — the static SVG version has made-up durations. I just typed numbers in because it's a flat image and has no way to know how long a song actually is.

The interactive version pulls real durations straight from YouTube, which meant dealing with a quirk: YouTube only tells you how long a video is *after* it's been loaded at least once. So if I'd done the lazy version, you'd see the real time for whatever's currently playing and `--:--` for every other track until you clicked into it. That felt like a downgrade from the static version where all 9 times are just sitting there.

So instead, the second the player's ready, it quietly loads every track in the background — no sound, nothing visible happening — just enough to ask YouTube "how long is this one?" for all nine, then snaps back to whatever track I actually had selected. End result: all the times show up within a second or two, same as the static version felt, except now they're real instead of made up.

---

## Using the static version (like in a README)

1. Drop `mahou-amp-player.svg` into your repo.
2. Reference it like any other image in your `README.md`:

```markdown
<p align="center">
  <img src="./mahou-amp-player.svg" alt="Mahou Amp" width="320" />
</p>
```

That's it. It's just an image at that point, so it'll show up anywhere images normally render — READMEs, wikis, you name it.

> Heads up: since it's a static image, nothing's actually clickable here. The buttons and scrolling are just for show. For the real thing, keep reading.

---

## Using the interactive version

**Heads up before you do anything else:** this player streams real audio from YouTube, and that specifically does **not** work if you just double-click the HTML file and open it straight from your file system. Here's why, and how to actually run it — read this part first, it'll save you some confusion.

### Why double-clicking the file won't play anything

When you open an HTML file directly from your computer, your browser loads it as a `file://` address. Browsers treat every `file://` page as its own locked-down security origin — and the YouTube IFrame API needs to talk back and forth with your page to actually initialize the player. That handshake gets blocked outright on a `file://` page, so the player UI shows up fine, but pressing play does nothing. No crash, no obvious error — it just silently never finishes loading.

If you open your browser's console (F12 → Console) while this is happening, you'll see something like:

```
Unsafe attempt to load URL file:///... from frame with URL file:///...
'file:' URLs are treated as unique security origins.
```

That message is the browser confirming exactly this — it's not a bug in the code, it's a security boundary that has nothing to do with what's written in the file.

### How to actually run it

**Option 1 — GitHub Pages (the real fix, and how you'll actually share this):**

1. Turn on GitHub Pages in your repo settings.
2. Drop `mahou_amp_player.html` (or `index.html`) wherever Pages is serving from.
3. It'll be live at `https://yourusername.github.io/your-repo/`.
4. Once it's loading from that `https://` address instead of `file://`, the origin restriction is gone and playback works normally — no code changes needed, it's purely about how the page is being served.

```markdown
🎧 [Open the interactive player](https://yourusername.github.io/your-repo/mahou_amp_player.html)
```

**Option 2 — a quick local server, for testing changes before you push:**

If you don't want to push to GitHub every time you tweak something, serve the folder locally instead of double-clicking the file. On Windows:

1. Open the folder containing the HTML file in File Explorer.
2. Click into the address bar at the top, type `cmd`, hit Enter — this opens a terminal already pointed at that folder.
3. Run:
   ```
   python -m http.server 8000
   ```
   (If that's not recognized, try `py -m http.server 8000` instead.)
4. Leave that terminal open, then visit `http://localhost:8000/mahou_amp_player.html` in your browser.

Either option gives you a real `http(s)://` origin instead of `file://`, which is the only thing that actually matters here.

### Once it's actually running

1. ▶ / ⏸ / ◀ / ▶▶ actually control playback
2. Clicking a track loads and plays it
3. Scroll or drag through the playlist to see all 9 tracks
4. Drag the seek bar to jump around in the song, drag the volume slider to control how loud it is

You'll need an internet connection either way, since it's streaming from YouTube in the background — but once it's served properly, it runs in basically any modern browser, desktop or mobile.

---

## Swapping in your own tracks

### YouTube (this is what it uses by default)

Just edit the `tracks` array near the top of the `<script>` section:

```js
var tracks = [
  {artist:"Artist Name", title:"Song Title", videoId:"VIDEO_ID_HERE"},
  // ...
];
```

The `videoId` is just the part after `watch?v=` in any YouTube link — so `youtube.com/watch?v=nCgOhvZ7AaA` gives you `nCgOhvZ7AaA`. Don't bother typing in durations, they pull automatically (see above for why that took a little extra work).

### Your own local files instead

If you'd rather host your own `.mp3`s than pull from YouTube, swap the YouTube wiring out for a plain `<audio>` tag:

```html
<audio id="audioEl" preload="auto"></audio>
```

```js
var audioEl = document.getElementById('audioEl');

document.getElementById('btnPlay').addEventListener('click', function(){
  audioEl.play(); playing = true; updateAll();
});
document.getElementById('btnPause').addEventListener('click', function(){
  audioEl.pause(); playing = false; updateAll();
});

function loadTrack(i){
  current = i;
  audioEl.src = tracks[i].file; // like "songs/midnight-blue.mp3"
  audioEl.play();
  playing = true;
  updateAll();
}

audioEl.addEventListener('timeupdate', function(){
  var pct = audioEl.currentTime / audioEl.duration;
  setSeek(18 + pct * (302 - 18));
});
audioEl.addEventListener('ended', function(){
  document.getElementById('btnNext').click();
});
audioEl.addEventListener('loadedmetadata', function(){
  durationCache[current] = fmtTime(audioEl.duration);
  renderPlaylist();
});
```

Couple things to know if you go this route:

- You're on your own for hosting the actual audio files — I'm not shipping any songs with this. Use your own stuff or anything you're actually licensed to use.
- Volume control: set `audioEl.volume = clamped / 100` inside `setVol()`.
- The equalizer and spinning disc are just animations tied to whether something's "playing" — they're not actually reacting to the audio's real frequencies. If you want a *true* audio-reactive equalizer, look into the Web Audio API's `AnalyserNode` — that's the real way to get bars that move with the actual sound.

### SoundCloud

SoundCloud's got a Widget API that works pretty similarly to YouTube's (play/pause/seek/events), so the same overall approach would carry over fine. I haven't wired it in here, but it wouldn't take much to add.

### Spotify (the annoying one)

Spotify's the odd one out. Getting this same level of control — my own buttons actually driving playback — needs their Web Playback SDK, which means a Premium account and an OAuth login flow. It can't just play off a public link like YouTube or SoundCloud can. You *can* drop in their basic embed (`open.spotify.com/embed/track/...`), but that gives you Spotify's own player UI sitting in an iframe — my buttons wouldn't be able to touch it.

---

## Making it your own

- **Tracks**: edit the `tracks` array — just needs `artist`, `title`, and `videoId`. Skip the duration, it's automatic.
- **Colors**: all the gradients live in one `<defs>` block at the top of the SVG (`#frame`, `#titlebar`, `#lcd`, `#barglow`) — change those `stop-color` values to retheme the whole thing.
- **How many tracks show at once**: there's a `VIEW_H` variable in the script (currently `190`) that controls how tall the visible playlist window is before it starts scrolling.

---

## License

Personal/portfolio use — <a href="https://github.com/accordanalyst/mahouyo">Mahouyo Amp </a> © 2026 by <a href="https://github.com/accordanalyst/">Analyst Accord</a> is licensed under <a href="https://creativecommons.org/licenses/by-nc-sa/4.0/">CC BY-NC-SA 4.0</a><img src="https://mirrors.creativecommons.org/presskit/icons/cc.svg" alt="" style="max-width: 1em;max-height:1em;margin-left: .2em;"><img src="https://mirrors.creativecommons.org/presskit/icons/by.svg" alt="" style="max-width: 1em;max-height:1em;margin-left: .2em;"><img src="https://mirrors.creativecommons.org/presskit/icons/nc.svg" alt="" style="max-width: 1em;max-height:1em;margin-left: .2em;"><img src="https://mirrors.creativecommons.org/presskit/icons/sa.svg" alt="" style="max-width: 1em;max-height:1em;margin-left: .2em;">
