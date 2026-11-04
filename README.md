# wall-of-love

**Your customers' kind words should not cost $29 a month.**

Senja charges $29/mo for a testimonials page — and its free
tier is capped at 15 testimonials *for the lifetime of the
account*; deleting one doesn't give the slot back.
Testimonial.to starts at $25/mo and still caps your video
testimonials; Famewall keeps its own branding on your wall
until the $79.99/mo plan. Over three years Senja's Starter
alone comes to $1,044 — for quotes your customers wrote once.
A guestbook, rented forever.

`wall-of-love` generates the guestbook instead. One
testimonials file (JSON or CSV) in, **one self-contained HTML
page** out — all CSS inline, no JavaScript, no trackers, and a
schema.org Review JSON-LD block generated from the same data
as the wall, so the SEO numbers can never drift from what
visitors see. Host it on GitHub Pages, your own site, any
static host, forever, for free. It cannot be shut down,
capped, or repriced, because there is no platform — it's a
file, and it's yours.

- Pure Python 3 standard library — no dependencies, no install step
- Local only — the tool reads one file and writes one file;
  it never fetches the network
- Script-friendly exit codes: `0` = clean, `1` = built with
  warnings (duplicate quotes, missing ratings), `2` = error,
  nothing written
- Honest by construction: the wall shows exactly the
  testimonials in your file, verbatim. Nothing invented,
  nothing padded.

## The input

A JSON file — either a bare list of testimonials, or an
object carrying page fields:

```json
{
  "business": "Okanagan Stone & Cedar",
  "tagline": "Fences, decks and stone work across the Okanagan.",
  "logo": "https://example.org/okanagan/logo.png",
  "theme": {"accent": "#7a4a21", "mode": "light"},
  "testimonials": [
    {"name": "Priya Nair", "role": "Homeowner, Kelowna",
     "quote": "The cedar fence has survived two winters...",
     "rating": 5, "date": "2026-09-14",
     "source": "https://example.org/reviews/priya-nair",
     "avatar": "https://example.org/avatars/priya.jpg"},
    {"name": "Sam Okafor", "role": "Café owner, Penticton",
     "quote": "Lovely crew, lovely work."}
  ]
}
```

Or a CSV with a header row — `name,role,quote,rating,date,
source,avatar` (only `name` and `quote` columns are required;
extra columns are ignored; the page fields live in the JSON
object form only).

| Field | Meaning |
|---|---|
| `name` | who said it (required, non-empty) |
| `quote` | what they said (required, non-empty) — rendered **verbatim** |
| `role` | role/company as one line, e.g. "Owner, Trailhead Coffee" (optional) |
| `rating` | integer 1–5 (optional — see the rules below) |
| `date` | `YYYY-MM-DD` (optional) |
| `source` | an http(s) URL where the testimonial lives (optional) — rendered as an "Original review" link |
| `avatar` | an http(s) URL or local path (optional) — **referenced** in an `<img>`, never embedded |
| `business` | page heading (object form; default "Wall of Love") |
| `tagline` | one line under the heading (object form, optional) |
| `logo` | an http(s) URL or local path (object form, optional) — referenced, never embedded |
| `theme.accent` | hex colour, `#rgb` or `#rrggbb` (default `#b3541e`) |
| `theme.mode` | `"light"` (default) or `"dark"` |

The input is validated on every run, and every error names
the field: `testimonials[0].rating`, `row[2].date`,
`theme.accent`. A malformed file is a clean exit-2 error,
never a traceback.

## Validation is the product

- **Ratings are strict.** A rating that is present but not an
  integer from 1 to 5 — `0`, `6`, `"five"`, `4.5`, `true` —
  is **refused**. A wall is evidence; a rating the tool cannot
  defend does not reach the page.
- **A missing rating is a warning, not an error.** Real
  testimonials collected on paper have no stars. The entry
  shows without stars and is excluded from the aggregate; the
  page still builds, exit 1.
