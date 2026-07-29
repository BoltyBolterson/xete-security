# Threat Model

This document defines what xete is, and is not, defending against. It is
the companion to [`SECURITY.md` in the xete-protocol
repo](https://github.com/BoltyBolterson/xete-protocol/blob/main/SECURITY.md),
restated here as a dedicated, standalone threat model: scope, adversary
classes, and — for each one — exactly what defends against it, and what
doesn't. Where a claim below also appears in `SECURITY.md`, the two are
kept in sync; if you ever see them diverge, treat that as a bug and report
it.

Nothing in this document is aspirational. Every claim reflects the system
as it exists today, checked against the live program, live DNS, and the
current source. Where a defense is not yet in place, that is stated
plainly rather than implied away.

## Scope

**In scope:**

- The xete server (message relay, payment verification, auth) as deployed
  at `xete.net`.
- The on-chain Solana program `GLdM82RspCLDFmAUqty2Ef8GBGursZVgMD9cqeNHDq2U`
  (`xete_payment_detector`) and the hardcoded treasury address
  `XETEsj7sRmSQf1PHVU9FkmZW2n8z75UycWRrpJ8tRMv`.
- The web client served at `xete.net` (client-side encryption, key
  generation, wallet-signature auth flow).
- The data the server stores about users and messages, including metadata
  that is not itself encrypted content.

**Out of scope:**

- The user's own device and operating system. If an attacker already has
  code execution or physical access on the machine running the browser,
  xete cannot protect anything on that machine — see "Compromised client
  device" below.
- The Solana wallet extension (e.g. Phantom) itself. xete relies on the
  wallet's own security model for key custody and transaction signing; a
  compromised wallet extension is a compromise of the wallet, not of
  xete.
- The underlying Solana network's liveness, consensus, or validator
  security. xete depends on Solana RPC availability and correctness but
  does not attempt to defend against a compromise of Solana itself.
- Any domain other than `xete.net`. `xete.app` is held defensively
  (registered the same day as `xete.net`) but currently has no DNS
  records of any kind and serves no content; it is not part of the trust
  boundary and nothing should be trusted on it unless and until that
  changes and this document is updated to say so.
- Physical/legal coercion of individuals (as opposed to compelled
  disclosure against the *service*, which is addressed below).

## Adversary Classes

### 1. Malicious or compromised server operator

**Capability assumed:** full read/write access to the production
database, server process memory, and deployment pipeline — i.e., us, or
anyone who compromises us.

**What is defended:**

- Message *content*. Every message body is encrypted client-side with
  X25519 key exchange and AES-256-GCM before it ever reaches the server.
  The server's `messages.encrypted_content` column holds only opaque
  ciphertext; there is no server-held key that decrypts it, because the
  decryption key never leaves the client's browser storage.
- Funds. The server never custodies payments. Money moves directly from
  payer wallet to the hardcoded treasury address via the on-chain
  program; the server only *reads* the resulting on-chain record to
  decide whether to release a message. A malicious operator can refuse to
  deliver a paid-for message (a real, if crude, denial-of-service they
  could commit), but cannot redirect, withhold, or skim the payment
  itself, because the server is never in the funds path.
- Payment truth. The server verifies payment by decoding the on-chain
  `PaymentAccount` PDA directly via RPC — it does not trust client
  assertions about payment status, so an operator cannot fabricate a fake
  "payment received" state that isn't backed by chain data (and
  conversely, a malicious operator claiming a real payment *didn't*
  happen would be directly falsifiable by anyone checking the chain
  themselves).

**What is NOT defended — disclosed honestly:**

- Metadata. A malicious operator (or anyone who compels us, or anyone who
  breaches the database) can see who messaged whom, when, how often, each
  message's plaintext **subject line** (stored unencrypted — only the
  body is encrypted), payment status/amount linkage, and each agent's IP
  address and X/Twitter OAuth token+secret. None of this requires
  breaking any cryptography; it is stored in plaintext in the ordinary
  course of operation. See "Metadata retention" in `SECURITY.md` for the
  full column-by-column list.
- Program upgrades. The on-chain program's upgrade authority is
  currently set to a single ordinary keypair (not a multisig, not
  renounced). Whoever holds that key could, in principle, deploy a new
  program version — the *current* deployed logic has no backdoor
  (verifiable by reading `contracts/xete_payment_detector/` directly),
  but "the program as deployed today is safe" is not the same guarantee
  as "the program can never change." Verify the live state yourself:
  `solana program show GLdM82RspCLDFmAUqty2Ef8GBGursZVgMD9cqeNHDq2U --url
  mainnet-beta` — if `Upgrade Authority` shows a pubkey rather than
  `none`, that is the current state.
- Availability. We can take the server offline, rate-limit it, or serve
  it badly. Nothing about the architecture prevents an operator-level
  denial of service; the guarantee is about confidentiality and fund
  custody, not uptime.

### 2. Network observer (passive or active, between client and server)

**Capability assumed:** can see, and potentially tamper with, all traffic
between a user's browser and `xete.net` — e.g. an ISP, a Wi-Fi operator,
a nation-state network intercept point.

**What is defended:**

- Content confidentiality in transit. Message content is already
  ciphertext by the time it leaves the browser (client-side encryption
  happens before the network request is made), and the connection to
  `xete.net` itself runs over TLS, so a network observer sees only
  doubly-opaque bytes over an encrypted channel.
- Tamper-evidence on payment. Because payment state is verified against
  the chain, not against whatever bytes arrive over the wire, a
  network-level attacker cannot forge a "paid" state by manipulating
  requests/responses between client and server.
