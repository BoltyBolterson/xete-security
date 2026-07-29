# Vulnerability Disclosure Policy

This document describes how to report a security vulnerability in xete, what
we commit to once you do, and what protection you have as a good-faith
researcher. It applies to the xete protocol, server, web client, and the
on-chain program at `GLdM82RspCLDFmAUqty2Ef8GBGursZVgMD9cqeNHDq2U`.

## How to report

**Do not file a public GitHub issue for a security vulnerability.** Public
issues are indexed and searchable immediately; an unpatched vulnerability
described in one is a gift to attackers, not to us.

Instead, email your report directly to a human:

- **Currently working, confirmed live: `john@xete.net`.** This is the real
  contact today. Use it.
- **Aspirational: `security@xete.net`.** We intend to stand up a dedicated
  security alias and route it to the same people who triage
  `john@xete.net`. As of this writing **`security@xete.net` is not yet set
  up** — do not rely on it. This document will be updated the moment it is
  live, and until then this paragraph stays as the honest status rather
  than being quietly deleted.

For severe issues affecting user funds or message privacy, and if you'd
prefer not to send full details over plain email, say so in your first
message and we'll arrange an encrypted channel (a PGP key or a Solana
pubkey we can reply to via an encrypted xete message) before you send
specifics.

### What to include

- A clear description of the vulnerability and its impact.
- Steps to reproduce, or a proof-of-concept, if you have one.
- The affected component (server endpoint, web client, on-chain program,
  etc.) and, if applicable, a transaction signature or account address.
- Whether you've disclosed this anywhere else, and to whom.

You do not need to be polished or formal. A working repro beats a
well-formatted report.

## Response commitment

We aim to respond within **48 hours** of receiving your report. This
mirrors the response-time commitment already published in
[`xete-protocol/SECURITY.md`](https://github.com/BoltyBolterson/xete-protocol/blob/main/SECURITY.md)
— we're not inventing a new number here, just restating the same one in
the place researchers are more likely to look for a disclosure policy
specifically.

"Respond" means acknowledgment and initial triage, not necessarily a fix.
For genuine, reproducible vulnerabilities we will keep you updated as we
work through remediation, and we'll credit you (unless you ask us not to)
once a fix ships and public disclosure is appropriate.

## Safe harbor

If you are acting in good faith to identify and report a vulnerability in
xete, we consider that research authorized, and we will not pursue legal
action or treat it as an attack, provided you stay within these bounds:

- **No data exfiltration.** Demonstrating a read vulnerability with a
  single record is enough — do not bulk-extract user data, messages, or
  keys.
- **No denial of service against production.** Don't run load tests,
  fuzzers, or anything else against `xete.net` or the mainnet program that
  could degrade service for real users. If you need to hammer something,
  do it against a local build or ask us for a scoped test environment
  first.
- **No real user funds.** Don't move, freeze, or attempt to redirect funds
  belonging to anyone other than a wallet you control, even if a bug would
  technically let you.
- **Report before you disclose.** Give us the 48-hour window (and
  reasonable time to actually remediate beyond that) before any public
  disclosure, talk, or write-up.
- **Only interact with accounts/data you control**, or synthetic data you
  created for the purpose of testing — don't touch other users' messages
  or wallets even if a vulnerability makes it possible.

Stay inside these bounds and we treat your traffic as research, not abuse,
even if it trips something on our end while you're doing it. This is a
commitment from us, not a claim that your access was pre-authorized under
every possible law in every jurisdiction — use ordinary judgment.

## Scope

**In scope:**
- `xete-protocol` (server, web client, auth, messaging, payment flow)
- The on-chain program at `GLdM82RspCLDFmAUqty2Ef8GBGursZVgMD9cqeNHDq2U`
- `xete.net` and infrastructure serving it

**Out of scope:**
- Third-party services we don't control (e.g. the Solana network itself,
  browser wallet extensions like Phantom, DNS registrars)
- Social engineering against xete team members
- Physical security
- Findings that require a compromised or rooted end-user device to
  exploit (see "What We Do NOT Guarantee" in `xete-protocol/SECURITY.md`
  — that risk is already disclosed, not something we're asking you to
  re-discover)

If you're not sure whether something is in scope, report it anyway and
say you're unsure — we'd rather triage a borderline report than miss a
real one because someone assumed it was out of bounds.

## `/.well-known/security.txt` (RFC 9116) — planned, not yet served

The content below is the intended text for
`https://xete.net/.well-known/security.txt`, formatted per
[RFC 9116](https://www.rfc-editor.org/rfc/rfc9116). **This is not live
yet.** Serving a file at that path requires a small server-side route
change in `xete-protocol` (a new static-file or handler entry), which has
not been made. Until it's wired up, this fenced block is the source of
truth for what that file *should* say — copy it in once the route exists,
rather than re-deriving it.

```
Contact: mailto:john@xete.net
Expires: 2027-07-28T00:00:00.000Z
Policy: https://github.com/BoltyBolterson/xete-security/blob/main/disclosure-policy.md
Preferred-Languages: en
```

Notes on the fields above, so whoever wires this up doesn't have to
re-derive the reasoning:

- **Contact** points at `john@xete.net`, not `security@xete.net`, because
  RFC 9116 contact fields should resolve to something that actually
  works today. Update this the day `security@xete.net` is live and
  confirmed — don't update it preemptively.
- **Expires** is set one year out from today (2026-07-28). RFC 9116
  requires this field, and requires the file be refreshed before it
  passes — whoever owns this file needs to either automate the refresh
  or put a reminder somewhere durable so it doesn't silently go stale.
- **Policy** points at this document's canonical GitHub URL so the
  full policy (safe harbor, scope, response window) is one hop away
  from the machine-readable file.

## Updates

This policy is versioned alongside the other public security documents in
this repo and in `xete-protocol`. If the response-time commitment,
contact address, or safe-harbor terms change, this file and
`xete-protocol/SECURITY.md` will be updated together.

Last reviewed: 2026-07-28.
