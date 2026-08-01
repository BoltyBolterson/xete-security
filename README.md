# xete security & safety

Public, audited record of how xete protects users and their funds: security practices,
audit history, and the safety posture of the code and infrastructure. This is the
"front of the curtain" — everything here has been reviewed for public release before
it's added; nothing internal, unresolved, or sensitive goes here.

For internal architecture and reasoning docs, see the private companion repo
(not public — invite-only).

## Contents

- [`threat-model.md`](./threat-model.md) — what xete defends against and what it
  doesn't, broken out by adversary class (malicious/compromised server operator,
  network observer, compromised client device, compelled legal disclosure), with
  concrete, currently-true claims for each.
- [`program-verification.md`](./program-verification.md) — independent-verification
  walkthrough for the on-chain payment program: exact commands to check the program
  ID, upgrade authority, and treasury balance yourself against live mainnet, with no
  need to take xete's word for any of it.
- [`crypto.md`](./crypto.md) — the actual cryptographic primitives xete uses today
  (Ed25519, X25519, AES-256-GCM, HMAC-SHA256), plus an honest accounting of the E2E
  messaging key lifecycle, including that it is static with no rotation mechanism.
- [`testing-and-assurance.md`](./testing-and-assurance.md) — results of an adversarial
  test suite run against the on-chain payment contract, and an equally direct list of
  what assurance work (fuzzing, CI-gated tests) hasn't happened yet.
- [`disclosure-policy.md`](./disclosure-policy.md) — how to report a security
  vulnerability, our response-time commitment, safe-harbor terms for good-faith
  researchers, and scope.
- [`incident-history.md`](./incident-history.md) — public record of past incidents
  (none to date), with a standing commitment to log future ones here and never edit
  away a past entry.
- [`known-limitations.md`](./known-limitations.md) — open, tracked gaps that aren't
  incidents (nothing has been exploited) but aren't fixed yet either — currently: the
  `xete-mcp` reference client has no client-side cap on server-quoted payment amounts.

Status: content is live and reviewed for public release; each entry is kept in sync
with claims made in the main protocol repository
repo (notably `SECURITY.md`) and updated as the underlying system changes.
