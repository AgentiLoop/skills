---
name: x402-web-tools
description: Pay-per-call web tools for agents with an x402 wallet (USDC on Base or Solana, no account or API key) — fetch any public page as clean Markdown, read a page's metadata/OpenGraph/JSON-LD, host a file at a stable public URL for 30 days, or render a QR code. Use when an agent needs page extraction, link metadata, a shareable file URL, or a QR image and can pay a few tenths of a cent with x402; also usable as a remote MCP server.
license: MIT
metadata:
  author: AgentiLoop
  version: "1.0"
  homepage: https://qrcode.pub/qr-code-api?ref=skills#x402
---

# x402 web tools (qrcode.pub)

Four HTTP endpoints priced in USDC, paid with the open [x402](https://x402.org) protocol (version 2): the server answers `402 Payment Required` with a `PAYMENT-REQUIRED` header, the client signs a USDC transfer authorization, retries with `PAYMENT-SIGNATURE`, and the facilitator settles on-chain. No account, no API key, no gas for the caller.

| Endpoint | Price | What you get |
|---|---|---|
| `GET /x402/extract?url=` | $0.010 | `{url, status, title, description, markdown, words, links[], truncated}` — scripts, styles and nav removed; deterministic, no model call |
| `GET /x402/meta?url=` | $0.005 | `{title, description, canonical, lang, og{}, twitter{}, icons[], jsonld[]}` |
| `POST /x402/store?name=` | $0.010 | upload ≤ 5 MB (png, jpeg, webp, gif, pdf, json, txt, md, csv) → `{id, url: https://qrcode.pub/f/<id>, expires, token}`; public for 30 days; `DELETE /x402/store/<id>` with `Authorization: Bearer <token>` removes it |
| `GET /x402/qr?data=` | $0.005 | PNG or SVG QR code; same parameters as the free `/api/qr` (`size`, `format`, `ecc`, `margin`, `color`, `bgcolor`) |

Base URL `https://qrcode.pub`. Nothing is charged for 4xx answers (bad input, unreachable page, oversized or disallowed file). Free catalog with full payment requirements and Bazaar schemas: `GET https://qrcode.pub/.well-known/x402` (also `/x402`), OpenAPI: `GET https://qrcode.pub/openapi.json`.

## Paying

Accepted: USDC on Base (`eip155:8453`, EIP-3009 `transferWithAuthorization`) or USDC on Solana mainnet. Any x402 v2 client works:

```ts
// npm i @x402/fetch @x402/evm viem   (official x402 v2 client; pays from a funded wallet key)
import { wrapFetchWithPaymentFromConfig } from "@x402/fetch";
import { ExactEvmScheme } from "@x402/evm";
import { privateKeyToAccount } from "viem/accounts";

const account = privateKeyToAccount(process.env.WALLET_KEY as `0x${string}`);
const fetchWithPay = wrapFetchWithPaymentFromConfig(fetch, {
  schemes: [{ network: "eip155:8453", client: new ExactEvmScheme(account) }],
});
const res = await fetchWithPay("https://qrcode.pub/x402/extract?url=" + encodeURIComponent(page));
const { markdown, links } = await res.json();
```

Manual flow: call without a payment header → read the base64 JSON in the `PAYMENT-REQUIRED` header (or body), pick one `accepts[]` entry, sign the payment payload for exactly `amount` to `payTo`, retry with `PAYMENT-SIGNATURE: <base64 payload>`. The settled transaction hash comes back in `PAYMENT-RESPONSE`.

Uploading: send the raw bytes with a matching `Content-Type`, or `multipart/form-data` with field `file`. Optional `?name=report.pdf` sets the download file name.

## As an MCP server

`POST https://qrcode.pub/mcp` is a stateless Streamable HTTP MCP server (no auth) listed in the official MCP registry as `pub.qrcode/tools`. Tools: `qr_code` (free), `page_to_markdown`, `page_metadata`, `host_file`. Paid tools called without a `payment` argument return the price and payment requirements as a structured tool error; pass the base64 `PAYMENT-SIGNATURE` value as `payment` to run and settle.

```json
{ "mcpServers": { "qrcode-pub": { "type": "http", "url": "https://qrcode.pub/mcp" } } }
```

## When to use which

- Need a QR code and have no wallet → use the free URL API (`qr-code-generator` skill in this repository).
- Need a page as Markdown for summarisation or RAG → `/x402/extract`.
- Need only title/description/preview image for a link → `/x402/meta` (cheaper).
- Need to hand a human or another agent a file by URL (chart, PDF, CSV, JSON) → `/x402/store`, then optionally a QR code of the returned URL.

Docs: https://qrcode.pub/qr-code-api?ref=skills#x402
