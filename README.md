# Awake? landing

Marketing site for Awake?, the macOS menu bar app that keeps client clocks one
click away.

Single self-contained `index.html`, same as the Pixlift and Worth it? sites.
Deploy to Vercel as a static site, no build step.

## What ships in this folder

| File | Notes |
|---|---|
| `index.html` | The whole page. Styles, markup and the interactive demo, inline. |
| `Awake-0.1.0.dmg` | The download the page links to. Copy a fresh one in after every `tools/release.sh` run and update the three `href="Awake-0.1.0.dmg"` links plus the size in the hero. |
| `og.png` | 1200x630 share card. Generated, not hand-drawn, see below. |
| `og-card.html` | The source of `og.png`, at exactly 1200x630. |
| `og-fonts/` | Bricolage Grotesque and IBM Plex Mono, copied from the app bundle so the card renders without touching the network. Both OFL 1.1. |

## Regenerating the share card

`og-card.html` is a 1200x630 page; the PNG is a headless Chrome screenshot of
it. Rendered at 2x and downscaled, because rendering straight to 1200x630 gives
visibly softer type.

```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless=new --disable-gpu --hide-scrollbars --allow-file-access-from-files \
  --force-device-scale-factor=2 --window-size=1200,630 \
  --screenshot=og@2x.png "file://$PWD/og-card.html"
sips -Z 1200 og@2x.png --out og.png && rm og@2x.png
```

`--allow-file-access-from-files` is what lets the `@font-face` rules load out of
`og-fonts/`. Without it Chrome silently falls back to a system face and the
render looks nothing like the product.

The card shows 4:00 PM in New York landing as 1:00 AM the next day in Dubai.
Those four times are hand-set and correct for **standard** time, EST at UTC-5.
They are wrong by an hour during US daylight saving, which is a fine trade for a
static image whose whole point is the headline, but it is why they are not
labelled with a date.

## One thing to fix before launch

`og:image` and `twitter:image` in `index.html` are **relative** paths, because
the production domain is not settled. Facebook, Slack and X resolve those
against the page URL; some other scrapers do not. Once the domain exists, make
both absolute (`https://.../og.png`).

## The demo is the app's real logic, reimplemented

The panel in the hero is not a screenshot. It runs the same rules the Swift
engine does, in about 200 lines of plain JavaScript, against `Intl` rather than
`Foundation`:

- the 24 hour range is centred on now, and the handle snaps to quarter hours
- every row converts the same instant, and shows `+1d` / `-1d` when that instant
  lands on a different calendar day than yours
- working windows are sampled at the same quarter hour and shaded behind the row
- Yuki has no hours entered, so she gets a clock and nothing else, which is the
  behaviour `Person.tracksHours` exists to produce
- the mock menu bar runs the priority ladder from `Summary.swift`, at the real
  now, and **deliberately does not follow the scrubber** — that is the claim the
  caption makes, so the page has to honour it

Rung 2 of the ladder, "closes in 45 minutes", is opt in per person and nobody in
the demo roster opted in, so it never fires here. Everything else does.

If the app's rules change, this page is now a second implementation that can
drift. It is a marketing page, not a source of truth, but the numbers in it
(34,080 cities, 418 zones, 136 tests, 1.4 MB) are real and worth re-checking
before a release.

## Local preview

```bash
python3 -m http.server 8931
```

Then open <http://localhost:8931>. Opening `index.html` as a `file://` URL works
too, but a server is closer to production.
