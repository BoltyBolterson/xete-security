# Testing & assurance

What we've actually verified about the on-chain payment contract
(`xete_payment_detector`), and what we haven't yet. Both halves matter — this page
exists so we don't have to claim more than we've earned.

## Adversarial audit of the payment contract

We ran a dedicated adversarial test suite against the deployed contract on a local
validator, purpose-built to try to break the one economic guarantee that matters:
**money should only ever move payer → treasury, and never backward.** The suite
tracked the treasury balance across the entire run and asserted it never decreased
at any step, then threw a range of malformed, adversarial, and boundary inputs at
the contract.

**Result: no anomalies found. The one-way invariant held in every case.** Across the
whole run the treasury balance only ever went up; the attacker/deployer side only
ever went down. No case produced a refund, a redirect, a double-spend, a free
delivery, or corrupted state.

What was tested:

- **Malformed instruction data** — empty payloads, truncated strings, trailing
  garbage, empty nonces, oversized nonces, zero blob counts, and blob counts past
  the allowed maximum. All rejected before any funds moved.
- **Account and privilege attacks** — attempting the call without the payer as a
  signer, omitting the system program, supplying a non-derived (wrong) PDA, marking
  the treasury account non-writable, and substituting a fake-but-executable stand-in
  for the system program. All rejected; no payment account was created and no
  transfer occurred in any of these cases.
- **Economic abuse** — overpaying (funds land, no refund is issued for the excess);
  underpaying (funds land, but the server-side verification step correctly refuses
  to deliver on an underpaid invoice, since the amount is below what was invoiced);
  and then attempting to recover by re-paying the same nonce correctly, which is
  rejected outright because that nonce's on-chain account already exists. Net effect
  in every economic-abuse case: the payer's funds still ended up in the treasury,
  with no path to a refund or a double-charge/recovery workaround.
- **Griefing** — an attacker paying against a *victim's* invoice nonce. The
  transaction succeeds on-chain, but the resulting record correctly attributes the
  attacker (not the victim) as sender, so server-side verification refuses to treat
  it as the victim's payment and nothing is delivered on the victim's behalf. The
  attacker simply burns their own funds (transfer amount plus network/rent costs)
  for no benefit — the treasury still only gains, never loses or misattributes in
  the victim's favor.
- **Insufficient-funds / "free delivery" attempts** — trying to get a large invoice
  fulfilled by a payer who doesn't have the funds to cover it. Because account
  creation and the fund transfer happen atomically in the same instruction, there is
  no way to end up with a valid, server-acceptable payment record without the funds
  actually landing first. No free delivery is possible.
- **Boundary cases** — the minimum and maximum allowed batch sizes were both
  exercised and both succeeded at exactly the expected fee, with no off-by-one
  behavior at either edge.

The overarching, proven invariant: **the treasury balance never decreases, and
funds only ever flow payer → treasury.** Every misuse path we found ends the same
way — the worst thing an attacker (or a confused/malfunctioning client) can do to
themselves is lose the fee they paid. There is no discovered path to a refund
exploit, a redirect, a double-spend, or a corrupted payment record.

This audit was run against the contract as deployed on a local validator running
the same runtime version used in production. We consider the logic-level result
applicable to the live mainnet deployment as well, since the contract's on-chain
behavior doesn't depend on which network it's running against — but we haven't yet
re-run this specific suite directly against mainnet, and we say so plainly rather
than implying we have.

## What's not yet in place

We want to be equally direct about the gaps, because "we tested it once and it
passed" is a different — and weaker — claim than "this is continuously verified."

- **No fuzzing harness.** The adversarial suite above is a fixed, hand-written set
  of attack scenarios, not a fuzzer generating and mutating novel inputs. It's good
  at confirming the cases we thought to test; it can't find the ones we didn't
  think of.
- **No CI-gated invariant tests.** Our continuous integration pipeline currently
  runs `cargo test` on every push, but test failures are explicitly non-blocking —
  the build can go green even if the test suite fails. In other words, we have
  tests, but nothing today stops a regression from merging if it breaks one. The
  economic-invariant checks described above are also not part of the automated CI
  run at all yet; they were a manual, one-time exercise.

Both of these are on our roadmap, not swept under the rug:

1. Make `cargo test` a required, blocking check in CI so a broken test can't merge.
2. Port the adversarial economic-invariant suite into an automated, CI-runnable
   form so the "treasury never decreases" property is checked on every change, not
   just once by hand.
3. Add a fuzzing harness targeting the contract's instruction parsing and account
   validation logic, to go beyond the scenarios we thought to write by hand.

Until those land, treat this page as an accurate record of a real, thorough
one-time adversarial pass — not as a claim of continuous, automated assurance. We'll
update this page (not rewrite history on it — see our [incident history](incident-history.md)
policy for how we handle updates to safety claims) as each roadmap item ships.