- **Duplicate quotes are flagged.** Two testimonials whose
  quotes match (ignoring case and whitespace) produce a
  warning naming both. The page still builds — a second
  customer may genuinely have said the same words — but a
  repeated quote reads as padding, and you should know.
- **Quotes and names are required and non-empty.** Dates must
  be real `YYYY-MM-DD` dates. Source links must be http(s).
  Avatar/logo schemes other than http(s) are refused — a
  `javascript:` "image" is not an image.

## The aggregate never drifts

The aggregate is the exact mean of the ratings present,
rounded to one decimal (half-up). The review count is the
number of testimonials in the file, rated or not. Both are
computed **once**, inside the tool, and the same numbers are
printed in the wall's summary line, the `preview` output, the
`--json` payload and the JSON-LD `aggregateRating` — they
cannot disagree, because there is only one computation. If no
testimonial carries a rating, there is no aggregate anywhere:
no summary line, no `aggregateRating`, `aggregate: null`.

## The page itself

- **One file, everything inline.** No external stylesheets,
  fonts or scripts — a system font stack, so there is nothing
  to wait for. The sample page below weighs **8,326 bytes**.
- **Zero JavaScript.** The only `<script>` in the output is
  the inert `application/ld+json` data block — no code runs.
- **Escaped, always.** Every name, quote, role and URL is
  HTML-escaped in text and in attributes, and inside the
  JSON-LD `<`, `>` and `&` are written as JSON unicode
  escapes — a quote containing `<script>alert(1)</script>`
  renders as visible text and cannot break out of the data
  block either. A JSON parser reads the embedded block back
  to the exact original strings.
- **Sorted for readers.** Highest rating first, then newest
  date; unrated entries after rated ones, undated after
  dated; ties keep input order.
- **Accessible by construction.** Semantic HTML (`header`,
  `main`, `figure`/`blockquote`/`figcaption`, labelled
  section), text stars with an aria label, visible
  `:focus-visible` outlines, responsive card grid.

## Usage

```bash
# 1. Build the wall (default output: wall.html)
python3 src/wall_of_love.py build \
    examples/sample-testimonials.json -o okanagan-wall.html

# 2. Preview first: display order, aggregate, page weight
python3 src/wall_of_love.py preview \
    examples/sample-testimonials.json

# 3. CSV works too
python3 src/wall_of_love.py build \
    examples/sample-testimonials.csv -o csv-wall.html

# 4. Drive it from a script: exit 1 means warnings to review
python3 src/wall_of_love.py build quotes.json || echo "check warnings"
```

Then put the HTML file anywhere static: GitHub Pages, your
existing website, Netlify/Vercel drag-and-drop, an S3 bucket.
Collect testimonials however you already do — a form, an
email thread, a spreadsheet export — and rebuild the wall
when the file changes.

## Use with AI agents

Every command takes `--json`: one JSON object with the same
facts and exit codes as the text view (errors come back as
`{"error": ...}`, argparse usage errors included):

```bash
python3 src/wall_of_love.py build \
    examples/sample-testimonials.json -o wall.html --json
```

Agents: read [SKILL.md](SKILL.md) first — commands, the full
JSON schemas, the exit codes, and the honesty rules (never
invent, pad or paraphrase a testimonial; the aggregate is
arithmetic, not copy; a warning is not an error).

## Real runs (verbatim)

The bundled sample is a wall for a fictional Okanagan
fence-and-deck builder: 6 testimonials with ratings
5, 5, 4, 5, 4, 5 — sum 28, mean 28/6 = 4.666…, so the
aggregate is 4.7. The preview:

