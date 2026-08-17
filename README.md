# Awake? landing

Marketing site for Awake?, the macOS menu bar app that keeps client clocks one
click away.

Single self-contained `index.html`, same as the Pixlift and Worth it? sites.
Deploy to Vercel as a static site, no build step.

## What ships in this folder

| File | Notes |
|---|---|
| `index.html` | The whole page. Styles, markup and the interactive demo, inline. |
| `Awake-0.1.7.dmg` | The download the page links to *and* the Sparkle enclosure — one artifact, two audiences. `tools/release.sh` now copies it in and deletes the superseded one itself; the three `href="Awake-…dmg"` links plus the size in the hero still need updating by hand. |
| `appcast.xml` | The Sparkle update feed. Written by `tools/release.sh`, never by hand — its version, build and length are read out of the DMG and its signature comes from the Keychain. |
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

## Where it is published

<https://shahzaibb101.github.io/awake-landing/>, GitHub Pages, served from the
root of `main` in the **public** `shahzaibb101/awake-landing` repo.

This folder is the source of truth. `awake-landing` is a deploy target holding
nothing but the contents of this directory, which is why the app source can stay
private while the site is public. Publish with:

```bash
git subtree push --prefix=site landing main
```

from the repo root, where `landing` is the remote for `awake-landing`. Commit to
`awake` first; the subtree push only carries what is already committed.

`.nojekyll` turns off Jekyll processing, which the page has no use for.

`og:image`, `twitter:image` and `og:url` in `index.html` are **absolute** and
hardcode that Pages URL, because scrapers do not all resolve a relative
`og:image` against the page. If the site ever moves to its own domain, those
three are what change.

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
(34,080 cities, 418 zones, 214 tests, 2.4 MB) are real and worth re-checking
before a release.

## Local preview

```bash
python3 -m http.server 8931
```

Then open <http://localhost:8931>. Opening `index.html` as a `file://` URL works
too, but a server is closer to production.
