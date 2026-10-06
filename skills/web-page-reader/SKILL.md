---
name: web-page-reader
description: Read any public web page as clean Markdown, get its metadata (OpenGraph, Twitter card, JSON-LD, icons), or read a page plus up to 4 linked pages on the same site in one call — paid per call in USDC (x402 or MPP), no account or API key. Use when an agent needs the text of a URL, a docs section, or link-preview metadata and has an x402 or mppx wallet.
---

# Web page reader (PageWire)

PageWire turns public web pages into agent-ready Markdown. Every endpoint is a plain HTTP GET that answers
`402 Payment Required` with the exact price; pay with any x402 v2 client or `mppx` and retry. Bad input and
pages that cannot be fetched (4xx) are never charged.

| Need | Endpoint | Price |
|---|---|---|
| Text of one page | `GET https://pagewire.dev/x402/extract?url=<page>` | $0.01 |
| Title, description, OpenGraph, Twitter, canonical, icons, JSON-LD | `GET https://pagewire.dev/x402/meta?url=<page>` | $0.005 |
| A page plus up to 4 same-site pages it links to | `GET https://pagewire.dev/x402/crawl?url=<page>&prefix=/docs/&limit=4` | $0.03 |

## Paying

- **mppx (MPP, USDC.e on Tempo or USDC on Base):** `npx mppx "https://pagewire.dev/x402/extract?url=https://example.com"`
- **x402 (USDC on Base):** wrap fetch with `@x402/fetch` and call the same URL; the client reads the
  `PAYMENT-REQUIRED` header, signs, and retries with `PAYMENT-SIGNATURE`.

## MCP

Remote Streamable-HTTP MCP server: `https://pagewire.dev/mcp` (tools `page_to_markdown`, `page_metadata`, `crawl_site`).

```sh
claude mcp add --transport http pagewire https://pagewire.dev/mcp
```

A paid tool called without its `payment` argument returns the x402 payment requirements; sign one `accepts`
entry and call again with the base64 payload as `payment`.

## Output (extract)

`{ url, status, title, description, markdown, words, links: [{ text, href }], truncated }` — pages over 2 MB are truncated.

## Notes

- Only public http(s) URLs; private/local addresses are refused.
- No JavaScript rendering: single-page apps that render client-side may return little text.
- Discovery: https://pagewire.dev/openapi.json · https://pagewire.dev/llms.txt
