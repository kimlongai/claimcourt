# ClaimCourt — an adjudication court for airdrop & token-claim announcements

Built on **GenLayer** (Intelligent Contracts, Optimistic Democracy).

## The problem

Airdrop farmers are spammed daily with fake claim announcements — "$GEN claims
are live!", "$XYZ airdrop is open, connect your wallet!" — most of them
phishing pages. There is no neutral, on-chain way to ask: *does this page
actually say what the post claims it says, and does it show scam red flags?*

## What ClaimCourt does

Anyone can open a case by submitting:

- `url` — the announcement / claim page
- `claim` — what the page is said to announce

The Intelligent Contract fetches the page itself, and GenLayer's AI validators
independently read it and judge the claim. The verdict is stored on-chain:

- **VERIFIED** — the page really makes this claim and shows no serious red flags
- **NOT_FOUND** — the page loads but does not make (or contradicts) the claim
- **SUSPICIOUS** — the page shows scam red flags (seed-phrase requests, "send
  crypto to verify/unlock", impersonation, guaranteed-profit promises, fake
  urgency countdowns)
- **UNREACHABLE** — the page could not be read

Equivalence principle: validators re-run the evaluation independently and must
agree on the **verdict** (the decision). The wording of summaries may differ —
consensus is on the judgment, not the bytes.

The page content is treated as untrusted data inside the prompt; the contract
never follows instructions found on the page being judged.

## Contract

`contracts/claim_court.py`

- `submit_case(url, claim) -> case_id` (write)
- `get_case(case_id) -> dict` (view)
- `get_case_count() -> int` (view)
- `get_all_case_ids() -> list` (view)

Runner pin matches the official GenLayer project boilerplate
(`py-genlayer:1jb45aa8…`), the same pin used for Studio and Bradbury.

## Live deployment (GenLayer Studio network, full consensus)

- **Contract:** `0x5F06335a32a921e98482f7217b4d0B4B0d7A98E2`
- **Deploy tx:** `0x978cd483646bc3dfe06683b04a59c1d677cc572ea9b0fc2817c64c187ee8d88` (ACCEPTED)

Cases judged so far:

| Case | Page | Claim | Verdict |
|------|------|-------|---------|
| case-1 | testnet-faucet.genlayer.foundation | "lets visitors claim 100 GEN testnet tokens, once per week" | **VERIFIED** — jury summary even noted the 0.01 ETH mainnet requirement (tx `0x04494f6b…fe009053`) |
| case-2 | genlayer.com | "GEN token claims are live now and anyone can claim GEN on this page" | **NOT_FOUND** — the homepage describes the adjudication protocol; no claim/airdrop function exists there (tx `0x3f991d07…e911f8`) |

## Deploy & try it (GenLayer Studio, Studionet — free, no faucet needed)

1. Open https://studio.genlayer.com and connect / sign in.
2. Make sure the network selector is on **Studionet** (Studionet transactions
   are free; if Studio shows a balance, use the 💧 faucet button in the
   account selector).
3. Create a new contract file named `claim_court.py` and paste the code from
   `contracts/claim_court.py`.
4. Open **Run and Debug** and deploy the contract. Save the contract address
   and the deploy transaction hash.
5. Call `submit_case` with a test case (below), wait for consensus, then read
   the verdict with `get_case("case-1")`.

### Test cases

**Case A — expect VERIFIED**

- url: `https://testnet-faucet.genlayer.foundation/`
- claim: `This page lets visitors claim 100 GEN testnet tokens, once per week`

**Case B — expect NOT_FOUND**

- url: `https://genlayer.com/`
- claim: `GEN token claims are live now and anyone can claim GEN on this page`

## Roadmap

- Simple web frontend that calls the contract (submit a case, browse verdicts)
- Deploy to Testnet Bradbury once testnet GEN is available
- Public "wall of verdicts" page for the farmer community

## Submission notes (GenLayer Portal — Projects)

- Real Intelligent Contract with non-deterministic judgment that adds value
  (web evidence + LLM jury), not a deterministic re-check
- Meaningful on-chain state: every case and verdict is stored and queryable
- Clear repro: this README + the two test cases above
