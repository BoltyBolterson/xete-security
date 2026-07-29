# Known limitations

Things we know are incomplete or unsafe under certain conditions, disclosed proactively
rather than waiting for someone else to find them. Entries move to
[`incident-history.md`](./incident-history.md) only if they're ever actually exploited;
until then they live here as open, tracked items.

## `xete-mcp` payment path has no client-side spend cap

**Status: open, fix designed, not yet shipped.**

The `xete-mcp` reference client (`github.com/xetenet/xete-mcp`) pays for message
sending by asking the server for an invoice, then signing and submitting a Solana
transaction for the amount the server quotes. Verified directly against
`src/xete_mcp/server.py` (`xete_send_message`) and `src/xete_mcp/payment.py`
(`pay_herd`): the paid amount comes straight from the server's invoice response
(`invoice.get("message_count", 1)`), and there is currently **no client-side maximum,
sanity check, or confirmation prompt** before the client signs and pays it.

The payment *destination* is safe — the on-chain program ID and treasury address are
hardcoded in the client itself (`payment.py` even says so in its own comment: "the
program id and treasury cannot be redirected by a malicious server"), so a bad server
can't redirect funds elsewhere. But the *amount* is entirely server-dictated. A
compromised or spoofed server, a misconfigured one, or a user who's been socially
engineered into pointing `XETE_SERVER_URL` somewhere untrusted could quote an
arbitrarily large `message_count` and the client would sign and pay it without
complaint — provided `XETE_SOL_KEYPAIR` is a funded wallet.

**Why this isn't urgent today:** the live server is currently free alpha — sending
costs nothing right now, so there's nothing for a malicious quote to actually charge.
This is a real design gap, not a real loss, as of this writing.

**What we're doing about it:** a client-side spend-cap design has been drafted
internally (a hard per-message and/or per-session ceiling enforced before signing,
independent of whatever the server claims) and is queued for implementation. This page
will be updated when the fix ships.

**If you're testing now with a funded keypair:** until the fix lands, treat
`XETE_SOL_KEYPAIR` as a hot wallet with no real protection against an over-quoted
invoice. Keep its balance low — enough to test with, not more — as a precaution rather
than because of any known exploit.
