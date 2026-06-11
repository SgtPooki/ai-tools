# FOC adoption kit

Growth/DevEx layer, distinct from the 33 product ideas. Goal: get a developer or agent from curiosity to first verifiable CID in under 60 seconds, then give them a reason to show someone else.

## Substrate already exists — do not rebuild it

[`@fil-b/foc-storage-mcp`](https://github.com/FIL-Builders/foc-storage-mcp) (FIL-Builders) is the official FOC MCP server: 10 tools (`uploadFile` with auto-pay, `getDatasets`, `createDataset`, `getBalances`, `processPayment`, `getProviders`, pricing/convert helpers), installable via `npx -y @fil-b/foc-storage-mcp`, Cursor one-click, or `.mcp.json`. Mainnet + calibration.

Adopt it as the integration surface. The adoption kit builds the three things it does NOT do.

## The gaps that are the opportunity

1. **No free onboarding.** It requires the user's own `0x` PRIVATE_KEY, a funded USDFC wallet, and 30 days paid upfront. That wallet wall is where first-try adoption dies. → keystone: a sponsored faucet (#34).
2. **No proof surface.** It uploads but doesn't render PDP verifiability, which is FOC's whole differentiator. → PDP badge + Action (#35). AFR's `verify --remote` already does the read side.
3. **No retrieval.** No download tool; the cheapest aha (anonymous GET, zero wallet) is unexposed. → `foc://` resolver shim (#36).

## Dependency graph

```
            #34 Sponsored faucet  (KEYSTONE: free first CID, no wallet)
                   |
   +---------------+----------------+------------------+
   |               |                |                  |
official MCP   #35 PDP badge   #36 foc:// shim   #37 Demo Passport
(substrate)    (prove side)    (read side)       (event growth loop)
```

Everything depends on #34. With it, the official MCP + npx + badge + passport all become <60s-free. Without it, each hits the wallet wall.

The keystone's own hard part is abuse control: owner-pays-egress makes a public sponsored key a griefing magnet, so rate limits / size caps / monitoring ARE the product (pairs with Egress Tripwire #16).

## Tonight (demo night)

Do not build under time pressure; agent-flight-recorder is the shipped asset. The adoption move is framing + a call to action:

- Reframe AFR's `seal` as the on-ramp: "everything I showed is one `npx` away."
- Close with the loop: a QR on the last slide to a Demo Passport board (#37); invite every project in the room to stamp their demo before leaving. Costs a slide, not a build, and produces a public verifiable artifact of the whole night.
- Optional 2h build only if AFR is untouchably solid: install the official MCP live on stage, seal the talk's own slides to a CID, hand the room the `foc://` link. Self-demonstrating loop, zero new code (the MCP exists).
