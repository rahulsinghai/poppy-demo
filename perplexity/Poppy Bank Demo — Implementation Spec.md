# Poppy Bank Demo — Implementation Specification

**Project:** Meridian Bank — a live demonstration of the Personal Agent Protocol (Poppy)
**Spec version:** 1.0 (2026-11-06)
**Audience:** a coding agent with full repo access. Build exactly what is specified; where this document says `VERIFY`, confirm against the ground-truth sources in §2 before coding that piece.

---

## 1. Goal and demo script

Build a working, deployable demo of Poppy (the Personal Agent Protocol) using a fictional UK retail bank, **Meridian Bank**, so a live audience can watch a personal AI agent discover a bank, ask a guest question, sign the customer in with scoped OAuth consent, read account data, and execute an approved irreversible payment.

### The 5-minute live script this system must support

| # | Actor | Action | Poppy building block demonstrated |
|---|-------|--------|------------------------------------|
| 1 | Presenter | Opens `https://agent.<DOMAIN>` (chat UI). Types: "Can you check my bank?" | — |
| 2 | Agent | Fetches `https://bank.<DOMAIN>/.well-known/poppy.json`, logs it in the activity panel | Discovery (§3 of the Poppy spec) |
| 3 | Agent | Opens a guest (signed-out) session, asks the company agent: "Do you offer fee-free current accounts?" — gets an answer | Sessions + conversations without identity |
| 4 | User | Types: "What's my balance?" | — |
| 5 | Agent | Detects sign-in is required; UI shows the bank's own authorization page link; user signs in and grants **only** `accounts:read` | Sign-in, scopes, company-hosted consent |
| 6 | Agent | Calls the accounts API, shows balance and recent transactions | Consented API access (OpenAPI route) |
| 7 | Agent | Notices rent due to Rivercrest Lettings; proposes paying £1,850 on the 1st | Agent reasoning |
| 8 | User | Approves. Bank's approval page opens showing payee + exact amount; agent requests `payments:create` (scope marked `irreversible`); user approves | Scoped escalation + Operations approval flow |
| 9 | Agent | Executes the payment via the bank API; balance and transaction list update | Act-with-approval tier |
| 10 | Presenter | Shows the contrast: the same request with the scope revoked / against a "blocked" mock bank returns a clean refusal | The access spectrum vs all-or-nothing |

### Non-goals

- Real payments, real money, real banking licences, regulatory compliance. All data is fictional and seeded.
- Implementing Poppy from scratch. Use the community SDKs (§2). Do not hand-roll JWT/DPoP crypto.
- Mobile UI, i18n, load testing, multi-tenancy.

---

## 2. Ground truth and dependencies

The Poppy draft is moving fast (0.1 → 0.2 → 0.3 within three weeks of launch, mostly breaking changes). **The specification and the SDKs are the source of truth, not this document.** Wherever this spec sketches a wire format, `VERIFY` it against these:

| What | Where |
|------|-------|
| Poppy draft 0.3 spec | https://personalagentprotocol.org/docs/0.3/spec (source: https://github.com/personal-agent-protocol/personal-agent-protocol, `content/versions/0.3/spec.mdx`) |
| Introduction / how it works | https://personalagentprotocol.org/docs/introduction |
| Python + TypeScript SDKs (implement draft 0.3 + Operations extension v2) | https://github.com/datalayer/personal-agent-protocol — PyPI package `personal-agent-protocol` |
| Community demos & validator | https://personalagentprotocol.org/docs/demos and https://github.com/adewale/pda (validator for poppy.json + OAuth metadata) |
| Curated ecosystem list | https://github.com/JaviChulvi/awesome-poppy |
| Kaggle notebook limits | https://www.kaggle.com/docs/efficient-gpu-usage (30h/week GPU quota, 12h max session, idle timeouts) |

**Pinned dependency policy:** pin exact versions in `requirements.txt` / `package.json`; record the Poppy draft version each build was tested against in `docs/BUILD_LOG.md`.

---

## 3. System architecture

Three always-reachable or demo-time components plus one ephemeral one:

