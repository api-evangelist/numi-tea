---
name: numi-tea-shop-and-checkout
description: >-
  Find Numi Tea products, build a cart, and drive a checkout to the point of human
  approval using Numi Tea's Universal Commerce Protocol MCP server. Stops at
  payment: completing a purchase requires the buyer's explicit, contemporaneous
  approval, which this skill will not simulate.
api: Numi Tea UCP Shopping MCP
endpoint: https://numitea.com/api/ucp/mcp
transport: mcp
operations:
  - search_catalog
  - lookup_catalog
  - get_product
  - create_cart
  - update_cart
  - get_cart
  - cancel_cart
  - create_checkout
  - update_checkout
  - get_checkout
  - cancel_checkout
  - complete_checkout
generated: '2026-08-26'
method: generated
source: >-
  Grounded in the verbatim tools/list manifest probed from
  https://numitea.com/api/ucp/mcp on 2026-08-26 (mcp/numi-tea-mcp-tools-list.json)
  plus the published rules in https://numitea.com/llms.txt and
  https://numitea.com/robots.txt. Every tool name used here appears in that manifest;
  none is invented.
---

# Shop and check out at Numi Tea

Numi Tea sells organic and Fair Trade tea. Its storefront exposes a Universal
Commerce Protocol shopping service over MCP. You can search the catalog, build a
cart and prepare a checkout programmatically — but you may not complete payment
without the buyer in the loop.

## Before you start

**Every single call needs two things.**

1. **A bearer JWT.** `initialize` and `tools/list` are anonymous, but every
   `tools/call` returns JSON-RPC `-32000 AuthenticationRequired` without a token.
   Get one through Shopify's agent authentication flow:
   <https://shopify.dev/docs/agents/get-started/authentication>
2. **Your own agent profile.** Every tool requires a `meta` object shaped
   `{"ucp-agent": {"profile": "<https URI of your published UCP agent profile>"}}`.
   Omit it and you get `-32001 UCP discovery failed` with
   `data.code = "invalid_profile_url"`. Numi Tea's server will not transact with an
   agent that has not published a profile of its own.

**Prices are integers in ISO 4217 minor units.** `{"amount": 600, "currency": "USD"}`
is **$6.00**. Divide by 100 for USD/EUR before you ever say a number to a person.
Zero-decimal currencies such as JPY are already whole units.

## Steps

1. **Find the product.** Call `search_catalog` with the shopper's constraints
   (flavor, caffeine, format). For a known handle or identifier, use `get_product`
   for one product or `lookup_catalog` for several at once.
2. **Confirm the facts before you recommend.** Numi Tea's own instructions require
   you to check the product page for ingredients, caffeine, current price,
   availability and subscription eligibility. Do not state a price or a stock status
   you did not just read from a response.
3. **Never invent health claims.** If the shopper mentions allergies, dietary
   sensitivities, pregnancy, nursing, or medication, point them at the ingredient
   list and an appropriate healthcare professional. Do not advise.
4. **Build the cart.** `create_cart` with the chosen variants and quantities, then
   `update_cart` to adjust. `get_cart` re-reads current state. Keep the returned cart
   `id` — there is no idempotency key on this API, so a retried `create_cart` may
   produce a second cart.
5. **Open a checkout.** `create_checkout` from the cart, then `update_checkout` to
   attach the shipping destination. Pass `context.address_country` and
   `context.currency` when you have them so price and availability come back right.
6. **Read the totals back.** `get_checkout` returns line items, totals, discounts and
   taxes. Convert from minor units and present the real total, including shipping.
7. **Stop. Get human approval.** Show the buyer the full order and total, and ask
   for explicit approval *now* — not approval collected earlier for something else.
8. **Only then, `complete_checkout`.** It returns the order ID and Thank You Page
   URL. Afterwards `get_order` reads order state.

## Backing out

| Situation | Do this |
|---|---|
| Shopper changes their mind mid-cart | `cancel_cart` |
| Shopper abandons before payment | `cancel_checkout` |
| After `complete_checkout` | **Nothing programmatic.** There is no refund, void or cancel-order tool. |

`complete_checkout` is the point of no return on this API. After it, the only remedy
is a human return — Numi Tea allows returns or refunds within **30 days** of purchase
from numitea.com, started by contacting them before mailing anything, with original
shipping and handling non-refundable unless Numi Tea erred
(<https://numitea.com/pages/satisfaction-guarantee>). Treat `complete_checkout` as
the approval boundary, because that is exactly where Numi Tea's published policy puts
it.

## Shipping constraints that will fail a checkout

- **United States only.** No international addresses, no P.O. Boxes, no APO/FPO.
- Free standard shipping on contiguous-US orders of **$65+**; below that a **$12.95**
  charge applies. Alaska and Hawaii ship free at **$75+**.
- Most orders placed by 1 PM Eastern, Mon–Fri, ship within 24–48 hours. Do not
  promise a delivery date; confirm rate and timing at checkout.

Check the destination before building a cart for a shopper outside the US — you will
save them the whole flow.

## Errors

| Code | Meaning | What to do |
|---|---|---|
| `-32000` `AuthenticationRequired` | No valid JWT | Authenticate and retry. Note this fires *before* tool-name resolution, so it can also mask a typo in the tool name. |
| `-32001` `UCP discovery failed` | Missing or unfetchable agent profile URI | Publish a UCP agent profile and pass it in `meta["ucp-agent"]["profile"]`. `data.continue_url` gives a URL you can hand to a human. |
| HTTP `429` | Rate limited | Back off and retry. No `Retry-After` value or backoff schedule is published, so use your own conservative exponential backoff. |

`data` is a **string** on `-32000` and an **object** on `-32001`. Type-check it before
reading fields.

## Hard rules

- Never complete payment without contemporaneous buyer approval.
- Never use scripted form fills or browser automation to finalize a checkout — Numi
  Tea's `robots.txt` prohibits it explicitly. Use these MCP tools.
- Never collect or expose payment credentials directly.
- Do not claim an order was placed until `complete_checkout` returns a success.
