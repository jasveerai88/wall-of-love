# Changelog

## v0.1.0 — 2026-11-04

First release.

- **One testimonials file in, one owned wall of love out** — a
  JSON or CSV file of testimonials (name, role, quote, rating,
  date, optional source URL and avatar) plus optional page
  fields (business, tagline, logo, theme) generates a complete
  testimonials page: one static HTML file, all CSS inline in a
  single `<style>` block, zero JavaScript, a system font
  stack, and no external requests except the configured logo,
  avatars and source links. Responsive card grid, text stars,
  `:focus-visible` outlines. The bundled sample page weighs
  8,326 bytes.
- **`build TESTIMONIALS [-o wall.html]`** — validates, then
  writes the page (default output `wall.html`). Exit 0 =
  clean, 1 = built with warnings (duplicate quotes, missing
  ratings), 2 = error and nothing written.
- **`preview TESTIMONIALS`** — the summary without writing
  anything: the testimonials in display order, the aggregate,
  the page weight in bytes (exactly the byte length of the
  file `build` writes) and the checks passed.
- **JSON-LD from the same data** — a schema.org `LocalBusiness`
  block with `aggregateRating` and the full `review` array is
  generated from the same validated data structure the wall
  renders, so the SEO numbers can never drift from the
  displayed ones. `<`, `>` and `&` are written as JSON unicode
  escapes inside the block, so no quote can interact with the
  HTML parser around it, while a JSON parser reads back the
  exact original strings.
- **Sorting** — highest rating first, then newest date;
  unrated testimonials after rated ones, undated after dated;
  ties keep input order.
- **The aggregate is exact** — the mean of the ratings
  present, computed in decimal, rounded to one decimal
  half-up; the review count is the number of testimonials in
  the file. Unrated testimonials display without stars and are
  excluded from the mean; with no ratings at all there is no
  aggregate, on the wall or in the JSON-LD.
- **Validation is the product** — a rating present but not an
  integer from 1 to 5 (0, 6, `"five"`, 4.5, `true`) is
  refused; an empty quote or a missing name is refused; dates
  must be real `YYYY-MM-DD` dates; source links must be
  http(s); avatar/logo schemes other than http(s) are refused.
  Every error names its field and exits 2 with nothing
  written. Duplicate quotes (case/whitespace-insensitive) and
  missing ratings are warnings: the page builds, exit 1.
- **Escaping everywhere** — every user-supplied string is
  HTML-escaped in text and attributes; a `<script>` in a quote
  renders as text, and the generated page contains exactly one
  `<script>` element: the inert JSON-LD data block.
- **Avatars and the logo are referenced, never embedded** —
  http(s) URLs or local paths in `<img src>`, so the page
  stays one small file.
- **Honesty by design** — the wall shows only the testimonials
  in the input file, verbatim: never invented, padded or
  paraphrased. Three testimonials in means three cards and a
  review count of three out.
- **`--json` on every command** — one JSON object with the
  same facts and exit codes as the text view; errors are
  `{"error": ...}` objects, argparse usage errors included.
- **`SKILL.md`** at the repo root for AI coding agents:
  commands, the JSON schemas, exit codes, and the honesty
  rules (never invent a testimonial; the aggregate is
  arithmetic, not copy; a warning is not an error).
- Bundled samples (`examples/`): a six-testimonial JSON wall
  for a fictional Okanagan fence-and-deck builder (ratings
  5, 5, 4, 5, 4, 5 — aggregate 4.7), a warnings sample
  (duplicate quote + missing rating, exit 1), and a
  three-row CSV sample (aggregate 4.7).
- 76 stdlib-unittest tests: the happy path in both formats,
  display order and its tie rules, aggregate math including
  half-up rounding, the JSON-LD/wall number match, the
  one-script-element contract, the external-URL scan,
  escaping in text, attributes and JSON-LD, warnings exact,
  every refusal naming its field, page-weight equality with
  the written file, and the clean-error shape in both modes.
- Pure Python standard library; local only; MIT licence.
