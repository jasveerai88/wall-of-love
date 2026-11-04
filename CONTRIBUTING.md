# Contributing

Contributions are welcome — bug reports, fixes and small focused features.

## Ground rules

- **Standard library only.** wall-of-love's promise is "clone
  and run". Pull requests that add a third-party dependency will
  not be merged.
- **Local only, and the tool never fetches.** No network calls,
  no telemetry, no uploads. The tool reads one testimonials
  file and writes one HTML file; that is the whole surface.
- **Never invent a testimonial.** The wall renders exactly the
  testimonials in the input file, verbatim. Features that pad,
  merge, summarise, "polish" or generate quotes change what the
  tool *is* and will not be merged, at any quality bar. The
  honesty rules in SKILL.md are the product.
- **The aggregate stays arithmetic.** Mean of the ratings
  present, one decimal, half-up; review count = testimonials
  in the file. No weighting, no recency boost, no "display
  rating" override. The wall, the preview, the `--json` payload
  and the JSON-LD carry the same numbers because they are
  computed once — keep it that way.
- **The page makes no request the input didn't ask for.** The
  generated HTML may reference only the configured logo,
  avatars and source links. No web fonts, no CDN assets, no
  analytics, and no JavaScript — the single `<script>` element
  is the inert JSON-LD data block, and the test suite counts
  script tags on purpose.
- **Escaping is not optional.** Any new place user text reaches
  the page goes through the same HTML escaping, in text and in
  attributes, and gets a test with markup in the input. The
  JSON-LD keeps its unicode-escape treatment for `<`, `>` and
  `&`; a raw `</script>` in a quote must stay impossible.
- **Ratings stay strict.** A rating that is not an integer
  from 1 to 5 is an error, never a silent clamp or a default
  of 5. A *missing* rating stays a warning — real testimonials
  collected on paper exist, and the wall shows them without
  stars rather than refusing them.
- **Both directions tested.** A behaviour change needs a test
  where good input succeeds and bad input fails cleanly — not
  just one side. Aggregate expectations are computed by hand in
  the test comments, never copied from the tool's own output.

## Getting started

```bash
git clone https://github.com/jasveerai88/wall-of-love
cd wall-of-love
python3 -m unittest discover -s tests -v
python3 src/wall_of_love.py preview examples/sample-testimonials.json
```

## Pull requests

1. Fork, branch, and keep the change to one topic.
2. Add or update tests under `tests/`; the full suite must pass.
3. Note the change in `CHANGELOG.md` under an "Unreleased" heading.
4. Describe the behaviour change with a real before/after run if
   the CLI output changes.
