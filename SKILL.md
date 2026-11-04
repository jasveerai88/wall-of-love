---
name: wall-of-love
description: Turn a testimonials file (JSON or CSV — name, role, quote, rating, date, optional source URL and avatar) into a wall of love you own — ONE self-contained static HTML page, all CSS inline, zero JavaScript (the only <script> is an inert schema.org JSON-LD block), no external requests except the configured logo, avatars and source links. `build` validates the input then writes the page (default wall.html); `preview` prints the testimonials in display order, the aggregate, the page weight in bytes and the checks passed, writing nothing. Sorting is highest rating first, then newest date; unrated sort last. The aggregate is the exact mean of the ratings present, rounded to one decimal (half-up); the review count is the number of testimonials in the file; both are computed once and the SAME values appear on the wall and in the JSON-LD, so they cannot drift. Validation is part of the product: a rating outside integer 1–5 (0, 6, "five", 4.5, true), an empty quote, a missing name, a bad date or an unsafe source/avatar scheme is REFUSED with exit 2 and nothing written; a duplicate quote (case/whitespace-insensitive) or a missing rating is a warning — the page still builds, exit 1. Every user string is HTML-escaped, and the JSON-LD is escaped for embedding (<, >, & as unicode escapes), so a quote holding markup renders as text and can never break out of the data block. The wall shows ONLY the testimonials in the input file, verbatim — never invented, padded or paraphrased. Use when a small business needs a testimonials page without a Senja/Testimonial.to-style subscription. Exit 1 is a finding (warnings), not a failure.
---

# wall-of-love — agent skill

## Purpose

Testimonial SaaS charges rent forever for a guestbook: Senja
$29/mo with a free tier capped at 15 testimonials for the life
of the account, Testimonial.to $25/mo, Famewall branding until
$79.99/mo. This tool generates the wall as one static HTML
file from one testimonials file — ownable, hostable anywhere,
permanent — and generates the schema.org Review JSON-LD from
the same data, which is the part the rivals paywall. Pure
Python standard library; local only; the input file is the
only thing it reads, and nothing it produces phones home.

**The honesty core of this tool:** the wall is evidence, and
evidence is never manufactured. The tool displays exactly the
testimonials in the input file, verbatim — if the file has 3,
the wall shows 3 and the review count says 3. The aggregate is
arithmetic on the ratings present, not a marketing number: an
unrated testimonial is shown without stars and excluded from
the mean, and if nothing is rated there is no aggregate at
all, on the wall or in the JSON-LD.

## When to use / not use

Use when:
- A business needs a testimonials page they own: write the
  file, `build`, put the HTML on any static host.
- A pipeline needs a yes/no gate before publishing a wall:
  `build` exit 0 = clean; exit 1 = published-with-warnings
  (decide if the duplicate is deliberate); exit 2 = do not
  publish, the error names the field to fix.
- An agent assembles a wall from data the user already holds
  (a review-platform export converted to JSON/CSV) and needs
  the ratings, duplicates and aggregate checked before it
  ships.

Do NOT use when:
- You need to COLLECT testimonials — this tool displays only.
  Collection stays with whatever form the business already
  has; that is the honest scope, stated up front.
- The testimonials are not real. Feeding this tool invented
  quotes produces a fraudulent wall with valid JSON-LD; the
  tool cannot detect that, and neither the tool nor your
  summary may dress it up. If you do not hold real, consented
  testimonials, do not build the wall.
- You need to edit a quote "for length" or "for punch" — the
  tool renders quotes verbatim by design. Any trimming happens
  in the input file, by the user, before the run.

## Commands

Every command takes the testimonials file as a positional
argument and supports `--json`. The input format is chosen by
extension: `.json` or `.csv`.

```bash
# Build the page (exit 1 with warnings; exit 2 and NO file
# written on any validation error)
python3 src/wall_of_love.py build TESTIMONIALS \
    [-o wall.html]

# Preview: display order, aggregate, page weight in bytes,
# checks passed — writes nothing
python3 src/wall_of_love.py preview TESTIMONIALS
```

JSON input is either a bare list of testimonials or an object
`{"business"?, "tagline"?, "logo"?, "theme"?, "testimonials":
[...]}` — the page fields (`business`, `tagline`, `logo`,
`theme`) exist only in the object form. CSV input is a header
row with columns `name,role,quote,rating,date,source,avatar`
(only `name` and `quote` columns are required; extra columns
are ignored; an empty rating cell means "no rating").

Per-testimonial fields: `name` (required, non-empty string),
`quote` (required, non-empty string), `role` (optional string
— role/company as one free-text line), `rating` (optional
integer 1–5), `date` (optional, `YYYY-MM-DD` exactly),
`source` (optional http(s) URL), `avatar` (optional http(s)
URL or local path, referenced in an `<img>`, never embedded).

`theme` is an optional object: `accent` = `#rgb` or `#rrggbb`
(default `#b3541e`), `mode` = `"light"` (default) or `"dark"`.

## Validation rules (exact)

Errors (exit 2, nothing written, the message names the
field): a missing `name` or `quote`; an empty or
whitespace-only `quote`; a rating that is present but not an
integer from 1 to 5 — `0`, `6`, `"five"`, `4.5` and `true` are
all refused (the strings `"1"`–`"5"` and an empty CSV cell are
the only string forms accepted, meaning 4 and "no rating");
a `date` that is not `YYYY-MM-DD` or not a real date; a
`source` that is not an http(s) URL with a host; an `avatar`
or `logo` with a scheme other than http(s); a bad
`theme.accent` or `theme.mode`; a malformed JSON or CSV file;
an empty file; an empty testimonials list; a JSON object with
no `testimonials` list; a CSV missing its `name` or `quote`
column or carrying no data rows.