```text
$ python3 src/wall_of_love.py preview examples/sample-testimonials.json
wall-of-love v0.1.0 — preview · file sample-testimonials.json
Okanagan Stone & Cedar · 6 testimonial(s) · page weight 8,326 bytes
Aggregate: 4.7 out of 5 from 6 rated testimonial(s) · review count 6

Testimonials, in display order (rating, then newest):
  1. Priya Nair · Homeowner, Kelowna — ★★★★★ · 2026-09-14
  2. Marcus Webb · Property manager, Webb Rentals — ★★★★★ · 2026-08-02
  3. Tom Bergeron · Owner, Trailhead Coffee — ★★★★★ · 2026-07-19
  4. Dan Whitfield · Strata council chair, Lakeshore Condos — ★★★★★ · 2026-05-23
  5. Elena Petrova · Homeowner, West Kelowna — ★★★★☆ · 2026-10-05
  6. Aisha Khan · Homeowner, Peachland — ★★★★☆ · 2026-06-11

Checks passed:
  ✓ 6 testimonial(s), every name and quote present
  ✓ every rating present is an integer from 1 to 5
  ✓ single file, no external requests besides the configured logo, avatars and source links
  ✓ JSON-LD generated from the same data as the wall — aggregate and count cannot drift
  ✓ aggregate 4.7 from 6 rated testimonial(s)
  ✓ accent colour is a valid hex value
  ✓ no duplicate quotes, no missing ratings
Warnings: none
(exit code: 0)
```

The build writes exactly the page the preview weighed:

```text
$ python3 src/wall_of_love.py build examples/sample-testimonials.json -o okanagan-wall.html
wall-of-love v0.1.0 — build · file sample-testimonials.json
Wrote okanagan-wall.html — 8,326 bytes · 6 testimonial(s)
Aggregate: 4.7 out of 5 from 6 rated testimonial(s) · review count 6

Testimonials, in display order (rating, then newest):
  1. Priya Nair · Homeowner, Kelowna — ★★★★★ · 2026-09-14
  2. Marcus Webb · Property manager, Webb Rentals — ★★★★★ · 2026-08-02
  3. Tom Bergeron · Owner, Trailhead Coffee — ★★★★★ · 2026-07-19
  4. Dan Whitfield · Strata council chair, Lakeshore Condos — ★★★★★ · 2026-05-23
  5. Elena Petrova · Homeowner, West Kelowna — ★★★★☆ · 2026-10-05
  6. Aisha Khan · Homeowner, Peachland — ★★★★☆ · 2026-06-11

Checks passed:
  ✓ 6 testimonial(s), every name and quote present
  ✓ every rating present is an integer from 1 to 5
  ✓ single file, no external requests besides the configured logo, avatars and source links
  ✓ JSON-LD generated from the same data as the wall — aggregate and count cannot drift
  ✓ aggregate 4.7 from 6 rated testimonial(s)
  ✓ accent colour is a valid hex value
  ✓ no duplicate quotes, no missing ratings
Warnings: none
(exit code: 0)
```

Scanning the generated page for external references returns
exactly the URLs the input asked for — the logo, one avatar,
two source links — plus `https://schema.org`, the JSON-LD
vocabulary identifier (a namespace name inside the data
block; no browser ever fetches it):

```text
$ grep -oE 'https?://[^" <>]+' okanagan-wall.html | sort -u
https://example.org/avatars/priya.jpg
https://example.org/okanagan/logo.png
https://example.org/reviews/priya-nair
https://example.org/reviews/tom-bergeron
https://schema.org
```

The warnings sample — one quote repeated verbatim under two
names, one testimonial with no rating — builds, and exits 1:

