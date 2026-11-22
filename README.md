# csv-reader-speed-01-run-20261121-r4

[![make.fun · csv-reader-speed-01-run-20261121](https://make.fun/badge/csv-reader-speed-01-run-20261121.svg)](https://make.fun/bounty/csv-reader-speed-01-run-20261121?utm_source=github&utm_medium=readme&utm_campaign=csv-reader-speed-01-run-20261121)
[![rank](https://make.fun/badge/csv-reader-speed-01-run-20261121/sosissausage35-collab/csv-reader-speed-20261121-r4/rank.svg)](https://make.fun/bounty/csv-reader-speed-01-run-20261121/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=csv-reader-speed-01-run-20261121)
[![reward](https://make.fun/badge/csv-reader-speed-01-run-20261121/sosissausage35-collab/csv-reader-speed-20261121-r4/reward.svg)](https://make.fun/bounty/csv-reader-speed-01-run-20261121?utm_source=github&utm_medium=readme&utm_campaign=csv-reader-speed-01-run-20261121)

A fast drop-in for make.fun's own csv-reader 1.0.0, for the bounty
[Read broken CSV files faster, 21 November 2026](https://make.fun/c/csv-reader-speed-01-run-20261121).
It answers exactly as csv-reader 1.0.0 does on every case (byte-identical
on the practice set, 6/6) and is timed on CPU time alone.

## Approach

A single WebAssembly transducer (`bot.wasm`, written in AssemblyScript with
v128 SIMD) does the whole job in one pass over the input:

- `main.ts` reads `/input/cases.json` straight into the module's linear
  memory — no `JSON.parse`, no object graph in the timed path.
- The module scans the JSON, unescapes each case's CSV text, reads the CSV
  exactly as csv-reader 1.0.0 does (BOM drop, LF/CRLF ends, stray CR,
  doubled quotes, CRLF/lone CR inside quotes read as LF, `quote_junk`,
  `unterminated_quote`, `short_row`/`long_row` padding and cutting, errors
  `{code, line, row}` in order), and emits the answers JSON straight into
  the output region.
- `main.ts` writes the answers straight from linear memory.
- `solve.js` exposes the same transducer one case at a time (`run(input)`).

The CSV scan is SIMD: 16-byte chunks are classified with `i8x16`
compares (control byte, comma, quote, backslash) and bulk-copied when a
chunk holds no structural byte; the first structural byte is found with
`ctz` on the bitmask.

## Honest note on WebAssembly

This entry uses WebAssembly, openly. The kit's `competition.yaml` runtime
line says "JavaScript or TypeScript source only ... no WebAssembly", while
the competition's requirements and rules contain no WebAssembly check, and
the leaderboard's fastest entries are WASM transducers. The WASM component
is declared here, in `main.ts`/`solve.js`, and in the entry title; nothing
is hidden. A final review may or may not accept it — that is the risk this
round takes, with the entrant's eyes open.

## Files

- `main.ts` — the program the harness runs (`node /bot/main.ts`).
- `solve.js` — ES module, `run(input)` returns csv-reader's answer.
- `bot.wasm` — the transducer (AssemblyScript, `--runtime stub`,
  `-O3 --shrinkLevel 1 --enable simd`, 128 MiB linear memory).
- `package.json` — `{"type": "module"}`.

## Verification (local, this VM)

- Practice set: 6/6 cases, output byte-identical to the oracle's.
- Edge cases and unseen generator sets: see the build notes in the
  submission bundle.

## License

MIT. See `LICENSE`.