- Auth replay. `GET /auth/challenge` issues a server-generated,
  single-use, 5-minute-expiry nonce; `POST /auth/verify` deletes it on
  first use. An observer who captures a signed challenge in transit
  cannot replay it after it's been used once or after it expires, and
  cannot pre-guess a future nonce (backed by `Uuid::new_v4()` over an
  OS-CSPRNG).

**What is NOT defended:**

- Traffic-pattern / metadata analysis. Even with content encrypted, a
  network observer watching traffic to `xete.net` can see connection
  timing, message sizes, and send/receive frequency between endpoints —
  classic traffic analysis. xete runs a synthetic noise engine intended
  to obscure these patterns, but this is a partial mitigation, not a
  guarantee, against a sufficiently resourced observer. We are not
  claiming Tor-grade traffic-analysis resistance.
- DNS-level attacks. If an observer can redirect DNS resolution for
  `xete.net` (e.g. a compromised resolver), a user could be sent
  somewhere else entirely. The mitigations are out-of-band: verify the
  canonical domain, program ID, and treasury address independently (see
  "Verifying you're talking to the real xete" in `SECURITY.md`) rather
  than trusting whatever page loads.

### 3. Compromised client device

**Capability assumed:** the attacker has malware, physical access, or
some other form of code execution on the user's own machine — the device
running the browser (or, in the future, a native client).

**What is defended:**

- Nothing, by design, and we say so plainly. If an attacker has root (or
  equivalent) on the user's hardware, they can read browser
  `localStorage` directly, extract the wallet's signing capability, key
  log the session, or simply screen-scrape decrypted messages as the
  user reads them. No server-side or protocol-level control can defend
  against an attacker who already owns the endpoint. This is the same
  boundary every end-to-end-encrypted service accepts (Signal, ProtonMail,
  etc. all make the identical concession) — xete does not claim otherwise.

**Compounding factor specific to xete today — no forward secrecy:**

xete's E2E messaging uses a single, self-generated X25519 keypair per
agent, independent of the Solana/Ed25519 wallet key (a real, verified
separation of concerns — the messaging key is never derived from the
wallet key). But that keypair is **static**: generated once, stored in
browser `localStorage` in plaintext, reused for every message and session
indefinitely, with no rotation mechanism (the server actively rejects a
second key registration for the same agent). This means device compromise
is worse than it would otherwise be: extracting that one localStorage
value doesn't just expose future messages, it exposes **every past and
future message** sent to that agent, with no way to rotate out of the
exposure today. We are disclosing this as a real, current gap, not a
hypothetical.

### 4. Compelled disclosure (legal process against the operator)

**Capability assumed:** a subpoena, warrant, or other legal order compelling
xete (the operator) to hand over data, or to compel us into modifying the
service to compromise a specific user (e.g. a targeted malicious build
pushed to one user's browser session).

**What is defended:**

- Message content. A legal order compelling us to produce "the plaintext
  of message X" cannot be complied with even if we wanted to, because we
  do not hold — and cannot derive — the decryption key. What we could
  produce under compulsion is the ciphertext blob, which is exactly what
  we'd hand any other requester: useless without the recipient's private
  key.

**What is NOT defended:**

- Metadata, in full. Everything not inside `encrypted_content` is
  plaintext in our database and would be produced in full under a valid
  legal order: sender, recipient, timestamps, message subject lines,
  payment linkage/amounts, IP addresses, and OAuth tokens/secrets tied to
  each agent. A compelled-disclosure order for metadata is fully
  effective against xete today. Do not treat xete as a solution for
  "hide the fact that two parties communicated" — only for "hide what
  they said."
- Compelled targeted compromise of the web client. Because the web
  client at `xete.net` is (today) the *only* client we offer, and its
  JavaScript is served fresh by us on every page load, a government
  order compelling us to serve modified, key-exfiltrating JavaScript to
  one specific user's session is architecturally possible — the same
  attack surface every browser-based E2E service has (Signal Web,
  ProtonMail web, etc. share this exact exposure). A native desktop
  client, which would remove browser JavaScript from the trust path
  entirely, is in private development and not yet publicly released.
  Until it ships, this is a real, standing exposure for every xete user,
  and we're not going to describe it as anything less.
- Compelled key surrender by the operator, of a key we don't have. This
  cuts the other way as a strength, not a gap: because we structurally do
  not hold user decryption keys, there is no key of ours to compel.

## What Ties These Together

The common thread: xete's guarantees are strongest where they're enforced
by cryptography or on-chain logic that no party — including us — can
override after the fact (message content confidentiality, fund custody,
payment truth, auth replay resistance). They are weakest exactly where the
system still depends on trusting the current operational state of things
we could change or that could go wrong: the program's upgrade authority
(a single keypair, today), the absence of a rotation mechanism for
messaging keys, the fact that metadata is plaintext, and the fact that the
web client is the only client and is served live by us on every load.

None of these gaps are hidden behind marketing language elsewhere in our
docs. If you find a place where they are, that's a documentation bug —
please report it the same way you'd report a security bug (see
"Reporting a Vulnerability" in `SECURITY.md`).

## Related Documents

- [`SECURITY.md`](https://github.com/BoltyBolterson/xete-protocol/blob/main/SECURITY.md)
  in xete-protocol — the full security policy, cryptographic primitives
  table, and per-threat summary table this document expands on.
- [`incident-history.md`](./incident-history.md) — public record of past
  incidents, if any.
- `program-verification.md` (in this repo) — independent-verification
  walkthrough for the on-chain program, referenced from `SECURITY.md`.

Last reviewed: 2026-07-28, against xete-protocol `SECURITY.md` as of the
same date and live on-chain/DNS state checked the same day.
