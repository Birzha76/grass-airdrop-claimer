# Grass Claimer: Season 2 airdrop claim tool (Solana / USDC)

**English** · [Русский](README-RU.md) · [中文](README-ZH.md)

Windows desktop app that claims the **Grass Season 2 airdrop** for a whole list of
accounts and sweeps the **USDC on Solana** to one address you control. One `.exe`,
no command line, no config files: everything is set in the window.

![Grass Claimer, Claim page](docs/screenshots/en/claim.png)

Grass pays Season 2 to an **embedded wallet** created inside your Grass account,
not to the wallet you linked. Claiming by hand means logging into every account,
exporting that wallet, and signing a Solana transaction for each one. This app does
that for the whole list: log in, export the wallet key in memory, build the claim,
simulate it, send it, sweep the USDC to your address.

---

## Contents

- [What you need](#what-you-need)
- [Quick start](#quick-start)
- [Step by step](#step-by-step)
- [Account list format](#account-list-format)
- [What it costs](#what-it-costs)
- [Interface languages](#interface-languages)
- [What is stored on your machine](#what-is-stored-on-your-machine)
- [Troubleshooting](#troubleshooting)
- [FAQ](#faq)

---

## What you need

| What | Why |
|---|---|
| **A sponsor wallet with SOL** | pays the network fee and fronts the rent. About **0.0014 SOL per account** is spent for good, plus about **0.0065 SOL once** for the first-run setup |
| **A captcha service key** (2captcha, anti-captcha or a compatible one such as CapSolver) | needed to log in to an account and to reset a password |
| **Access to the account mailboxes** | Grass emails a code before it lets the embedded wallet be exported. Without that code a claim is impossible |
| **Proxies** | every request to Grass goes through them, they are mandatory |
| **A Solana RPC endpoint** | a public one is filled in by default and works. On a long list a private RPC (Helius, Triton, QuickNode) is faster, because public nodes throttle requests |

You do **not** need the Grass password for every account. If a password is missing
the app resets it through the mailbox and remembers the new one. That costs two
extra emails and one extra captcha per account, so supply passwords when you have them.

## Quick start

1. Download `GrassClaimer.exe` from [Releases](../../releases) and put it in its own
   folder. The app writes its database and logs next to itself.
2. Run it. Pick your language in the top card of **Settings**.
3. Fill in **Settings**: sponsor private key, collector address, captcha key, mailbox,
   proxies. Press every **Check** button, they answer immediately.
4. Open **Accounts**, paste the list, press **Apply list**.
5. Open **Claim**, press **Check everything**. When every row is green, press
   **Start the claim**.

## Step by step

### Settings

![Settings page](docs/screenshots/en/settings.png)

| Field | What to put in |
|---|---|
| **RPC endpoint** | leave the default public node, or paste your private RPC URL |
| **Sponsor private key** | base58 private key of the wallet that pays the fees. It only pays, it never receives the airdrop |
| **USDC collector address** | where the claimed USDC is swept. Leave empty to use the sponsor address |
| **Minimum claim amount** | accounts with a smaller allocation are skipped. Default 0.20 USDC |
| **Captcha** | provider and API key. The Custom API host field takes any 2captcha compatible service |
| **Email** | pick the mode that matches your setup (see the table below) and fill in the mailbox |
| **Proxies** | one per line: `user:pass@host:port`, `host:port`, `host:port:user:pass` or `socks5://user:pass@host:port` |
| **Accounts at a time** | default 1. The mailbox is the bottleneck, not Solana: many providers allow very few simultaneous IMAP sessions. 3 to 5 is usually the practical ceiling |

Press **Save settings** at the end.

### Accounts

![Accounts page](docs/screenshots/en/accounts.png)

Paste the list and press **Apply list**. The hint above the box shows the exact line
format for the email mode you selected. The table below shows what is stored: that
list, not the paste box, is what a run processes, and it survives a restart.

### Claim

**Check everything** runs read-only checks: settings, account list, proxies, RPC,
sponsor balance, address table, captcha balance, mailbox. Nothing is spent by them.
The Start button unlocks only when all of them pass.

Every transaction is **simulated on-chain before it is sent**. If the simulation
fails, that account is marked with an error and skipped, and nothing is spent.
Results appear in the table and in the log, and **Export to CSV** saves the report.

## Account list format

The expected line depends on the email mode selected in Settings:

| Email mode | Line |
|---|---|
| Shared mailbox (all addresses forward into one box) | `email` or `email:password` |
| Own IMAP for each account | `email:password:imap_login:imap_password` |
| mail.tm | `email:mailbox_password` |
| Enter the codes by hand | `email` or `email:password` |

`password` is always the **Grass account password**, not the mailbox one. Leave it
empty if you do not know it: `email::imap_login:imap_password` works, and the app
will set a password itself.

For iCloud, Gmail, Outlook and similar providers use an **app-specific password**,
not the main account password, and make sure IMAP is enabled in the mailbox settings.

## What it costs

- **Solana network:** about **0.0014 SOL per account** does not come back (rent of
  the on-chain record that marks the claim as taken). The token account rent is
  returned inside the same transaction.
- **One-time setup:** about **0.0065 SOL** for the address lookup table and the two
  token accounts. Paid once per machine, on the first run.
- **Service fee:** **0.07 USDC** per successful claim, taken inside the same
  transaction. Accounts whose allocation is smaller than the fee are skipped
  automatically and recorded in the log. No fee is taken for a claim that fails.

The pre-flight check adds this up for your list and refuses to start if the sponsor
wallet is short.

## Interface languages

English, Russian and Chinese. **English is what a fresh install starts in**,
whatever the language of your Windows. Switch it any time in the first card of
**Settings**, and the choice is remembered.

![Language selector](docs/screenshots/zh/settings.png)
 Every message,
including the ones in the log, follows the selected language. The language cannot be
switched while a run is in progress: stop it first.

## What is stored on your machine

Next to the executable the app creates `claimer.db` and a `logs` folder.

The database holds the accounts, their passwords, session tokens and the claim
results. Passwords and keys are **encrypted with a key derived from this machine**,
so the file is useless on another computer.

**Private keys of the embedded Grass wallets are never written to disk.** They exist
only in memory while one account is being processed. That is a deliberate trade: if
a run is interrupted after the export, Grass will email a new export code next time,
and nobody can harvest wallet keys out of the database.

Nothing is sent anywhere except Grass, your captcha provider, your proxies and the
Solana RPC.

## Troubleshooting

| Message | What to do |
|---|---|
| `Short by ... SOL` | top up the sponsor wallet |
| `the ... did not arrive in N attempts` | check mailbox access: app-specific password, IMAP enabled, correct server. Lower the accounts-at-a-time value |
| `the allocation of ... is below your threshold` | not an error: the payout is too small. Lower the minimum in Settings if you still want it |
| `the simulation failed: ... already in use` | this account was already claimed |
| `Grass returned no payout: the account is not ready to claim yet` | Grass itself refuses: usually every device on the account has to be updated to the version the airdrop requires |
| `The proxy list is empty` | add proxies, they are mandatory |
| Mailbox answers slowly, codes time out | lower **Accounts at a time**, many providers cap simultaneous IMAP sessions |

The full log is in `logs/claimer.log`. Attach it when you ask for help.

## FAQ

**Is the airdrop claimed to my wallet or to the Grass wallet?**
The claim goes to the account's embedded Grass wallet and the USDC is swept to your
collector address in the same transaction. One transaction, nothing left behind.

**Can I run it again after a crash or a stop?**
Yes. Accounts already claimed are skipped by their stored result. An account that
was interrupted mid-way starts over and will need a new email code.

**Does the sponsor wallet need USDC?**
No, only SOL. USDC comes from the airdrop itself.

**Why does Windows SmartScreen or my antivirus complain?**
The executable is packed and obfuscated, which generic heuristics flag routinely.
Download it only from the Releases page of this repository and check the file size
and hash if one is published.

**How many accounts can I run at once?**
Start with 1. The limit is the mailbox provider, not the app: several IMAP sessions
into one shared inbox get throttled or refused, and a lost code costs a whole retry.

**Why does an account need a proxy?**
Grass rate-limits and blocks by IP. Without proxies a list of any size fails on login.

**What if my collector address is the same as the fee address?**
That is handled: the transaction then carries a single transfer instead of two.

---

### About

Grass Season 2 airdrop claimer / claim bot for [Grass.io](https://grass.io) accounts:
bulk claim, multi-account, Solana, USDC, embedded Turnkey wallet export, address
lookup table, pre-send simulation, proxy support, IMAP and mail.tm codes,
2captcha and anti-captcha, English / Russian / Chinese interface.

Keywords: grass airdrop, grass season 2, grass claim, claim grass airdrop,
grass.io claimer, solana airdrop claimer, usdc claim tool, multi account claimer,
airdrop claim bot, grass s2, getgrass.
