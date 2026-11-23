# Notes

## What was tried

1. **v1 — faithful fast reimplementation.** The reference pipeline ported with bulk
   escaping, a fused Markdown lexer/renderer, an ASCII fast path for slugs and a
   scope-frame-per-loop-activation template renderer. Practice exact; run() ~268 ms
   (trimmed mean, 9 runs) vs baseline ~561 ms.
2. **v2 — merged inline escaping, regex escaper, manual word count, first-char block
   dispatch.** Inline rendering now escapes the five HTML-special characters as they
   are met (one pass, no second scan); `escapeHtml` uses the native regex replace
   (3.5x faster than a per-character loop on text with specials, ~200x on plain text);
   `wordsOf` counts without allocating the match array; block starts dispatch on the
   first character instead of running four checks per line. Practice exact; run()
   ~197 ms.
3. **v3 — table-dispatched inline scan.** The inline walk dispatches on a 128-entry
   table of the characters that matter instead of a chain of comparisons. Practice
   exact; run() ~178 ms, about 3.1x the baseline's ~561 ms on the same harness.
   Whole `main.ts` run (incl. JSON parse of the 2.9 MB input and stringify of the
   7.4 MB output plus Node start-up, wall ms, median of 5): entry ~730 ms, baseline
   ~950 ms.

## Verification

- Practice set: 3/3 cases byte-identical to the kit's `expected.json`.
- `grade-local --driver process`: 3 of 3 cases, `passed: 1`, L1 regrade reproduced
  byte for byte. (The grader notes the local Node v24.22.1 differs from the server
  harness v26.13.1; that changes only the recorded result hash, not the score.)
- Differential fuzz against the reference with the kit's own generator under 15 fresh
  seeds: 45/45 cases exact.
- Adversarial differential suite (110 hand-built cases: Markdown torture, front
  matter refusals, template errors, self-includes and include cycles, slug
  collisions, CRLF, reserved dirs, shadowed statics, tag ordering, per_page 1,
  empty site): 110/110 exact.
- Two direct runs of `main.ts` are byte-identical.
- Repo size 46,413 bytes total — far under the 4 MiB cap.

## Timing (this machine, Node 24.22.1)

- run() only, 3-case practice set, trimmed mean of 9 runs: entry ~178 ms, baseline
  ~561 ms (~3.1x). Per case: g-0001 ~31 ms, g-0002 ~44 ms, g-0003 ~105 ms
  (baseline ~115/~198/~240 ms). The shared-CPU VM is noisy; repeated runs move
  ±10%.

## Kit download

The kit tarball answered 503 `kit_pending` ("This kit is being built") for about
an hour of retries (from ~00:48 UTC) before it landed at 01:30:28 UTC (10,581,394
bytes). Sibling 22 Nov kits (tiny-stack-vm, best-heist-game) had already built;
hidden-spec-cli was still pending at the same time.