```
┌──────────────────────────────────── Oracle VM (Always Free, 4 Arm OCPU / 24 GB) ───────────────────────────────────┐
│                                                                                                                     │
│  ┌─────────────────────┐        ┌──────────────────────────┐         ┌──────────────────────────────┐               │
│  │  bank.<DOMAIN>      │        │  agent.<DOMAIN>           │         │  Postgres (Docker)           │               │
│  │  Meridian Bank      │        │  Personal Agent Runtime   │         │  users, accounts, txns,      │               │
│  │  - bank website     │        │  - chat UI (demo console) │         │  payees, payments,           │               │
│  │  - /.well-known/    │        │  - Poppy client (SDK)     │         │  poppy sessions/tokens/ops   │               │
│  │    poppy.json       │        │  - LLM provider adapter   │         │  + agent job queue           │               │
│  │  - OAuth issuer +   │        │  - activity log (SSE)     │         │                              │               │
│  │    consent pages    │        └────────────┬─────────────┘         └──────────────────────────────┘               │
│  │  - OpenAPI APIs     │                     │ REST (OpenAI-compatible)                                          │
│  │  - company agent    │                     ▼                                                                   │
│  │    conversations   │        ┌──────────────────────────┐         ┌──────────────────────────────┐               │
│  │  - Operations      │◄───────│  Cloudflare Tunnel        │◄────────┤  (fallback) Gemini / Groq     │               │
│  │    approvals       │  poll  │  (outbound from Kaggle)   │         │  free-tier API                │               │
│  └─────────────────────┘        └──────────────▲───────────┘         └──────────────────────────────┘               │
│                                               │ outbound only                                                        │
└───────────────────────────────────────────────┼─────────────────────────────────────────────────────────────────────┘
                                                │
                                   ┌────────────┴───────────┐
                                   │ Kaggle GPU notebook    │
                                   │ vLLM, Qwen2.5-7B-      │
                                   │ Instruct on 2×T4,      │
                                   │ OpenAI-compatible      │
                                   │ endpoint, self-registers│
                                   └────────────────────────┘
```

**Key design decisions (do not change without recording in docs/DECISIONS.md):**

1. **Control plane on the VM, inference on Kaggle.** The Poppy agent client — signing keys, sessions, tokens, permissions — runs in a stable service on the VM so demo state survives Kaggle restarts. Kaggle only serves an OpenAI-compatible LLM endpoint.
2. **Outbound-only from Kaggle.** The notebook starts a Cloudflare quick tunnel and POSTs its URL to the agent runtime's registration endpoint. The runtime never needs to reach into Kaggle any other way. If the tunnel is down, the runtime falls back to a hosted free LLM API (or a scripted mock provider) so the demo cannot die with the notebook.
3. **Both sides of Poppy in one monorepo, two services.** The bank (Company) and the agent runtime (Personal Agent) are separate FastAPI services with separate domains, because poppy.json must be published on the company's domain and the agent's client metadata on the agent's domain.
4. **Postgres holds all state.** Sessions, tokens, pending operations, the agent job queue, and the bank's ledger. A restart of any component resumes the demo mid-flow.
5. **Everything Dockerized**, fronted by Caddy for automatic HTTPS.

### Domains

Use one domain with subdomains (a DuckDNS subdomain works for free; a real domain is better):

| Service | URL |
|---------|-----|
| Bank (Company) | `https://bank.<DOMAIN>` |
| Agent runtime | `https://agent.<DOMAIN>` |
| OAuth issuer | `https://bank.<DOMAIN>/poppy` (issuer URL; metadata per RFC 8414) |

`<DOMAIN>` is a build-time variable, e.g. `meridianbank.duckdns.org`.

---

## 4. Repository layout

