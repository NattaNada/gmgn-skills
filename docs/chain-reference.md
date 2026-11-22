# Chain Reference — Canonical Chain Lists for gmgn-skills

**Single source of truth for chains in this repo.** Every skill and doc that names
chains must use the lists below, verbatim. Do not write a different chain count
("9 chain", "8 EVM", "7 chain") when you mean the CLI's supported set — and when a
skill deliberately covers only a subset (e.g. `gmgn-heat-rank` sweeps seven chains
by design), say so explicitly as a subset of this list, never as the CLI's full set.

Source of truth in code: `src/validate.ts` (`VALID_CHAINS`, `READ_CHAINS`,
`CALLOUT_CHAINS`), verified against `gmgn-cli` 1.7.2
(`gmgn-cli <command> --help` prints the same lists).

## 1. Read chains (11) — `READ_CHAINS`

Every read-only command (`token`, `market`, `portfolio`, `track`) accepts these
11 chains. 10 of them are EVM; `sol` is the only Solana chain.

| Chain | Type | Trading supported? | Notes |
|-------|------|--------------------|-------|
| `sol` | Solana | Yes | Base58 addresses (`^[1-9A-HJ-NP-Za-km-z]{32,44}$`) |
| `bsc` | EVM | Yes | `0x` + 40 hex |
| `base` | EVM | Yes | `0x` + 40 hex |
| `eth` | EVM | Yes | `0x` + 40 hex |
| `arbitrum` | EVM | Yes | `0x` + 40 hex |
| `hyperevm` | EVM | Yes | `0x` + 40 hex |
| `robinhood` | EVM | Yes | `0x` + 40 hex |
| `arc` | EVM | Yes | `0x` + 40 hex |
| `stable` | EVM | Yes | `0x` + 40 hex |
| `avalanche` | EVM | Yes | `0x` + 40 hex |
| `elysium` | EVM | **No — read-only** | Kinetiq's HyperEVM L2. No trading yet: `swap` / `order` / `strategy` / `cooking` reject it (`validateChain` in `src/validate.ts`). Read commands accept it, and `gas-price` accepts it too. |

Address validation (`src/validate.ts`, `validateAddress`): every chain except `sol`
is EVM and takes a `0x` + 40-hex address; `sol` takes a 32–44 character base58
address. Never list only some of the EVM chains when the rule covers all ten.

## 2. Trade chains (10) — `VALID_CHAINS`

All read chains **except `elysium`**:

`sol` / `bsc` / `base` / `eth` / `arbitrum` / `hyperevm` / `robinhood` / `arc` /
`stable` / `avalanche`

These are the chains `swap` / `order` / `strategy` / `cooking` accept. Passing
`elysium` to one of them fails with "read-only: trading is not supported".

## 3. Chains that are NOT in these lists

- **`monad`** was removed from `VALID_CHAINS` — it survives only as a commented-out
  entry (`/*, "monad" */`) in `src/validate.ts`. It is in **neither** `VALID_CHAINS`
  nor `READ_CHAINS`: no CLI command accepts it today. Do not list it as a
  supported chain; if it returns, it returns in `src/validate.ts` first.
- **`megaeth` / `tron` / `xlayer`** are **not** read or trade chains. They appear
  only in the separate call-out chain list below.

## 4. Call-out chains (15) — `CALLOUT_CHAINS` (separate list, do not conflate)

Call-outs are read-only data, so the service accepts a wider chain list than the
trade/read chains (`src/validate.ts`, `CALLOUT_CHAINS`; mirrors `calloutChains`
in gmgn-openapi-service):

`sol` / `bsc` / `eth` / `base` / `megaeth` / `hyperevm` / `tron` / `monad` /
`xlayer` / `robinhood` / `arbitrum` / `arc` / `stable` / `avalanche` / `elysium`

That is the 11 read chains **plus** `megaeth`, `tron`, `monad`, `xlayer`. A token
surfaced by `callout global` on e.g. `tron` or `monad` may not be queryable with
`gmgn-token` or tradable with `gmgn-swap`. Only `gmgn-callout` and
`gmgn-callout-score` use this list; every other skill uses §1/§2.

## 5. Rules for every skill and doc

1. **Quote §1 when you mean "the chains the CLI reads", §2 when you mean "the
   chains the CLI trades on", §4 when you mean call-outs.** Never merge the
   lists, and never copy a chain list from memory — check `src/validate.ts`
   or `gmgn-cli <command> --help`.
2. **Never write another chain count for the CLI's supported set.** No "9 chain",
   no "8 EVM", no "7 chain" as the CLI total. The numbers are 11 read, 10 EVM,
   10 trade, 15 call-out. A skill that covers a subset (e.g. `gmgn-heat-rank`'s
   seven-chain sweep) must label it as its own subset.
3. **When the CLI changes, this file changes first.** A new chain is added to
   `src/validate.ts`, then to this file, then to the skills that use it — in
   that order, in the same change.
4. **Per-command exceptions live in the command's own doc**, not here: e.g.
   `market signal` does not support `hyperevm`; `market hot-searches` defaults
   to the 11-chain set. This file states the sets; the command docs state the
   exceptions.
