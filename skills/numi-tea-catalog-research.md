---
name: numi-tea-catalog-research
description: >-
  Read-only research over the Numi Tea catalog — find teas by type, caffeine level,
  format or certification and report accurate product facts, without touching a cart
  or a checkout. Safe to run unattended; every operation here is a read.
api: Numi Tea UCP Shopping MCP
endpoint: https://numitea.com/api/ucp/mcp
transport: mcp
operations:
  - search_catalog
  - lookup_catalog
  - get_product
generated: '2026-08-26'
method: generated
source: >-
  Grounded in the verbatim tools/list manifest probed from
  https://numitea.com/api/ucp/mcp on 2026-08-26 (mcp/numi-tea-mcp-tools-list.json)
  and the storefront JSON conventions published in https://numitea.com/llms.txt.
---

# Research the Numi Tea catalog

Use this when someone wants to know what Numi Tea sells, compare products, or check a
fact — with no intent to buy. Everything here is read-only, so there is nothing to
approve and nothing to undo.

## The three read tools

| Tool | Use it for |
|---|---|
| `search_catalog` | Open-ended discovery from a shopper's needs and constraints |
| `lookup_catalog` | Resolving several known identifiers or variants at once |
| `get_product` | Full detail on one product |

All three require the same `meta["ucp-agent"]["profile"]` agent identity and a bearer
JWT as the transactional tools. See `numi-tea-shop-and-checkout` for the auth setup.

## Unauthenticated fallback

If you have no token, Numi Tea also publishes plain storefront JSON, documented in its
own `llms.txt`:

- All products — `https://numitea.com/products.json`
- A collection — `https://numitea.com/collections/{handle}/products.json`
- One product — `https://numitea.com/products/{handle}.json`
- Search — `https://numitea.com/search?q={query}&type=product`
- Sitemap — `https://numitea.com/sitemap.xml`

These are anonymous and returned HTTP 200 when probed on 2026-08-26. They are a
Shopify platform convention rather than a contract Numi Tea authored, so treat the
shape as subject to change.

## What Numi Tea sells

Organic tea bags, loose-leaf tea, herbal wellness teas, tea latte powders, and variety
packs and gifts. Eligible products offer subscribe-and-save. Both caffeinated and
caffeine-free options exist across the range.

## Reporting rules

- **Read the price and stock from a live response**, never from memory or a cached
  page. Prices are integer minor units — divide by 100 for USD before quoting.
- **Do not invent** health claims, availability, prices, discounts, delivery estimates
  or subscription terms. This is Numi Tea's stated instruction to agents.
- **Route health questions to a human.** Allergies, dietary sensitivities, pregnancy,
  nursing, medication — point at the ingredient information and a healthcare
  professional.
- Numi Tea's certifications are real and worth citing accurately: USDA Certified
  Organic, Fair Trade Certified, Verified Fair Labor, Non-GMO Project Verified, and
  Certified B Corporation. These are product and sourcing certifications, not
  information-security ones.