```
poppy-bank-demo/
├── SPEC.md                    # this document
├── docs/
│   ├── DECISIONS.md           # ADRs (one paragraph each)
│   ├── BUILD_LOG.md           # SDK versions + spec version tested per build
│   └── RUNBOOK.md             # demo-day checklist (§11)
├── company/                   # Meridian Bank — the Poppy "Company"
│   ├── app/
│   │   ├── main.py            # FastAPI app factory
│   │   ├── poppy_discovery.py # /.well-known/poppy.json
│   │   ├── oauth.py           # issuer metadata, authorize, token, revocation
│   │   ├── sessions.py        # session validation (Poppy-Session JWT)
│   │   ├── api.py             # accounts/transactions/payments resources
│   │   ├── conversations.py   # company agent (type "poppy") endpoint
│   │   ├── operations.py      # Operations extension (approvals)
│   │   ├── web_pages.py       # bank site, login, consent, approval pages (Jinja2)
│   │   ├── agent_llm.py       # LLM client for the company agent (small model OK)
│   │   └── db.py              # SQLAlchemy models + session
│   ├── static/                # bank branding (Meridian teal/navy, fictional)
│   ├── templates/             # Jinja2 pages
│   ├── openapi.yaml           # the OpenAPI document offered in poppy.json
│   ├── tests/
│   ├── Dockerfile
│   └── pyproject.toml
├── agent/                     # Personal Agent runtime
│   ├── app/
│   │   ├── main.py            # FastAPI app
│   │   ├── chat.py            # chat UI + SSE activity feed
│   │   ├── poppy_client.py    # wraps Datalayer SDK: discovery→session→sign-in→API→conversations
│   │   ├── llm.py             # provider adapter: kaggle | hosted | mock (OpenAI-compatible)
│   │   ├── kaggle_registry.py # POST /internal/llm-register, heartbeat
│   │   ├── orchestrator.py    # conversation loop, tool-calling, scope escalation logic
│   │   └── db.py              # agent-side state (mirrors into same Postgres, separate schema)
│   ├── templates/  static/    # demo console UI
│   ├── tests/
│   ├── Dockerfile
│   └── pyproject.toml
├── worker/                    # Kaggle notebook
│   ├── poppy-worker.ipynb     # vLLM server + cloudflared + registration loop
│   └── README.md
├── deploy/
│   ├── docker-compose.yml     # company, agent, postgres, caddy
│   ├── Caddyfile
│   ├── .env.example
│   └── init/01_schema.sql     # schema + seed data
└── scripts/
    ├── validate.sh            # run community validator against bank.<DOMAIN>
    ├── smoke_test.py          # end-to-end script (§13 acceptance)
    └── demo_seed.py
```

---

## 5. Data model (Postgres)

One Postgres 16 container, two schemas: `bank` (company side) and `agent` (agent side). DDL in `deploy/init/01_schema.sql`; use SQLAlchemy 2.0 models in code.

### Schema `bank`

