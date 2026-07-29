# Program verification

This page exists so you don't have to take our word for anything on this page. Every
claim below is something you can check yourself, right now, against live Solana
mainnet-beta state, using nothing but a terminal or a block explorer. If you find that
reality has diverged from what's written here, please open an issue — this document
is only useful as long as it's accurate.

Last verified: 2026-07-28.

## The deployed program

- **Program ID:** `GLdM82RspCLDFmAUqty2Ef8GBGursZVgMD9cqeNHDq2U`
- **Network:** Solana mainnet-beta
- **Owner:** `BPFLoaderUpgradeab1e11111111111111111111111` (the standard upgradeable
  BPF loader)
- **Executable:** yes

Verify it yourself with the Solana CLI:

```
solana program show GLdM82RspCLDFmAUqty2Ef8GBGursZVgMD9cqeNHDq2U --url mainnet-beta
```

This prints the program's owner, its ProgramData address, the slot it was last
deployed at, and — critically — its **upgrade authority**. Read that output
yourself rather than trusting a summary; that's the entire point of this page.

If you don't have the Solana CLI installed, you can get the same information from
any Solana-aware block explorer by searching the program ID above, or via a raw RPC
call:

```
curl https://api.mainnet-beta.solana.com -X POST -H "Content-Type: application/json" -d '
  {"jsonrpc":"2.0","id":1,"method":"getAccountInfo","params":["GLdM82RspCLDFmAUqty2Ef8GBGursZVgMD9cqeNHDq2U",{"encoding":"jsonParsed"}]}
'
```

## Current status: upgradeable, single-key authority

**As of 2026-07-28, this program is upgradeable, and its upgrade authority is a
single ordinary keypair — not a multisig, and not renounced.**

Running the command above will show an upgrade authority address on the program's
ProgramData account. That means whoever holds the private key for that authority
address can deploy new code to this program ID at any time, unilaterally, with no
second signer and no timelock. This directly contradicts language that has appeared
elsewhere in xete's public materials (e.g. "no upgrade path... once deployed, the
rules are physics") — that language was wrong and is being corrected. We would
rather you hear the accurate version from us, with a command you can run to confirm
it, than discover the discrepancy on your own.

We are not going to name the authority address or discuss what else that key
controls on this page. What matters for your trust decision is the fact you can
verify directly: **this program's on-chain rules are not yet immutable, and are not
yet protected by a multisig.** Treat the program today as "upgradeable by a single
operator," not as an immutable, physics-guaranteed contract. Anyone extending it
real trust for real money should weigh that fact accordingly.

## What "fixed" will look like, and how you'll know

The plan is to move this program to one of:

- an immutable deployment (upgrade authority set to `None`), or
- a multisig-controlled upgrade authority (e.g. Squads or equivalent), with a
  public timelock on upgrades.

Neither of those changes has happened yet. This is a known, tracked gap, not a
surprise we're hoping nobody notices. When it changes, this document will be
updated with the new state, and you'll be able to confirm it with the exact same
`solana program show` command above — the upgrade authority field will either read
`None` (immutable) or point at a multisig program account (verifiable by checking
that address's owner). We won't ask you to trust an announcement; we'll point you
back at this same command.

## The treasury address

- **Treasury address:** `XETEsj7sRmSQf1PHVU9FkmZW2n8z75UycWRrpJ8tRMv`

This is the hardcoded destination address for payments processed by the program
above. You can check its balance and full transaction history on any Solana block
explorer, for example:

- https://solscan.io/account/XETEsj7sRmSQf1PHVU9FkmZW2n8z75UycWRrpJ8tRMv
- https://explorer.solana.com/address/XETEsj7sRmSQf1PHVU9FkmZW2n8z75UycWRrpJ8tRMv

Or via RPC:

```
solana balance XETEsj7sRmSQf1PHVU9FkmZW2n8z75UycWRrpJ8tRMv --url mainnet-beta
```

**As of 2026-07-28, this address has a balance of zero and has never received a
single lamport.** In other words: no real mainnet payment has landed through this
program in production yet. We're stating that plainly rather than implying
production traffic that doesn't exist. Check the balance and the (empty)
transaction history yourself using either link above — an empty account with no
transaction history is exactly what you should expect to see right now, and if you
ever see otherwise, that's a signal this page is stale and needs an update.

## Why this page exists

An external security review of xete raised, as its top finding, that claims about
this program's immutability could not be independently verified from the public
docs alone. That finding was correct, and the underlying claim was wrong: the
program is not immutable today. Rather than walk the claim back quietly, we're
publishing the exact commands needed to check the real on-chain state, so that
"trust us" is never required — you can verify the program ID, the upgrade
authority, and the treasury's payment history directly against mainnet, at any
time, including well after this page was written.
