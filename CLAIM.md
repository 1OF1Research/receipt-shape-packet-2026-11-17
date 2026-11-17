# The claim, read against the code (receipt-shape packet)

Repository `1OF1Research/hfq001-wider-comparison`, commit
`ea480ea4fc7cea7c7e4b77301b14eed6c3438891`. The claim's code is the folder
`hfq001-wider-comparison-2026-11-15/`: `compare.py` (the profiles, the
verifier, minting) imported unchanged, and `compare-extension.py`, which adds
exactly the two mechanisms under test by subclassing `compare.RP`.

## The profiles, as the code holds them

Four receipt shapes share one byte and signature layer. P6 keeps a
(signer, nonce) pair-store and no time fields. The hybrid (HYB) is P6 plus
signed `iat`/`exp` and one licence, stated in `DESIGN.md`: a recorded pair
may be pruned once its receipt's `exp` has passed. The hybrid is the house
receipt profile as adopted 2026-11-15: P6 + signed `iat`/`exp`, pairs
prunable after `exp`, leeway published beside the maximum window.

## X1 — what carries it

`PruningRP` (`compare-extension.py`) remembers each recorded pair's receipt
`exp` and its `prune(now)` drops pairs whose `exp` has passed — the DESIGN.md
licence, nothing else. The cell presents a first receipt, prunes at the first
licensed moment, then presents a fresh receipt reusing the nonce in a fresh
window, then the original again. Expected and observed:
**accept / accept / reject** — the pruned nonce is free for a fresh receipt,
and pruning does not revive the old one, which is refused on its own expiry.
The unpruned control (**accept / reject**) isolates pruning as the cause.
The P6 control (**accept / reject**) shows the shape that never prunes
refusing the same reuse. What would prove it wrong: the pruned arm refusing
the fresh receipt (some other rule carrying nonce history), or P6 accepting
it (the store not doing what the profile says).

## X2 — what carries it

`CappedRP` adds one rule inside clause (d)'s time checks: reject when
`exp − iat` exceeds the relying party's published cap, 300 s, with the cap
named as the cause; "exceeds" is strict, so exactly 300 s passes. Expected
and observed, under both timed shapes (EXP and HYB): at the cap **accept**;
one second over **reject**, cause the cap; thirty days **reject**, cause the
cap; and under HYB a cap-refused receipt records no pair (state changes only
on accept). X2d then re-runs the thirteen settled cells of the original suite
against the capped relying party: every outcome identical to the uncapped
run, because every mint in the suite sits exactly at the cap. What would
prove it wrong: the at-cap cell refusing, an over-cap cell accepting, or any
settled cell moving.

## The border

This is a claim about a specification and its model implementation, at one
commit, in simulated time with test keys. It is not a claim about a deployed
service. The profiles' byte layer is the same one exercised by the settled
vector suite (the harness gates on it before running), and the open question
X1 leaves — unique forever, or unique within a window — is a profile-text
decision, recorded in `VERDICT.md` and the adoption amendment, not something
a run can settle.