```sql
CREATE SCHEMA bank;

CREATE TABLE bank.users (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email         TEXT UNIQUE NOT NULL,          -- 'rahul@demo.example'
  password_hash TEXT NOT NULL,                 -- argon2; demo only
  display_name  TEXT NOT NULL,                 -- 'Rahul'
  created_at    TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE bank.accounts (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id      UUID NOT NULL REFERENCES bank.users(id),
  kind         TEXT NOT NULL CHECK (kind IN ('current','savings')),
  name         TEXT NOT NULL,                  -- 'Everyday Current', 'Rainy Day Savings'
  sort_code    CHAR(6) NOT NULL,               -- '040075'
  account_no   CHAR(8) NOT NULL,               -- '12345678'
  balance_p    BIGINT NOT NULL DEFAULT 0,      -- pence; NEVER floats for money
  currency     CHAR(3) NOT NULL DEFAULT 'GBP'
);

CREATE TABLE bank.payees (
  id        UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id   UUID NOT NULL REFERENCES bank.users(id),
  name      TEXT NOT NULL,                     -- 'Rivercrest Lettings'
  sort_code CHAR(6) NOT NULL,                  -- '236972'
  account_no CHAR(8) NOT NULL
);

CREATE TABLE bank.transactions (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  account_id  UUID NOT NULL REFERENCES bank.accounts(id),
  ts          TIMESTAMPTZ NOT NULL DEFAULT now(),
  amount_p    BIGINT NOT NULL,                 -- negative = debit
  description TEXT NOT NULL,                   -- 'Rent — Rivercrest Lettings'
  balance_after_p BIGINT NOT NULL,
  category    TEXT                             -- 'housing', 'groceries', 'income', 'transfer'
);

CREATE TABLE bank.payments (
  id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id    UUID NOT NULL REFERENCES bank.users(id),
  payee_id   UUID NOT NULL REFERENCES bank.payees(id),
  amount_p   BIGINT NOT NULL CHECK (amount_p > 0),
  status     TEXT NOT NULL DEFAULT 'pending'
             CHECK (status IN ('pending','approved','executed','rejected','failed')),
  reference  TEXT,
  operation_id UUID,                           -- links to the Poppy operation that approved it
  created_at TIMESTAMPTZ DEFAULT now(),
  executed_at TIMESTAMPTZ
);

-- Poppy company-side state
CREATE TABLE bank.poppy_sessions (
  id           TEXT PRIMARY KEY,               -- session id chosen by the agent
  user_ref     TEXT,                           -- opaque per-company user id issued at sign-in
  user_id      UUID REFERENCES bank.users(id), -- resolved only after sign-in
  created_at   TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE bank.oauth_codes   (code TEXT PRIMARY KEY, session_id TEXT, user_id UUID, scopes TEXT[], expires_at TIMESTAMPTZ, consumed BOOL DEFAULT FALSE);
CREATE TABLE bank.oauth_tokens  (token_hash TEXT PRIMARY KEY, session_id TEXT, user_id UUID, scopes TEXT[], kind TEXT CHECK (kind IN ('access','refresh')), dpop_jkt TEXT, expires_at TIMESTAMPTZ, revoked BOOL DEFAULT FALSE);
CREATE TABLE bank.operations    (id UUID PRIMARY KEY, session_id TEXT, user_id UUID, kind TEXT, payload JSONB, status TEXT DEFAULT 'pending' CHECK (status IN ('pending','approved','rejected','expired')), created_at TIMESTAMPTZ DEFAULT now(), resolved_at TIMESTAMPTZ);
```

### Schema `agent`

```sql
CREATE SCHEMA agent;

CREATE TABLE agent.conversations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE agent.messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  conversation_id UUID REFERENCES agent.conversations(id),
  role TEXT NOT NULL CHECK (role IN ('user','assistant','system','tool')),
  content TEXT NOT NULL,
  ts TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE agent.activity (          -- powers the live protocol log panel
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  ts TIMESTAMPTZ DEFAULT now(),
  event TEXT NOT NULL,                 -- 'discovery.fetched', 'session.created', 'oauth.authorize', ...
  detail JSONB
);

CREATE TABLE agent.llm_providers (
  name TEXT PRIMARY KEY,               -- 'kaggle' | 'hosted' | 'mock'
  base_url TEXT,
  status TEXT DEFAULT 'offline',       -- 'online' | 'offline'
  last_heartbeat TIMESTAMPTZ
);

CREATE TABLE agent.permissions (       -- standing user approvals shown in UI
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  conversation_id UUID,
  company_domain TEXT NOT NULL,
  scope TEXT NOT NULL,
  granted_at TIMESTAMPTZ DEFAULT now(),
  revoked_at TIMESTAMPTZ
);
```

### Seed data (`scripts/demo_seed.py`)

- User **Rahul** (`rahul@demo.example` / password `PoppyDemo!2026`), display name "Rahul".
- Accounts: **Everyday Current** £4,812.33; **Rainy Day Savings** £12,450.00.
- Payee: **Rivercrest Lettings** (rent £1,850.00, due the 1st).
- ~15 realistic transactions over the last 30 days (salary credit £6,200 on the 28th, groceries, transport, a subscription) with running balances that reconcile to the current balances. Money math in pence throughout.

---

## 6. Company service — Meridian Bank

FastAPI (Python 3.12), served at `https://bank.<DOMAIN>`. Use the **Datalayer Python SDK** (`personal-agent-protocol`) for protocol types wherever it exposes company-side helpers; `VERIFY` what the SDK provides before writing custom code, and prefer SDK models.

