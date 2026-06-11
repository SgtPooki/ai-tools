# Vertical alignment: where AI-agent x FOC actually fits

Most FOC-vs-cloud value comes from four differentiators a normal bucket can't match:

- **Verifiable persistence** (PDP): prove data still exists, unaltered, daily, onchain.
- **Content addressing**: immutable provenance and permanent references.
- **Censorship resistance**: multi-provider, jurisdiction-independent, no single takedown point.
- **Machine-native payments + identity** (Pay, ERC-8004): agents transact and are addressable without cards or accounts.

A vertical is a good fit when its data carries legal/scientific/financial weight that needs "prove it existed and wasn't changed", AND real agent work is happening there, AND public storage isn't a dealbreaker (or encryption is designed in).

## Scoring

| Vertical | Differentiator that matters | Agent work | Privacy barrier | Verdict |
|---|---|---|---|---|
| Scientific research / reproducibility | content-addressed dataset hashes, permanence | high | low (open data) | strong |
| Journalism / human rights / media provenance | censorship resistance (RIMA live on Filecoin) | high | medium | strong |
| Legal / eDiscovery | chain of custody, contemporaneous immutable record | high | high | strong |
| RegTech / AI audit | immutable audit trail; EU AI Act high-risk live Aug 2 2026 | very high | medium | strong (partly covered by AFR + compliance kit) |
| Healthcare / clinical trials / pharma | data integrity (FDA 21 CFR Part 11) | high | very high | medium; encryption-first only |
| Supply chain / ESG / carbon | provenance of claims + sensor data | medium | low | medium |
| Creator economy / IP / gaming | provenance, asset permanence | medium | low | medium |
| Government / civic / public records | permanence + transparency + censorship resistance | medium | medium | medium |
| Identity / KYC / personal finance | (privacy-dominated) | low | very high | weak |

## Grounding signals (2026)

- EU AI Act high-risk obligations activate 2026-08-02; the required record must be contemporaneous, complete, immutable; a majority of orgs rely on fragmented logs that can't produce a coherent chain of custody.
- The Russian Independent Media Archive preserves 6M+ documents from independent journalists on Filecoin specifically for censorship resistance.
- Open-source model teams publish on-chain dataset hashes so reproducibility is checkable years later.
- Decentralized storage market ~$623M (2024), enterprise ~45% share; growing.

## Note on privacy

Public storage rules out plaintext PHI/PII. For healthcare and any regulated personal data, the design must be encryption-first (commit-and-selectively-reveal, age-encrypted shards) per agent-flight-recorder's PRIVACY_AND_ENCRYPTION.md. Verticals scored "very high" privacy barrier are viable only with that.

## Vertical ideation results

Ran one ideation pass with each model assigned a distinct vertical cluster. Filed the distinct winners; recorded the folds below.

Filed (issues #27-#33):
- #27 Frozen Analysis Window (science: preregistration integrity)
- #28 Takedown-Resilient Mirror Desk (journalism: censorship-resistant archival agent)
- #29 Source Risk Splitter (human rights: redaction + provenance)
- #30 GxP Shadow-Chain Sentinel (pharma: lab data-integrity fraud)
- #31 Privilege Log Custody (legal/eDiscovery)
- #32 Recall Provenance Spine (supply chain/ESG: decision-time attestation pinning)
- #33 FOIA Deadline Streams (civic: retention-funded public records)

Folded, not filed (duplicates of taken work):
- Agent Claim Anchors (science): overlaps citation notary (ai-tools#1).
- Repro Evidence Pack (journalism): overlaps flight recorder + science cluster.
- Provenance Quarantine Agent (deepfake C2PA): valuable but a provenance-stamp shape; revisit if a media partner wants it.
- Model Change Ledger (RegTech): the proposer flagged it as flight-recorder overlap; the only new angle is pre-deployment config gating.
- Blinded Cohort Arbiter, Biobank Telemetry Witness, Pharma IP Ledger (healthcare): each leans on a hard external dependency (pay-stream finance shape, sensor TEE, or prior-art notary). Parked pending a real healthcare partner, since the privacy/TEE constraints dominate.
