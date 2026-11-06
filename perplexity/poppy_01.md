# What Poppy is

Poppy (the Personal Agent Protocol) is an open standard announced on 6 October 2026 by Sierra (Bret Taylor and Clay Bavor) and Meta, for how a customer's personal AI agent works with a business on that customer's behalf. The first draft shipped 9 October under Apache 2.0; v0.2 was frozen 24 October and v0.4 is in progress. Backing is heavy: 40+ design partners including OpenAI, Visa, Mastercard, PayPal, Bank of America, Shopify, Stripe and Walmart.

Think of it as a front door for agents. Today, when you ask your agent to "return this jacket," it has to impersonate you — log in as you, click through pages built for humans. The business can't tell an agent from a customer, so its only options are blocking agents or granting full account access. Poppy replaces that all-or-nothing model with graduated, OAuth-based access: the customer picks what the agent may do, the business sets the boundaries.

It has five building blocks:
- Discovery — the business publishes /.well-known/poppy.json, describing sign-in options, scopes, APIs, its website and its agent endpoints.
- Sessions — the agent identifies itself with a signed JWT on every request; you get a pseudonymous ID per company, so businesses can't join your profiles across brands.
- Sign-in — standard OAuth (with PKCE or device flow). Scopes are company-defined (e.g. orders:read, orders:write) and can carry hints like reversible vs irreversible.
- One session across every channel — the same session spans the website, APIs and conversations, so a question asked before sign-in and an action after it count as one visit.
- Getting it done — via the company's website, its APIs (OpenAPI or MCP), or its own agent, with human handoff when needed.

## How it relates to MCP
MCP (Anthropic, Nov 2024, now under the Linux Foundation's Agentic AI Foundation) is the tool-access layer: how an AI app calls tools and data. Poppy doesn't replace it — it wraps it. A company lists its remote MCP servers as one of Poppy's two API types, and Poppy supplies what MCP leaves out for this use case: agent identity, a pseudonymous user, sign-in to a specific account, and one session across channels. MCP answers "how does the model invoke this tool"; Poppy answers "should this agent touch my account at all, and for what."

## How it relates to A2A
A2A (Google, Apr 2025, also Linux Foundation-hosted) is agent-to-agent communication — discovery via Agent Cards, task delegation, status streaming. Mostly complementary, with one overlap to watch. Poppy doesn't reference A2A at all; its company-agent conversations use Poppy's own JSON-over-HTTPS protocol. But "talk to the company's own agent" is one of Poppy's three routes, and that is agent-to-agent traffic — so expect the two to bump into each other there. The key difference: A2A is about how two agents exchange work; customer consent is out of its scope, while for Poppy it's the core.

## The one-liner for leadership
They stack, they don't compete: Poppy = who is calling and what may it do (consent layer). A2A = how agents talk to each other. MCP = how an agent calls tools. Poppy lists MCP as an API route and leaves A2A-style conversations to its own protocol.

## Caveats and what I'd do
Be honest about maturity: it's a fast-moving draft.
One important update from the official documentation: the current published target is now Poppy draft 0.4, not 0.3 as in the earlier briefing. Draft 0.4 makes a particularly important security distinction: every Poppy resource request carries a signed Poppy-Session header, while account access is separately represented by an OAuth access token. I’ll base the demo on the version-pinned 0.4 spec and call out that it is still a draft.
0.2 froze fifteen days after 0.1 with mostly breaking changes — with no production deployments yet; only a handful of companies even serve a valid poppy.json so far. Sierra promises workshops and a reference implementation. My advice: track, don't rebuild. MCP-enabling your APIs is useful regardless of Poppy's fate; a poppy.json can follow once the spec stabilises.
