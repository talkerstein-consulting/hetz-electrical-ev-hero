# Hetz Electrical — EV Charger Scroll Hero

Handover package for a scroll-driven hero section for the **EV Chargers** service
on [hetzelectrical.com](https://www.hetzelectrical.com/): a video whose
playhead is scrubbed by scroll position (not autoplayed), paired with a
pinned text panel that swaps captions per "chapter" as the user scrolls.

## What's in here

```
assets/
  hetz-ev-hero.mp4              4.5s, 1920x1080 (16:9), the scroll-scrubbed video
  hetz-ev-hero-wide-reveal.png  Static photo (video's last frame) shown after the video ends
webflow/
  hero-embed.html               Drop-in Webflow Embed block (HTML + CSS + JS)
```

## The video

One continuous 4.5-second, 16:9 shot:

| Time | Chapter | Visual |
|---|---|---|
| 0.0s–~1.5s | 01 — The Current | Camera flies through a glowing energy tunnel — blue current with red/yellow charge lines rushing toward a bright convergence point |
| ~1.5s–~3.5s | 02 — Contact | Close-up of a wall-connector cable being plugged into the charge port of a purple EV in a garage |
| ~3.5s–4.5s | 03 — Home | Camera settles into a wide side profile of the car parked next to the mounted wall charger; scroll then swaps to the static wide photo (`hetz-ev-hero-wide-reveal.png`, extracted from the video's last frame) for the resting frame under the final caption + CTA |

The video is muted with no audio track — it's a background element, not a
video-with-sound.

The chapter boundary timestamps above are approximate — re-check them against
the actual footage if the captions ever feel out of sync with the visuals,
and adjust `CHAPTER_BOUNDARIES` in `hero-embed.html` accordingly (see Tuning
below).

## Setting it up in Webflow

Webflow doesn't support scroll-scrubbed `<video>` natively — this needs a
small amount of custom code. Everything required is self-contained in
`webflow/hero-embed.html`.

### Steps

1. **Host the assets.** Webflow's Asset Manager can hold the video directly
   (Assets panel → upload `hetz-ev-hero.mp4` and
   `hetz-ev-hero-wide-reveal.png`), or use an external CDN
   (Cloudflare R2, Bunny, Cloudinary) if you'd rather not count it against
   Webflow's asset storage. Either way, you'll get a hosted URL for each
   file.

2. **Open `webflow/hero-embed.html`** and replace the two placeholders:
   - `VIDEO_SRC` → your hosted video URL
   - `WIDE_REVEAL_SRC` → your hosted image URL

3. **In the Webflow Designer**, drop an **Embed** component where the EV
   Charger hero section should live, and paste the entire contents of
   `hero-embed.html` into it. It's self-contained (structure + `<style>` +
   `<script>` all in one block), so no other Webflow-side setup is
   required.

4. **Publish and test on scroll** — the video's playhead should track
   scroll position (not autoplay), and the caption panel should step
   through three chapters as you scroll through the section.

### Tuning

- **Scroll length**: the hero section is `400vh` tall (see `.ev-hero` in
  the embed's CSS) — this controls how much scrolling it takes to get
  through the whole sequence. Increase/decrease that value to make the
  scroll feel slower/faster.
- **Chapter boundaries**: `CHAPTER_BOUNDARIES` and `WIDE_REVEAL_AT` in the
  `<script>` block map scroll progress (0–1) to the video's actual
  timeline (0s–4.5s) and to the moment the static wide photo swaps in.
  They're expressed as fractions of total scroll progress, not absolute
  seconds, so they don't need to change just because the video's duration
  changed — but re-tune them if the new chapter cut points land at
  different fractions of the video (see table above for the current
  approximate cut points).
- **Copy**: the three `.ev-hero__chapter` blocks in the HTML hold the
  eyebrow/headline text per chapter — edit directly.

### Mobile note

Scroll-scrubbed video works on modern mobile browsers, but `currentTime`
seeking on iOS Safari can be less smooth than desktop. If mobile QA shows
jank, a common fallback is to swap the mobile breakpoint to a simple
autoplaying muted loop (no scroll-scrub) with the text stacked underneath
instead of pinned — flagging this as a possible follow-up, not built here.

## Source assets / regeneration

The current `hetz-ev-hero.mp4` is a client-supplied video (replacing an
earlier fal.ai-generated version). `hetz-ev-hero-wide-reveal.png` is simply
the video's last frame, extracted with ffmpeg:

```
ffmpeg -sseof -0.1 -i hetz-ev-hero.mp4 -frames:v 1 -q:v 2 hetz-ev-hero-wide-reveal.png
```

If the video is swapped again, re-run that extraction so the wide-reveal
photo keeps matching the video's final frame.
