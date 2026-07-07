# Hetz Electrical — EV Charger Scroll Hero

Handover package for a scroll-driven hero section for the **EV Chargers** service
on [hetzelectrical.com](https://www.hetzelectrical.com/): a video whose
playhead is scrubbed by scroll position (not autoplayed), paired with a
pinned text panel that swaps captions per "chapter" as the user scrolls.

## What's in here

```
assets/
  hetz-ev-hero.mp4              10s, 1920x1080 (16:9), the scroll-scrubbed video
  hetz-ev-hero-wide-reveal.png  Static photo shown after the video's final chapter
webflow/
  hero-embed.html               Drop-in Webflow Embed block (HTML + CSS + JS)
```

## The video

One continuous 10-second, 16:9 shot, generated with fal.ai (Kling 2.5 Turbo
Pro image-to-video, start+end frame interpolation):

| Time | Chapter | Visual |
|---|---|---|
| 0.0s–4.0s | 01 — The Current | Camera flies through the inside of a cable — copper strands and a blue energy pulse rushing past |
| 4.0s–6.0s | 02 — Contact | Camera bursts out through an iris/aperture, resolving into a photoreal close-up of the actual wall-mounted EV charger, glowing indicator ring |
| 6.0s–10s | 03 — Home | Camera settles; scroll then swaps to the static wide photo (`hetz-ev-hero-wide-reveal.png`) showing the full driveway/charger/car scene, for the resting frame under the final caption + CTA |

The video is muted with no audio track — it's a background element, not a
video-with-sound.

**Known cosmetic note:** there's a faint mark near the top of the charger
housing around the 6s–10s mark that wasn't verifiable against the source
photo (that area was outside the original crop used to anchor the
generation). It reads as a generic charger badge/notch at normal viewing
size, but flagging it in case a designer wants to touch it up or blur it in
an editing pass.

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
  timeline (0s–10s) and to the moment the static wide photo swaps in.
  These are already tuned to the video's content (see table above) — only
  change them if the video itself changes.
- **Copy**: the three `.ev-hero__chapter` blocks in the HTML hold the
  eyebrow/headline text per chapter — edit directly.

### Mobile note

Scroll-scrubbed video works on modern mobile browsers, but `currentTime`
seeking on iOS Safari can be less smooth than desktop. If mobile QA shows
jank, a common fallback is to swap the mobile breakpoint to a simple
autoplaying muted loop (no scroll-scrub) with the text stacked underneath
instead of pinned — flagging this as a possible follow-up, not built here.

## Source assets / regeneration

If the video ever needs to be regenerated or extended, it was built via:

1. A still image looking into a cable's cross-section (fal.ai
   `flux-pro/v1.1`) as the start anchor.
2. A tight crop of the real charger photo (`hetz-ev-hero-wide-reveal.png`,
   cropped to just the charger) as the end anchor.
3. A single `fal-ai/kling-video/v2.5-turbo/pro/image-to-video` call at
   `duration: "10"` with both anchors set (`image_url` / `tail_image_url`),
   describing the full camera journey in one prompt — letting the model
   interpolate the whole arc in one pass rather than stitching separate
   clips together.

This single-call, dual-anchor approach avoided artifacts that came up when
earlier attempts stitched two independently-generated clips together
(mismatched objects at the seam).
