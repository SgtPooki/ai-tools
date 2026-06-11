# Idea validation against FOC source (2026-06-11)

20 ideas from a 4-agent brainstorm (cursor, codex, gemini, claude), validated against actual contract and SDK source in filecoin-pay, filecoin-services (FWSS), pdp-explorer, curio, synapse-sdk, and filecoin-pin. Survivors became issues; the rest are recorded here with the reason, so they don't get re-proposed without new primitives.

## Validated primitive facts (with sources)

These are the load-bearing facts every verdict below rests on:

1. **Pay rails support conditional settlement via a validator.** A rail can name a `validator` contract; at settlement, `IValidator.validatePayment()` can reduce the amount or refuse to settle past an epoch (filecoin-pay `src/FilecoinPayV1.sol`, validator flow around the settle path). FWSS itself is such a validator: it reduces provider payment for faulted epochs. Escrow-with-condition is therefore real, but each new condition means writing a validator contract.
2. **Anyone can call `settleRail`** (public, no access control). One-time payments exist via `modifyRailPayment`'s `oneTimePayment`, drawn from fixed lockup.
3. **No hard lockup-duration cap** in the contract; the payer-set per-operator `maxLockupPeriod` governs. SDK default is 30 days.
4. **PDP challenges are protocol-scheduled only.** The proof clock advances via `nextProvingPeriod()` with the challenge epoch fixed by `IPDPProvingSchedule`; there is no function for an outsider to demand an extra challenge (curio `PDPCURIOSPEC.md`, PDPVerifier ABI).
5. **Proof history is fully reconstructable from events**: `PossessionProven`, `ProofFeePaid`, `NextProvingPeriod`, and `FaultRecord` (missed proofs). The pdp-explorer subgraph already indexes these. A script can build "dataset X proven daily from A to B" from chain data alone.
6. **Faults reduce payment automatically** (FWSS as validator). No slashing, no third-party claim mechanism.
7. **Retrieval is anonymous, unauthenticated HTTP GET** from provider endpoints or FilBeam (`https://{owner-wallet}.{filbeam-domain}/{pieceCid}`). With `withCDN`, egress is drawn down from the OWNER's pre-locked funds as retrievals happen. **There are no retrieval receipts and no per-retriever billing.** Anyone with the CID can fetch, and it costs the owner.

## Rejected ideas, with reasons

| Idea (source) | Why rejected |
|---|---|
| Latency Bond (cursor) | Needs per-retrieval SLA settlement and a latency oracle; no retrieval receipts exist (fact 7), and "how slow is slow" has no oracle. |
| Dream Buffer (cursor) | Mechanically just "pay to store a peer's CID"; its own pitch admitted it's ceremonial without a discovery layer. |
| Swarm Split Bill (cursor) | A dataset has one payer; multi-payer cost split isn't native to rails. Off-chain invoice is genuinely simpler. |
| Negotiation Tape (cursor) | The tape is agent-flight-recorder; the escrowed finality needs a custom validator. The novel part is the part that doesn't exist. |
| Orphan Tool Graveyard (cursor) | "Pay-to-retrieve" is inverted: the owner pays egress, the reader pays nothing (fact 7). Plain pinning of old schemas is trivial and low-value. |
| Agent Data Lease (codex) | Retrieval rights can't be enforced onchain (fact 7: anonymous fetch). The encrypted variant collapses into key management, already covered by agent-flight-recorder's PRIVACY_AND_ENCRYPTION doc. |
| Proof-Gated Benchmark (codex) | Weaker duplicate of Sealed Benchmark Vault (issue #6). |
| L2 Context Pager (gemini) | Mechanically fine, conceptually a cache with extra steps; the "deterministic sub-second retrieval" claim isn't supported by anything in source. |
| Liar's Bounty Auditor (gemini) | On-demand PDP challenges do not exist (fact 4), and payment already auto-reduces on faults (fact 6). The observable part is pdp-explorer, which exists. |
| Verified-Fetch Honey-Pot (gemini) | Requires wallet-attributed retrieval receipts, which do not exist (fact 7). |
| Storage Darwinism (claude) | Needs per-dataset retrieval revenue as a demand signal; retrieval earns the owner nothing and costs them egress (fact 7). Economics inverted as pitched. |

## Survivors

Tracked as issues #6-#12: Sealed Benchmark Vault, agent dead-man's switch (design-first), paid context packs (merged from Metered RAG Faucet + Model Context Vending, with the payment model corrected), Compute Witness Cache, Forecast League, Robot Receipt Inbox, Hibernation Vault.

## Round 2 (2026-06-11, same day): primitives-first ideation

Round 2 gave the four agents the verified facts up front and asked them to exploit the corners. Convergence was strong: all four proposed settlement keepers, fault-conditioned finance, and egress-griefing protection independently.

### Additional verified facts

8. **Proof state is contract-readable, not just event-emitted.** `FilecoinWarmStorageServiceStateView.sol` exposes `provenPeriods(dataSetId, periodId)` and `provenThisPeriod(dataSetId)` as public views. Deployed on mainnet (`0xB1B3A3d979c1f233c1021EF98dff9c0932FF1bb9`) and calibration (`0x537320bd004a7FDd3c1932ca64BD88268301322A`); ABI ships in synapse-sdk (`abis/generated.ts`). A custom Pay validator can therefore settle against PDP proof history with no oracle.
9. **Settlement timing carries no alpha.** FWSS `validatePayment` (FilecoinWarmStorageService.sol:1475) computes payment per epoch from `_isPeriodProven`; faulted periods settle at zero whenever settlement runs, and open periods block settlement until resolved. Settle now or settle later, the amounts are identical.
10. FWSS `validatePayment` + `_findProvenEpochs` is a working in-tree template for any "pay only for proven epochs" validator.

### Round 2 rejected

| Idea (sources) | Why rejected |
|---|---|
| Settlement Sniper / Crank (all four) | Fact 9: no timing alpha. What remains is settlement-as-a-convenience, and payees can self-settle. |
| ERC-8004 Egress Gatekeeper (gemini) | The FilBeam URL is derivable from public info (owner address + CID); a gatekeeper that hands out public URLs gates nothing after the first leak. |
| Fault-responsive self-healing re-pin (gemini, cursor) | Mechanically viable, but Warm Storage's own roadmap lists automated repair; check with the team before duplicating protocol-level work. |
| Persistence-gated reveal / Epoch Secret Vault / Time Capsule (cursor, codex, gemini) | Real (fact 8 makes the validator version work), but it's a generalization of Sealed Benchmark Vault (#6); folded there as a note rather than a separate issue. |

### Round 2 survivors

Issues #13-#16: Provider Credit Score Oracle, Fault Futures Desk, Proof-Paced Vesting, Egress Tripwire. The first three compose: PDP proof history as a zero-oracle financial settlement layer.
