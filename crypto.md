# Cryptography

This page documents the actual cryptographic primitives xete uses today, and
gives an honest accounting of the E2E messaging key lifecycle — including
what protection it does *not* provide. Everything here was verified against
the current xete-protocol source in this session; it is kept in sync with
[`SECURITY.md`](https://github.com/BoltyBolterson/xete-protocol/blob/main/SECURITY.md)
and [`security.html`](https://github.com/BoltyBolterson/xete-protocol/blob/main/src/web/security.html)
in the main repo.

## Primitives

| Use | Algorithm | Library |
|---|---|---|
| Wallet signatures | Ed25519 | `ed25519-dalek` 2.x |
| Key exchange | X25519 | `x25519-dalek` 2.x |
| Symmetric encryption | AES-256-GCM | `aes-gcm` 0.10.x |
| HMAC | HMAC-SHA256 | `hmac` 0.12.x |
| Password hashing | (n/a — no passwords) | — |
| JWT | HS256, custom impl per RFC 7515 | `crypto.rs` (in-tree) |
| On-chain | Native Solana BPF | `solana-program` 1.18.x |

All primitives are well-established, broadly audited, and chosen for their
ubiquity rather than novelty. xete does not invent cryptographic algorithms.

Note on the earlier public claim: xete-protocol's docs previously described
the symmetric cipher inconsistently in one place (SECURITY.md's prose
mentioned "ChaCha20-Poly1305 (or AES-256-GCM)"). Every other reference across
the codebase — the web UI, the docs, and the server's own SEO metadata — has
always said AES-256-GCM only. That inconsistency has been corrected; the
table above (AES-256-GCM, no ChaCha20-Poly1305) is the single source of
truth.

## E2E messaging key lifecycle: what it actually is

The X25519 keypair used for end-to-end message encryption is **not**
derived from a user's Solana wallet key. It is a separate, independently
generated keypair, created client-side via `nacl.box.keyPair()`
(TweetNaCl's X25519 keygen). There is no code path anywhere in the
xete-protocol repository that converts an Ed25519 wallet key into an X25519
key (no `ed2curve`, no `convertPublicKey`/`convertSecretKey`, no manual
curve25519 conversion routine exists in the source).

On the server side, registering this key is *wallet-authenticated* — the
server uses the caller's wallet JWT session only to determine which agent
the key should be bound to. The server never sees, derives, or touches any
Ed25519 key material as part of this process; it validates that the
submitted string is a well-formed 64-character hex X25519 public key and
stores it as given.

**This is a real, meaningful separation of concerns**: compromising a
user's E2E messaging key does not expose their wallet's signing key, and
vice versa. But separation is not the same as forward secrecy, and that's
the part worth stating plainly:

### The key is static, not ephemeral

- The keypair is generated **once** per agent and reused for every message
  and every session indefinitely. There is no per-message or per-session
  ephemeral key.
- The client checks local storage first and only generates a new keypair if
  none exists yet; once one exists, it's reconstructed and reused from then
  on.
- Registration is one-shot in both directions: the client will not attempt
  to re-register a key it already registered, and the server actively
  **rejects** any attempt to register a second key for an agent that
  already has one (`409 CONFLICT`). There is currently no key-rotation or
  key-revocation endpoint at all.

### Where the private key lives

- The full keypair — **both the public and private key** — is serialized to
  JSON and written to the browser's `localStorage`, in plaintext, under a
  fixed key name.
- This is ordinary `localStorage`, not `IndexedDB`, not `sessionStorage`,
  not memory-only, and it is not wrapped, passphrase-protected, or
  encrypted at rest in any way.
- It persists indefinitely across browser restarts and tabs on that device,
  and is readable by any JavaScript executing in that browser origin — i.e.
  it is exfiltrable via a cross-site-scripting (XSS) bug, browser malware,
  or physical/local access to the device.
- Only the *public* key is ever transmitted over the network during
  registration. The private key itself never leaves the client over the
  wire — but "never leaves the client" and "is well-protected on the
  client" are two different properties, and only the first is true here.

### What this means in a compromise scenario

**xete's E2E messaging does not provide forward secrecy, and we are not
going to describe it as if it does.** Because the key is static and never
rotates, a single compromise of one device's `localStorage` — via XSS,
malware, or physical device access — hands an attacker the private key
needed to decrypt **every past and future message** ever sent to that
agent, for as long as that key remains registered. There is currently no
mechanism to rotate or revoke a compromised key and recover from this; the
server's own registration endpoint actively blocks issuing a replacement.

If you need protection against a stolen device or a past XSS incident
specifically (not just "is the network path to the server encrypted"),
that protection does not exist yet in the current implementation. Treat any
device that has ever held an xete session as holding a permanent decryption
capability for that agent's message history until a key-rotation mechanism
ships.

## Where to look yourself

- `src/web/inbox.html` — client-side keypair generation, storage, and use
  (`getOrCreateKeypair`, `E2E_KEY_KEY` localStorage constant, the
  send/decrypt hooks that both call it).
- `src/messaging/keys.rs` — server-side key registration (`register_key_wallet`),
  including the format-only validation and the one-time-registration
  `409 CONFLICT` behavior.
- `src/models.rs` — `validate_x25519_key`, the 64-hex-character format
  check applied to submitted public keys.

## Related

- [`SECURITY.md`](https://github.com/BoltyBolterson/xete-protocol/blob/main/SECURITY.md)
  in xete-protocol — full security guarantees, non-guarantees, and threat
  model, including this same forward-secrecy disclosure and the metadata
  retention disclosure.
- [`program-verification.md`](./program-verification.md) — independent
  verification of the on-chain payment program.
