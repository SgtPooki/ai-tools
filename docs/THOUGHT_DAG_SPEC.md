# Agent context as a Merkle-CRDT (working name: Continuum / `cot`)

A resumable, forkable chain of thought for agents, persisted on Filecoin Onchain Cloud. One session encodes its prior context; any agent can continue it from a CID on a fresh machine by walking the DAG and folding to state. Shared history is shared by reference, never duplicated.

This is the substrate that earlier point ideas (resume-from-death, agent handoff, verifiable memory) are special cases of.

## Core structure

A thought is an IPLD (`dag-cbor`) node: `{op, parent CID, deps[], payload, meta, sig}`. Because a node's CID is the hash of that struct, it commits to its parents transitively. The DAG is a [Merkle-CRDT](https://hector.link/presentations/merkle-crdts/merkle-crdts.pdf) (Sanjuán et al., the construction behind OrbitDB and Berty):

- The DAG **is** the CRDT causal history.
- Forks are concurrent updates; merge is the CRDT join (associative, commutative, idempotent), so it is automatic and lossless.
- State is the deterministic fold over the DAG. Two agents folding the same head CID get byte-identical state, which is what lets a fold be signed.

## CRDT type

Not a sequence/RGA CRDT. A **G-Set of immutable thought-nodes** (join = set union, CID-keyed dedup) plus a **causal multi-value register per scratch key** (goal, open files). Scratch uses DAG-causal dominance, not LWW: there are no trusted clocks across agents, so concurrent writes both survive and the resuming LLM resolves them in context. Auto-picking a winner would be a correctness lie.

Causal stability is unsolvable under permissionless forking (anyone can fork an old CID forever), so tombstone GC is impossible. Design around it: G-Set has no removes; model `open_files` as a bounded multi-value register holding the whole small list rather than an OR-Set.

The honest limit: the CRDT converges the data, not the reasoning. Two forks can each be valid yet jointly incoherent. Semantic merge stays an agent act (a merge node where the agent writes a reconciliation thought); the CRDT only guarantees no history is lost.

## Cost and walk efficiency

- Batch thoughts into packs with IPLD-internal links; seal CHECKPOINTS to Warm Storage, not every thought.
- Skip-list back-pointers (parent plus 2^k ancestors) give O(log n) resume.
- Compaction via `fold` / snapshot nodes: `{coveredHeads, foldedState, prevSnapshot}`. Old packs stay fetchable for audit; resume cost is O(snapshot + suffix).
- Checkpoint cadence ~50 ops or 5 minutes. Agents never hit FOC per thought.

## Context without token bloat

The running agent keeps a hot window (~20 recent ops + active goal/plan). Older history is fold nodes (deterministic checkpoint summaries). Resume replays folds plus the window, never the full token history. A hard cap refuses resume if an unpack exceeds budget without another fold.

## Authorization: UCAN

The capability travels with the branch; the receiving agent verifies locally with no authorization server ([`@ucanto/*`](https://github.com/storacha/ucanto), UCAN spec 1.0.0-rc.1).

- Resource `with: ipld://<branchCID>`, abilities `thought/read | thought/append | thought/fork`, plus a separate `head/update`.
- Attenuation: owner -> A -> B, each capability equal or narrower than its proof.
- Root authority is the ERC-8004 owner/operator key; agent identities are `did:pkh` verification methods bound in the ERC-8004 registration.
- `thought/append` and `thought/fork` are enforceable at the fold layer: reject a node unless its author's UCAN chain validates and its caveats hold.
- `head/update` carries `toMustDescendFrom: <oldHead>`.

Read is advisory. FOC retrieval is public, so a UCAN cannot deny a fetch; `thought/read` means "authorized to receive the CID and decryption material," and encryption is the real read boundary. Revocation is the weak spot: prefer short TTLs; for high-value branches require head-update validators to consult a revocation set at a well-known CID.

## Confidentiality: skip-ratchet encryption

Anyone can fetch any CID, so treat the DAG as opaque structure over public bytes (WNFS/Peergos-style).

- Each node encrypted with XChaCha20-Poly1305 under a key derived by ratchet `K_n = H(K_{n-1})`. Skip-ratchet key-links mirror the DAG skip-list, so a head-key holder derives any historical key in O(log n). One pointer scheme serves both the walk and key derivation.
- The branch root key is wrapped (ECIES on secp256k1) to authorized ERC-8004 identities and delivered through the UCAN. The head points at a KeyPackage CID with keys wrapped per authorized DID.
- Removing an agent rotates the key forward: forward secrecy only. The removed agent keeps old keys; re-encrypting history is cost-prohibitive. State this plainly.
- Reject global convergent encryption (confirmation attacks: hash a guessed thought, find its CID). Use a per-session salt for intra-session dedup only; the dedup loss is worth the immunity.
- Even encrypted, the DAG leaks metadata: topology, upload-timing pulse, merge-degree, and node size (pad to 256-byte boundaries to blunt size correlation).

## Live handoff (the demo)

```
Pane A (Alice, 0xAlice):
  cot init --erc8004 0xAlice
  cot append --op goal --payload '{"text":"refactor auth middleware"}'   # via post-tool hook
  cot checkpoint                                                          # seals pack, updates head
  cot handoff --to 0xBob                                                  # flushes a sync micro-pack

Pane B (Bob, fresh VM, no prior files):
  cot resume --head erc8004:0xBob   # walk skip-list, fold, inject goal + last ops + fold summary
  # Bob's first turn is a continuation op, not a restart
```

Ops are tagged idempotent vs mutating so a fork-merge does not replay side effects. The illusion breaks on cold-fetch latency (prefetch + cache), daily-not-instant PDP (label "persisted + proven through epoch N"), and unsealed buffers (synchronous micro-pack on handoff; `--tail-local` for same-machine swap).

## What is solid vs aspirational (2026)

Solid: the upload/retrieve/checkpoint/resume loop on FOC, anonymous resume from a CID, signed thought chains (agent-flight-recorder proved this end to end), Merkle-CRDT joins (OrbitDB/go-ds-crdt in production), `@ucanto` delegation, IPLD.

Aspirational: a ratified `fil/cot/v1` node schema (ship a JSON Schema first), a standard ERC-8004 `cotHead` metadata field, frictionless cross-vendor agent hooks, and sub-second handoff. The primitives exist; the convention layer does not yet.

## Minimal v1 (scope cut for a first build)

One CAR, IPLD-internal links, checkpoint seals, local head ref (skip ERC-8004 and encryption), G-Set + scratch register, `init/append/checkpoint/resume/fork`. Defer UCAN, encryption, skip-ratchet, and live cross-machine handoff to v2. This v1 is a believable few-day build on top of agent-flight-recorder.
