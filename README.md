# Receipt-shape packet: pruning and the window cap, checked on your machine

One narrow rule-set in our receipt profiles, checked on your own machine with
one command:

> At commit `ea480ea4fc7cea7c7e4b77301b14eed6c3438891` of
> https://github.com/1OF1Research/hfq001-wider-comparison, in the comparison
> harness's model of the profiles: **(X1)** under the hybrid profile — signed
> receipts carrying `iat`/`exp`, with a (signer, nonce) pair-store that may
> prune a pair once its receipt has expired — a fresh receipt that reuses a
> pruned nonce in a fresh window is accepted, and the original receipt stays
> refused on expiry; under P6, which keeps every pair, the same reuse is
> refused. **(X2)** a relying party enforcing a published maximum window of
> 300 s refuses a receipt whose `exp − iat` is 301 s or 30 days, accepts one
> at exactly 300 s, records no pair for a cap-refused receipt, and changes
> none of the thirteen settled comparison cells.

Passing these cells would not show that any deployed system behaves this way.
It would show that the profiles, as specified in `DESIGN.md` and implemented
in the harness at that commit, do what the text says — on your machine, not
ours.

The two cells were proposed by Lawless1987 on the Hugging Face forum (topic
180706, post #26, 2026-11-15). The predictions were written and committed
before the extension harness existed. Because the repository is public, the
order is checkable by anyone: predictions `f5927713` (2026-11-15T21:40:38Z),
harness `d7554879` (21:40:45Z), first results `ea480ea4` (21:40:50Z).

## Run it

You need Python 3.10 or newer and one package:

```bash
python3 -m pip install pycryptodome
git clone https://github.com/1OF1Research/hfq001-wider-comparison
cd hfq001-wider-comparison/hfq001-wider-comparison-2026-11-15
python3 compare-extension.py
```

The harness first re-verifies itself against the settled test vectors (a
gate; if it fails, nothing runs), then prints one line per cell and writes
`extension-results.json`. Success is the last line:

```
DIVERGENCES from registered predictions: none
```

with exit code 0. `expected.json` in this packet lists all 23 rows (expected
and observed) so the printed table can be compared without trusting the
harness's own summary. Our runs: 2026-11-15, two runs byte-identical, 23 of
23; 2026-11-17, a fresh run at the same commit, 23 of 23, identical to the
committed results.

## What is measured, and what is not

Measured, by the harness's own verifier outcomes: the three X1 sequences
(pruned, unpruned control, P6 control), the X2 cap cells under both timed
shapes (at the cap, one second over, thirty days, and the untouched store),
and the thirteen settled cells re-run against the capped relying party (X2d).

Not measured: a deployed service, a live chain, real keys (the harness mints
with fixed test keys in simulated time), or the question the cells leave open
on purpose — whether a nonce *should* be unique forever or only within its
window. X1 makes the current text's answer observable; the profile text is
where that decision belongs, and the hybrid profile as adopted states it:
pairs are prunable after expiry, and the maximum window is published beside
the profile, so the reuse X1 shows is the documented behaviour, not a
surprise.

`CLAIM.md` reads the claim against the harness code, mechanism by mechanism.