### 6.1 Discovery — `GET /.well-known/poppy.json`

Served with `Content-Type: application/json` and long cache headers. Shape (field names `VERIFY` against draft 0.3 / SDK):

```json
{
  "protocol_version": "0.3",
  "organization": {
    "name": "Meridian Bank",
    "domain": "bank.<DOMAIN>"
  },
  "auth": {
    "issuer": "https://bank.<DOMAIN>/poppy",
    "direct": { "scopes": [
      { "name": "accounts:read",     "changes": "none",        "personal_data": true },
      { "name": "transactions:read", "changes": "none",        "personal_data": true },
      { "name": "payments:create",   "changes": "irreversible", "personal_data": true }
    ]},
    "device": { "scopes": ["accounts:read", "transactions:read"] }
  },
  "agent": {
    "protocols": [
      { "type": "poppy", "endpoint": "https://bank.<DOMAIN>/poppy/conversations" }
    ]
  },
  "web": { "browser_session_endpoint": "https://bank.<DOMAIN>/poppy/browser-session" },
  "apis": [
    { "type": "openapi",
      "url": "https://bank.<DOMAIN>/openapi.yaml",
      "resource": "https://bank.<DOMAIN>/poppy",
      "description": "Meridian Bank accounts, transactions and payments" }
  ],
  "extensions": {
    "operations": { "version": "1", "endpoint": "https://bank.<DOMAIN>/poppy/operations" }
  }
}
```

`VERIFY`: exact scope-hint vocabulary (`changes` values, `personal_data`) and the Operations extension shape against the draft and the SDK; adjust names but keep the three-tier story (read / read / irreversible-write).

### 6.2 OAuth issuer

Implement OAuth 2.0 with the **authorization code + PKCE** flow ("Direct Sign-In") and the **device grant**, per the Poppy spec. Requirements:

