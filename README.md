# ai-tools

Tracking repo for small agent x [Filecoin Onchain Cloud](https://filecoin.cloud) experiments. Each tool is a POC that should be buildable in a few hours on the Synapse SDK / [filecoin-pin](https://github.com/filecoin-project/filecoin-pin) core API. Ideas live here as issues until they're real; anything that ships gets its own repo and a link below.

## Shipped

- [agent-flight-recorder](https://github.com/SgtPooki/agent-flight-recorder): tamper-evident agent audit trail sealed to Filecoin Warm Storage. Hash-chained Claude Code hook events, signed seals, self-funding Filecoin Pay deposits, daily PDP proofs. Receipts, not self-reports.

## Tracked ideas

Adoption/DevEx strategy (built on the official [`@fil-b/foc-storage-mcp`](https://github.com/FIL-Builders/foc-storage-mcp), not a rebuild): [docs/ADOPTION_KIT.md](docs/ADOPTION_KIT.md). Vertical fit: [docs/VERTICAL_ALIGNMENT.md](docs/VERTICAL_ALIGNMENT.md). Idea validation log: [docs/IDEA_VALIDATION.md](docs/IDEA_VALIDATION.md).

See [open issues](https://github.com/SgtPooki/ai-tools/issues) for the backlog. The theme across all of them: agents need storage, payments, and provenance with no human in the loop, and FOC provides all three as contracts instead of accounts.

Common ground each POC builds on (proven in agent-flight-recorder):

- upload: pack content into a UnixFS CAR, `executeUpload` via filecoin-pin core
- payments: `checkUploadReadiness` + capped USDFC auto-deposit
- verification: content addressing (CID) + wallet-signed manifests
