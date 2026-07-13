# Solius — Signal Attestation

A **tamper-evident, third-party-verifiable record of when each day's trading
signals existed** — published here before their fill bar, every trading day, as
an unbroken cryptographic chain.

This repository contains **only hash records** — no signals, no models, no code.
Each file is a SHA-256 fingerprint of a day's signal roster plus the metadata
that links it to the day before. GitHub's server-side commit timestamp anchors
*when* each fingerprint was published.

## What this proves — and what it doesn't

**It proves _when_:**

- Each day's signals were sealed and published at an externally-anchored time,
  before anyone could have acted on them.
- The full daily roster was recorded — including demotions — with no
  survivorship pruning (a running count of the internal candidate pool is
  attested too).
- The chain is continuous: every link commits to the previous one, so no day
  can be silently dropped, inserted, or reordered after the fact.

**It does _not_ prove _how well_.** The public record is a hash only. The
signals themselves are not here — they stay private until a deliberate reveal in
due diligence, where the sealed payloads are handed over and re-hashed against
this chain. This record is about *timing and integrity*, never performance.

## How it works — commit-then-reveal

Each trading day, the live pipeline:

1. Builds a canonical JSON payload of that day's signal roster (promoted +
   demoted models, ensembles, strategies, config hashes, pool counts,
   transitions).
2. Computes `sha256 = SHA-256(payload_bytes)`.
3. Embeds the previous link's hash as `prevSha256` — forming an append-only
   hash chain.
4. Commits a small hash-record file to this repository.

A git commit hash cryptographically commits to the *entire history behind it*,
so a commit link you saved months ago is self-authenticating: if any earlier
record were rewritten, that saved hash would no longer resolve to consistent
content. Branch protection on `main` blocks force-pushes and deletions, so the
published history is append-only even to us.

## Record format

Files are committed at `attestations/YYYY/MM/DD-seqN.json`, canonically
serialized (sorted keys) so a re-publish is byte-identical:

```jsonc
{
  "seq": 143,                  // monotonic chain position (genesis = 1)
  "tradingDate": "2026-07-09", // the closed bar's date, UTC (data-time)
  "sha256": "ab34…",           // SHA-256 of the day's sealed payload
  "prevSha256": "9f2c…",       // previous link's hash (null on genesis)
  "schemaVersion": 1           // payload schema version
}
```

A **revision** link — a later re-attestation of an already-attested date —
additionally carries `"revisesTradingDate": "YYYY-MM-DD"`.

## How to verify

**The commitment timeline (anyone, with just a link):**

- Open any record's commit. GitHub shows the file that was added and the
  server-side timestamp it was committed at.
- Walk the chain: each record's `prevSha256` must equal the prior record's
  `sha256`, with `seq` increasing by one and no gaps. An unbroken chain from
  `seq: 1` to today is the proof that the timeline can't have been fabricated
  after the fact.

**Full verification (in due diligence):** we provide the sealed payloads and a
standalone verifier. It re-hashes each payload, confirms it matches the
`sha256` published here, checks chain continuity end-to-end, and re-derives
performance from public market data — including a conservative delayed-fill mode
for continuous (crypto) markets.

## No proprietary content lives here

By design, this repository never contains signal values, model configurations,
or any construction detail — only hashes and chain metadata. Publishing it in
full leaks nothing about how the signals are produced.

## Learn more

- Verify page: <https://soliusalpha.com/verify>

## License

Released under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/)
(public-domain dedication) — these are factual public records; you are free to
clone, mirror, archive, and cite them without restriction. That's the point.
