# Build a static site faster — site-generator-speed-01-run-20261122

A faster drop-in for makefun-site 1.0.0: `solve.js` (a dependency-free ES module)
whose `run(input)` returns exactly the output tree makefun-site's `build` returns for
one site `{ site, templates, files }`, and `main.ts`, the program the harness runs,
which reads every case from `/input/cases.json`, answers each with `run`, and writes
`/output/answers.json`.

## How it is faster

The reference spends its time in three places — Markdown inline rendering (one
`escapeHtml` regex call per character), template rendering (a scope object and a
closure per loop item), and `slugify` (Unicode normalisation on every title and tag).
This entry keeps every rule and changes how it is computed:

- **HTML escaping** scans for the next special character and copies literal runs in
  bulk instead of escaping one character at a time.
- **Markdown** lexes and renders in one fused pass, checking block starts by hand
  instead of running several regular expressions on every line; inline rendering walks
  a string once and flushes literal runs through the bulk escaper.
- **Slugs** take an ASCII fast path (NFKD is the identity on ASCII) and are memoised
  per case — titles, tag names and headings repeat a lot.
- **Templates** reuse one scope frame per loop activation (mutating the loop variable
  and the loop record per item) instead of allocating a frame and a closure per item,
  and templates that can only read the case-level values (`site`, `tags`, `recent`)
  are rendered once per case instead of once per page.

Nothing is cached from one case to the next: every per-case table is built inside
`run()`.

## Verification

- Practice set: 3/3 cases byte-identical to the kit's `expected.json`.
- Differential fuzz against the reference with the kit's own generator under fresh
  seeds: see NOTES.md.
- Two direct runs of `main.ts` are byte-identical.