```text
$ python3 src/wall_of_love.py build examples/sample-warnings.json -o warnings-wall.html
wall-of-love v0.1.0 — build · file sample-warnings.json
Wrote warnings-wall.html — 4,325 bytes · 3 testimonial(s)
Aggregate: 4.5 out of 5 from 2 rated testimonial(s) · review count 3

Testimonials, in display order (rating, then newest):
  1. Priya Nair · Homeowner, Kelowna — ★★★★★ · 2026-09-14
  2. Jordan Ellis · Homeowner, Vernon — ★★★★☆ · 2026-08-30
  3. Sam Okafor · Café owner, Penticton — no rating · 2026-07-15

Checks passed:
  ✓ 3 testimonial(s), every name and quote present
  ✓ every rating present is an integer from 1 to 5
  ✓ single file, no external requests besides the configured logo, avatars and source links
  ✓ JSON-LD generated from the same data as the wall — aggregate and count cannot drift
  ✓ aggregate 4.5 from 2 rated testimonial(s)
Warnings:
  ! duplicate quote: testimonials[1] ('Jordan Ellis') repeats testimonials[0] verbatim — the wall shows what the file holds, but a repeated quote reads as padding
  ! missing rating: testimonials[2] ('Sam Okafor') has no rating — it is shown without stars and excluded from the aggregate
(exit code: 1)
```

And a refused rating — the error names the field, and no
file is written:

```text
$ python3 src/wall_of_love.py build bad.json -o bad.html
ERROR: testimonials[0]: field 'rating' must be from 1 to 5 (got 9)
(exit code: 2)
```

## Tests

```bash
python3 -m unittest discover -s tests -v
# Ran 76 tests — OK
```

The suite covers both directions: the happy path in JSON and
CSV (all testimonials rendered, display order, logo/avatar/
source references); the aggregate computed by hand three ways
(including half-up rounding — 4.25 renders 4.3, never 4.2);
the JSON-LD parsing back and matching the wall's displayed
numbers exactly; the page's contracts (exactly one `<script>`
element, the external-URL scan pinned to the input, page
weight equal to the written file's byte length); escaping (a
`<script>` quote renders as text in the wall and round-trips
exactly through the JSON-LD; ampersands attribute-safe;
unicode, emoji and a 5,000-character quote verbatim);
warnings exact (duplicates matched across case and
whitespace, missing ratings excluded from the mean but
counted in the review count); and every refusal (ratings 0,
6, `"five"`, 4.5, `true`; empty quote; missing fields; bad
and impossible dates; unsafe source/avatar/logo schemes;
malformed files) naming its field with nothing written and no
traceback, in text and `--json` modes alike.

## Dual use

- **For small businesses (the Sponsors surface):** a
  testimonials page you own outright, generated locally — the
  free, permanent answer to a $29/mo guestbook, a 15-quote
  lifetime cap, and somebody else's branding on your happiest
  page.
- **For its author:** a standard deliverable in Focus
  Brilliant website builds — trades and clinic clients get a
  testimonial page on their own domain inside the website
  package instead of a Senja recommendation that bills them
  forever — plus a sponsors wall for the author's own GitHub
  profile, built from real sponsor quotes when they exist
  (and not before: the wall shows what the file holds).

## Known limits

- **Display only.** This tool does not collect testimonials —
  no forms, no invites, no review-platform imports. Collection
  stays with whatever the business already has; that is the
  honest scope, stated up front rather than implied away.
- **The input is trusted to be true.** The tool validates
  shape, not honesty: it cannot tell a real quote from an
  invented one. Feeding it fabricated testimonials produces a
  fraudulent wall with valid JSON-LD — don't.
- **Page fields live in the JSON object form.** A CSV input
  gets the default heading and theme. Convert to JSON if you
  need a business name, logo or accent colour.
- **Avatars and the logo are referenced, not embedded.** An
  http(s) image is one external request (yours, to your
  host); a local-path image must sit beside the page where
  you host it.
- **Styling is one accent colour and a light/dark mode.** This
  is a testimonials page, not a website builder — and because
  you own the file, you can hand-edit it.

## Privacy

What it touches: only the testimonials file you point it at.
What it sends: nothing — there is no network code in this
tool. The page it generates sends nothing either: no
analytics, no trackers, no fonts — the only requests a
visitor's browser makes are the images and source links you
configured.

## Support

If this retires a testimonial subscription for you, consider
sponsoring: **https://github.com/sponsors/jasveerai88**

## Licence

MIT — see [LICENSE](LICENSE).