- `GET /poppy/.well-known/oauth-authorization-server` — RFC 8414 metadata. `VERIFY` exact metadata path the spec requires.
- `GET /poppy/authorize` — the bank's own page: user logs in (seeded credentials), sees the requesting agent's identity (name, developer, verified status from the agent's client metadata), the requested scopes with plain-English descriptions and the `irreversible` badge on `payments:create`, and Approve / Deny buttons. This page is the emotional core of the demo — invest in the design.
- `POST /poppy/token` — issues access + refresh tokens, bound to the DPoP key (`dpop_jkt`, ES256) and the Poppy session. `VERIFY` the DPoP binding requirements in draft 0.3; use the SDK's helpers if present.
- `POST /poppy/revoke` — revocation endpoint.
- Access tokens: short-lived (15 min), JWT or opaque with hash stored in `oauth_tokens`. Refresh tokens: 30 days.
- Scopes are additive: an agent holding `accounts:read` that needs `payments:create` must go through `authorize` again (step 8 of the script). This escalation is the demo's second act — make it explicit in the UI.

### 6.3 Sessions

- `POST /poppy/sessions` (or the SDK's equivalent) — the agent names a new session (UUID) and begins signed-out. `VERIFY` the exact session-creation semantics: draft 0.3 has the agent choosing the session id and signing a Session JWT per request.
- Every Poppy request carries the session assertion in a header (`VERIFY` exact header name, e.g. `Poppy-Session`) — validate the ES256 JWT against the agent's published key (§7.3), check `session_id` exists, resolve `user_id` only if signed in.
- Guest sessions may converse (§6.5) without any account.

### 6.4 API (OpenAPI route)

Publish `openapi.yaml` describing, under base path `/poppy/api`:

| Method | Path | Scope | Behavior |
|--------|------|-------|----------|
| GET | `/accounts` | `accounts:read` | List the signed-in user's accounts (masked account numbers) |
| GET | `/accounts/{id}/transactions?limit=` | `transactions:read` | Recent transactions, newest first |
| POST | `/payments` | `payments:create` | Create a payment **proposal**; returns 201 with `status: "pending"` and an operation id. Does NOT move money. |
| GET | `/payments/{id}` | `payments:create` | Payment status |

- Auth: bearer access token (DPoP-bound) + valid Poppy session header. Insufficient scope → OAuth-style 403 with `insufficient_scope`.
- All responses include the session context so the demo console can show "same session, every channel".

### 6.5 Company agent conversations

`POST /poppy/conversations` — JSON-over-HTTPS messages (`VERIFY` exact envelope against the SDK). Behavior:

- Guest session: answer general questions about the bank ("Do you offer fee-free current accounts?" → canned/LLM answer about the fictional Everyday Current account, no personal data).
- Signed-in session: may answer account-specific questions by calling the same internal services, and can propose actions.
- The company agent is a small LLM call (hosted free-tier API is fine; it does not need the Kaggle GPU) with a strict system prompt: never disclose account data unless the session is signed in; propose, never execute.

### 6.6 Operations extension (approvals)

`VERIFY` the Operations extension contract (Datalayer SDKs implement "Operations v2"; the spec's extension is what the bank offers). Required behavior:

1. Agent posts an operation (kind `payment.create`, payload: payee, amount, reference, `changes: "irreversible"`) for a signed-in session.
2. Bank creates a pending operation row and returns a URL to an approval page.
3. The approval page (`GET /poppy/operations/{id}/approve`) shows: the agent identity, the exact payee, the exact amount in £, the word IRREVERSIBLE, and Approve / Reject. Requires the user to log in (the customer, not the agent).
4. On approval: operation → `approved`, the linked payment → `executed`, money moves atomically (debit current account, insert transaction row, update balance), `executed_at` set.
5. Agent polls `GET /poppy/operations/{id}` (or receives the resolution via the conversation channel — `VERIFY` which the extension defines) and reports back to the user.

### 6.7 Bank website

A credible 4-page fictional bank site: home, accounts overview (login required), the consent page, the approval page. Branding: Meridian Bank — teal/navy, clearly fictional, footer "Fictional bank for protocol demonstration. Not a real bank." No real-bank likeness.

### 6.8 Browser session (optional, build last)

`web.browser_session_endpoint` — lets a human-controlled browser join the agent's session. Implement a minimal version if time allows; mark as stretch in §13.

---

## 7. Agent runtime — the Personal Agent

FastAPI at `https://agent.<DOMAIN>`. This service is the Poppy "Personal Agent" acting for the user.

### 7.1 Demo console (chat UI)

Single-page app (server-rendered + HTMX or a small React page; keep dependencies light):

- Chat pane: user ↔ agent messages.
- **Activity panel (right side)** — the demo's proof engine. Streams (SSE) every protocol event with timestamps: `discovery.fetched` (show the poppy.json), `session.created`, `conversation.message`, `oauth.authorize` (link), `token.granted` (scopes), `api.call` (method, path, scope used), `operation.proposed`, `operation.approved`, `payment.executed`. Each event renders as a chip; clicking shows the raw JSON.
- Permission strip: current scopes held per company, each with a Revoke button (demo step 10).
- LLM provider indicator: kaggle / hosted / mock, with heartbeat age.

### 7.2 Orchestrator

The conversation loop (Python, any-agent framework optional — plain loop is fine):

1. User message → LLM with tools: `discover_company(domain)`, `ask_company_agent(text)`, `call_api(path, method, body)`, `request_scope(scope)`, `propose_payment(payee, amount)`, `check_operation(id)`.
2. The LLM must follow the demo narrative naturally; system prompt encodes the script of §1 as guidance, not hard-coded key words.
3. Scope handling: when a tool call needs a scope the agent doesn't hold, the orchestrator runs the OAuth flow: generates PKCE pair, builds the authorize URL, emits an `oauth.authorize` event with the link, and pauses the conversation with "I need your permission at Meridian Bank — please approve on their page." The chat UI polls for completion; on callback the loop resumes.
4. Payment: LLM proposes → orchestrator creates the operation, surfaces the approval link, waits for resolution, then confirms to the user with the new balance.

### 7.3 Agent identity & client metadata

- ES256 signing key pair for session assertions + one DPoP key per user per company. Generate at first boot, store in Postgres (demo) — never in the repo.
- Serve client metadata at `https://agent.<DOMAIN>/poppy/client.json` (the client_id URL pattern; `VERIFY` exact fields): agent name "Aria — Rahul's personal agent", developer "Demo", verified badge false, redirect URIs.
- Per-company pseudonymous user ids: one stable random id per company, stored agent-side. Show this in the activity panel ("Meridian sees you as user `f3a9…`, not your email") — it is one of Poppy's best privacy stories.

### 7.4 LLM provider adapter

OpenAI-compatible interface (`/v1/chat/completions` semantics internally). Providers, in priority order:

1. **kaggle** — base URL registered by the notebook (§8). Model: Qwen2.5-7B-Instruct served by vLLM on 2×T4 (16 GB each; run one instance, `--max-model-len 8192`, tensor parallel 1 — a 7B fits on one T4).
2. **hosted** — Gemini or Groq free tier via their OpenAI-compatible endpoints; API key from env.
3. **mock** — scripted responses that walk the §1 script deterministically. Used for CI and as demo-day last resort.

Failover: if kaggle heartbeat is stale > 90 s, switch to hosted, then mock. Provider switch is visible in the UI (honest, and a good talking point).

### 7.5 Kaggle registry endpoints (internal, API-key auth)

- `POST /internal/llm/register` `{base_url, model, secret}` — notebook announces itself.
- `POST /internal/llm/heartbeat` — every 30 s; flips provider online.

---

## 8. Kaggle worker notebook

`worker/poppy-worker.ipynb`, designed for interactive sessions (max 12 h, ~30 h/week GPU quota, idle timeout 20–60 min — keep a keep-alive cell that pings itself every 5 min).

Cells:
1. Install: `vllm`, `cloudflared` (download binary from GitHub releases), `requests`.
2. Start vLLM server on `:8001`: `Qwen/Qwen2.5-7B-Instruct`, OpenAI-compatible.
3. Start Cloudflare quick tunnel: `cloudflared tunnel --url http://localhost:8001` → parse the `https://<random>.trycloudflare.com` URL from logs.
4. Register with the agent runtime: POST the tunnel URL + shared secret to `https://agent.<DOMAIN>/internal/llm/register`; heartbeat every 30 s; keep-alive loop.
5. A status cell printing the tunnel URL, quota notes, and "restart this notebook 30 minutes before the demo".

README covers: phone verification needed for GPUs, quota reset Saturday midnight UTC, tunnel URL changes each restart (registration handles it), and the fallback story.

---

## 9. Deployment

Target: Oracle Cloud Always Free — VM.Standard.A1.Flex, 4 OCPU / 24 GB, Ubuntu 24.04, 100 GB boot volume. Open ingress 80/443 only.

- `deploy/docker-compose.yml`: services `company`, `agent`, `postgres:16`, `caddy:2` (one Caddyfile, two hostnames, automatic Let's Encrypt).
- `deploy/.env.example` with every variable in §10.
- First-boot: `docker compose up -d`, run `scripts/demo_seed.py`, run `scripts/validate.sh` (community validator against `bank.<DOMAIN>/.well-known/poppy.json` + OAuth metadata — must pass before any demo).
- The RUNBOOK (`docs/RUNBOOK.md`) is the demo-day checklist: T-30 min restart Kaggle notebook; T-10 min smoke test; T-5 min open console; backup recording ready.

---

## 10. Configuration & secrets

| Variable | Used by | Notes |
|----------|---------|-------|
| `DOMAIN` | compose, Caddy | e.g. `meridianbank.duckdns.org` |
| `POSTGRES_PASSWORD` | all | |
| `BANK_SESSION_SECRET` / agent signing keys | generated at boot, stored in DB | |
| `KAGGLE_SHARED_SECRET` | agent + notebook | for `/internal/*` endpoints |
| `HOSTED_LLM_API_KEY`, `HOSTED_LLM_BASE_URL`, `HOSTED_LLM_MODEL` | agent | Gemini/Groq fallback |
| `AGENT_PUBLIC_URL`, `BANK_PUBLIC_URL` | both | |

Secrets never in git. `.env` on the VM only.

---

## 11. Security & honesty notes (put these in the README)

- Demo-only code: no real auth hardening, rate limiting, or audit trails. The README says so in bold.
- Fictional bank, fictional data, no real-bank branding or likeness.
- OAuth pages clearly labelled as a demonstration.
- Keys in Postgres is acceptable for a demo and unacceptable for production — noted in code comments.

---

## 12. Build order (milestones)

Each milestone ends with something demoable or testable. Do them in order:

| M | Scope | Done when |
|---|-------|-----------|
| 1 | Repo skeleton, docker-compose, Postgres schema + seed, Caddy, two Hello-World FastAPI services on HTTPS | `curl https://bank.<DOMAIN>/healthz` green from the internet |
| 2 | Bank: poppy.json + validator passing, OAuth metadata, login/consent pages (stub tokens) | `scripts/validate.sh` passes |
| 3 | Agent: SDK client — discovery, guest session, conversation with company agent (mock LLM) | Guest Q&A works end-to-end in console with activity panel |
| 4 | Sign-in: PKCE flow, token issuance, `accounts:read` + `transactions:read`, balance + transactions in chat | Steps 1–6 of the script pass |
| 5 | Payments: `payments:create` scope escalation, Operations approval page, atomic ledger update | Steps 7–9 pass; balances reconcile |
| 6 | Kaggle worker: vLLM + tunnel + registration + failover | Real model answers; kill notebook → hosted fallback engages |
| 7 | Polish: revocation (step 10), blocked-bank contrast mock, browser-session stretch, RUNBOOK, recording | Full script timed under 5 minutes |

---

## 13. Acceptance criteria (final gate)

`scripts/smoke_test.py` must pass all of these headlessly (using the mock LLM):

1. `GET /.well-known/poppy.json` returns valid JSON that the community validator accepts.
2. A guest session can hold a conversation and receive no account data.
3. OAuth code+PKCE flow issues a DPoP-bound token for `accounts:read, transactions:read`.
4. `GET /accounts` with the token returns both seeded accounts; without scope → 403 `insufficient_scope`.
5. Payment proposal without `payments:create` → 403; with it → 201 pending + operation URL.
6. Approval page login + approve → payment `executed`; current account debited exactly £185,000 p; transaction row exists; balances reconcile.
7. Revoking the token → subsequent API call → 401.
8. Restart of the agent container mid-conversation → conversation resumes from Postgres state.
9. Full §1 script in the console with the activity panel showing at least: discovery, session, conversation, authorize, token, api.call, operation.proposed, operation.approved, payment.executed.

---

## 14. References

- Poppy draft 0.3 spec — https://personalagentprotocol.org/docs/0.3/spec
- Poppy introduction — https://personalagentprotocol.org/docs/introduction
- Poppy repo — https://github.com/personal-agent-protocol/personal-agent-protocol
- Datalayer Python/TypeScript SDKs (draft 0.3) — https://github.com/datalayer/personal-agent-protocol
- Community demos page — https://personalagentprotocol.org/docs/demos
- PDA validator — https://github.com/adewale/pda
- Awesome Poppy (ecosystem list) — https://github.com/JaviChulvi/awesome-poppy
- Poppyseed (Next.js company-side implementation, draft 0.3) — https://github.com/akshay5995/poppyseed
- descope/poppy-js (agent-side TS client) — https://github.com/descope/poppy-js
- Kaggle GPU usage docs — https://www.kaggle.com/docs/efficient-gpu-usage and https://www.kaggle.com/docs/notebooks
- Context: banks blocking agents vs Poppy partners (Citi statement; BofA, Wells Fargo, BBVA, Synchrony, Chime as design partners) — https://www.bankingnewsai.com/agentic-banking/agent-access and https://sierra.ai/uk/blog/poppy