Warnings (exit 1, the page IS built): a duplicate quote — two
testimonials whose quotes match after case-folding and
whitespace collapsing — and a missing rating. Both warnings
name the testimonials involved.

Checks passed are reported by name: every name and quote
present; every rating present is an integer from 1 to 5;
single file with no external requests besides the configured
logo, avatars and source links; JSON-LD generated from the
same data as the wall; the aggregate from the rated count
(when one exists); no duplicate quotes and no missing ratings
(only when true — the line is absent, not faked, when warnings
exist).

## The aggregate (exact)

`aggregate = mean(ratings present)`, computed in decimal and
rounded to one decimal, half-up (4.25 → 4.3, never banker's
4.2). `reviewCount` = the number of testimonials in the file,
rated or not. The summary line on the wall, the `preview`
output, the `--json` payload and the JSON-LD `aggregateRating`
all carry the same two numbers, computed once inside the tool.
If no testimonial is rated, there is no aggregate: no summary
line, no `aggregateRating` in the JSON-LD, `aggregate: null`
in `--json`.

## The page (exact)

One `.html` file: `<!DOCTYPE html>`, `<html lang="en">`,
charset + viewport meta, `<title>{business} — Wall of Love`,
a meta description from the tagline, ALL CSS in one inline
`<style>` (system font stack; accent colour; `:focus-visible`
outlines; responsive card grid). One `<script
type="application/ld+json">` block holding schema.org
`LocalBusiness` data: `name`, `aggregateRating`
(`ratingValue` as a one-decimal string, `reviewCount`,
`bestRating`/`worstRating`) when ratings exist, and a `review`
array in display order (`author`, `reviewBody`, plus
`reviewRating`, `datePublished` and `url` when the input
carries them). **That data block is the only `<script>`
element in the output** — there is no JavaScript. Body:
`header` (logo `<img>` when configured, `h1`, tagline, the
aggregate summary), `main` with a testimonials `section` of
`figure` cards: `blockquote` quote (verbatim, escaped), text
stars (★/☆ characters, not images), `figcaption` with avatar
`<img>` (when configured), name, role, date and an "Original
review" source link (when configured). Every user-supplied
string is HTML-escaped in text and attributes; inside the
JSON-LD, `<`, `>` and `&` are written as JSON unicode escapes
(`<`, `>` and `&` become `\u003c`, `\u003e` and
`\u0026`), so the embedded data round-trips exactly
through a JSON parser while no quote can interact with the
HTML parser. Page weight = `len(page.encode("utf-8"))` —
`preview`'s number is exactly the byte length of the file
`build` writes.

## Output JSON schema (`--json`)

Common fields: `tool` (`"wall-of-love"`), `version`,
`command`, `input_file` (file name), `business` (string or
null), `page_bytes`, `count` (all testimonials),
`rated_count`, `aggregate` (one-decimal number or null),
`testimonials` — each `{name, quote, role?, rating?, date?,
source?, avatar?}` in DISPLAY order (rating desc, then date
desc) — `checks_passed` (list of strings), `warnings` (list
of strings). `build` adds `output_file` (the path written, as
given or `wall.html`).

On error, stdout is a single object: `{"error": "<message>"}`
with exit code 2 — including argparse usage errors in `--json`
mode. Never plain text, never a traceback.

## Exit codes

- `0` — clean: `build` wrote the page with no warnings;
  `preview` found nothing to flag.
- `1` — a finding: duplicate quotes and/or missing ratings.
  The page WAS built (for `build`) — act on the warnings,
  don't treat them as a crash.
- `2` — ERROR: input missing or malformed, a required field
  missing, an out-of-range rating, a bad date, an unsafe URL
  scheme, an unwritable output path, an invalid flag. For
  `build`, NO file was written. No output facts may be quoted
  from a failed run.

## Limits & honesty rules the agent must respect

- **Never invent, pad or paraphrase a testimonial.** The wall
  shows only real input testimonials, verbatim. Do not top a
  wall up to a rounder number, do not merge two quotes into
  one, do not "fix" wording. A wall of 3 honest quotes is the
  product working; a wall of 12 with 9 invented is fraud with
  good markup.
- **The aggregate is arithmetic, not copy.** Report it exactly
  as the tool computes it, with its rated count: "4.7 from 6
  rated testimonials", never a bare "5-star rated!" unless the
  rated mean actually is 5.0 — and even then, name the count.
- **A warning is not an error and not nothing.** Exit 1 means
  the page exists AND carries a repeated quote or unrated
  entries. Surface the warnings verbatim; a duplicate may be
  a genuine second customer saying the same words, or padding
  — the user decides; never silently de-duplicate by hand.
- **Exit 2 from `build` means no page exists.** Do not
  reconstruct the HTML yourself from the input to "help" —
  the refused rating or empty quote is exactly why the gate
  exists. Fix the input and re-run.
- **The no-external-requests contract is the product.** If you
  post-process the page, do not add fonts, CDNs, trackers or
  scripts. The configured logo, avatars and source links are
  the only external references allowed. (The string
  `https://schema.org` inside the JSON-LD is a vocabulary
  identifier, not a request — no browser fetches it.)
- **Avatars and the logo are references.** A local-path image
  must be hosted beside the page; never claim the page is
  fully self-contained when image URLs are configured — say
  "one file plus the images it references".
- **Unrated is a real state, not a defect to hide.** A
  testimonial without a rating displays without stars and is
  excluded from the aggregate. Never assign it a rating
  yourself to make the numbers prettier.
- Local only: the tool reads only the testimonials file named
  on the command line and sends nothing anywhere.
