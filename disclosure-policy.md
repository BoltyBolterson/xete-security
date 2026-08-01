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

- **`security@xete.net`** — the dedicated security address. Stood up and
  confirmed working **2026-07-31** (verified by SMTP `RCPT TO` against the
  live MX, which returns `250`; it previously returned `550 Address not
  found`). Use this one.
- **`john@xete.net`** also still works and reaches the same person. It was
  the sole contact until 2026-07-31 and is kept as a fallback rather than
  retired, so older copies of this document and anything already published
  pointing at it do not dead-end.

Earlier revisions of this section described `security@xete.net` as
aspirational and not yet set up. That is no longer true — it is live, and
`/.well-known/security.txt` now points at it.

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
the protocol repository's `SECURITY.md` (not publicly available)
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
- The xete server, web client, auth, messaging and payment flow
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

## `/.well-known/security.txt` (RFC 9116) — live

Served at `https://xete.net/.well-known/security.txt`, formatted per
[RFC 9116](https://www.rfc-editor.org/rfc/rfc9116). Caddy serves it with a
`file_server` handler from the site docroot. The block below is the exact
served content — if the two ever disagree, the live file is what matters
and this block is the thing that is wrong.

```
Contact: mailto:security@xete.net
Expires: 2027-06-14T00:00:00.000Z
Policy: https://github.com/BoltyBolterson/xete-security/blob/main/disclosure-policy.md
Preferred-Languages: en
Canonical: https://xete.net/.well-known/security.txt
```

Notes on the fields, so nobody has to re-derive the reasoning:

- **Contact** points at `security@xete.net` as of **2026-07-31**. It
  previously pointed at `john@xete.net` on the principle that an RFC 9116
  contact must resolve to something that actually works — `security@` did
  not exist until 2026-07-31. It does now, and was verified live before
  this change, not assumed.
- **Expires** is 2027-06-14. RFC 9116 requires the field and requires the
  file be refreshed before it passes. **There is currently no automation
  and no reminder for this** — it will go stale silently unless someone
  owns it. That is a known gap, stated rather than hidden.
- **Policy** points at this document's canonical GitHub URL so the full
  policy (safe harbor, scope, response window) is one hop away from the
  machine-readable file.
- **Canonical** is present in the served file and was not in earlier
  drafts of this block; it declares the file's authoritative location so a
  copy served from elsewhere can be recognised as not ours.

**Not integrity-monitored.** `security.txt` is not listed in `routes.txt`
or `manifest.json`, so the 15-minute drift monitor that covers the public
pages does **not** watch this file. Changing it does not require a
manifest update — and equally, tampering with it would not raise the
alarm. Worth adding to the monitored set.

## Updates

This policy is versioned alongside the other public security documents in
this repo and in `xete-protocol`. If the response-time commitment,
contact address, or safe-harbor terms change, this file and
`xete-protocol/SECURITY.md` will be updated together.

Last reviewed: 2026-07-31.
