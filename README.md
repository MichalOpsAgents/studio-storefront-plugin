# studio-storefront

> Live shopping for **OpsAgents Studio** — drop-in MCP plugin that gives Claude direct access to the studio.opsagents.agency catalog, cart, and policies.

Wraps the public Shopify Storefront MCP endpoint at `https://studio.opsagents.agency/api/mcp`. Zero auth, zero setup, real-time data.

---

## What this plugin gives Claude

| Tool | What it does |
|---|---|
| `search_catalog` | Find products by free-text query, price range, or category. Returns full product objects (id, title, description, variants, price, availability, URL). |
| `get_product_details` | Look up a specific product by ID; pass `options` to select a variant. |
| `get_cart` | Read a cart by ID — items, shipping options, discounts, and the checkout URL. |
| `update_cart` | Add / remove / update line items, buyer info, attributes. |
| `search_shop_policies_and_faqs` | Answer customer questions about shipping, returns, guarantees, and store policy. |

These are the standard UCP catalog tools exposed by every Shopify store. The MCP version is `2026-04-08`.

---

## Why a per-store plugin?

Each Shopify store has its own Storefront MCP at `https://{shop}/api/mcp`. A *per-store* plugin gives Claude a focused mental model:

- **Brand-aware:** Claude knows it's talking about *OpsAgents Studio* products specifically (AI fashion content, not just "products")
- **Composable:** install one plugin per client store; they don't conflict
- **Fast onboarding:** new client = new plugin = 30 seconds of work

The companion skill (`studio-storefront`) teaches Claude when to reach for these tools and how to phrase results in the OpsAgents Studio voice.

---

## Setup

1. Click **Install** on the `.plugin` file in chat
2. That's it. No env vars. No auth. The plugin connects to the live store on first use.

Verify in any session: *"what's in the OpsAgents Studio catalog?"* → Claude should call `search_catalog` and return real products.

---

## SOSA — Level 1 (low impact)

| Pillar | How it's satisfied |
|---|---|
| **Supervised** | Cart updates and policy reads are public-shopper-equivalent operations. There's no admin surface. The user is always in the loop for "actually go check out." |
| **Orchestrated** | SKILL.md enforces: search → inspect → cart → surface checkout URL (never auto-complete checkout). |
| **Secured** | No credentials. Single egress (studio.opsagents.agency only). Endpoint is the same one a customer's browser hits. |
| **Agents** | R=studio-storefront, T=storefront-mcp (5 tools), M=none (stateless), P=search → inspect → cart → checkout-url |

---

## Architecture note

The plugin is a single `.mcp.json` pointing at the live Shopify endpoint. There is no server to build, no Cloud Run service, no tokens. If Shopify adds tools to the Storefront MCP (e.g. `get_order_status`), they'll show up automatically the next time Claude lists tools.

To swap to another store: copy this plugin folder, change the URL in `.mcp.json`, change the names in `plugin.json` and `skills/`, ship.

---

## Layout

```
studio-storefront/
├── .claude-plugin/plugin.json     # SOSA L1 manifest
├── .mcp.json                      # HTTP transport → studio.opsagents.agency/api/mcp
├── README.md
└── skills/studio-storefront/
    └── SKILL.md                   # When/how to use the tools, brand voice
```

## License

MIT.
