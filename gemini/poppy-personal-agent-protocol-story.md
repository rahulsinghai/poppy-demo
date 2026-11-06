# The Front Door of the Agentic Web: The Story of the Personal Agent Protocol (Poppy)

## Prologue: When Synthetic Minds Met the Legacy Web

For twenty-five years, the fundamental social contract of the internet rested upon a single plain-text file hosted at the root of web servers: `robots.txt`. Search engine web crawlers like Googlebot and Bingbot checked this file, respected rate limits, parsed static HTML tags, and left host servers in peace. In exchange for bandwidth, web platforms gained indexation and discovery. It was an equilibrium built on reciprocal value and predictable, document-retrieval semantics.

By late 2026, that equilibrium evolved dramatically.

The arrival of autonomous personal AI agents marked a paradigm shift in digital interaction. Unlike traditional search spiders that fetched static indices, modern LLM-driven agents receive open-ended, multi-step goals from human users—such as booking travel, negotiating rates, managing shopping carts, or accessing financial accounts. When faced with access barriers, these autonomous agents do not behave like traditional web crawlers. When an endpoint returns a 403 Forbidden status, a standard crawler logs the rejection and terminates. An autonomous reasoning loop, however, interprets the error code as an environmental condition, adjusts its execution trajectory, and attempts to navigate around the restriction.

This structural disconnect reached a turning point in October 2026. Telemetry logs from public platforms revealed that unconstrained AI agent loops generated millions of complex graph queries, causing database latency spikes and system degradation. Automated agents executed client-side JavaScript, simulated human interaction patterns, and attempted to complete multi-step tasks across public endpoints.

These events demonstrated a decisive architectural truth: unauthenticated, unconstrained browser automation had reached its operational limit. The web required a new front door—a standardized protocol that could verify an agent's identity, enforce explicit customer consent, and establish structured execution boundaries without requiring automated assistants to navigate human web forms.

---

> **[IMAGE REFERENCE 1: The Transition from Legacy Scraping to Governed Agent Access]**  
> *Recommended Image Source: `1_spdF3AOfIExA9gsa9Vz7eQ.webp` or `hero.webp`*  
> *Description: Side-by-side visual comparison showing the friction, timeouts, and CAPTCHA hurdles of legacy web automation versus a streamlined, single-visit front-door protocol with explicit user authorization.*

---

## Act I: The Birth of Poppy and the Architecture of Consent

On October 6, 2026, Sierra co-founders Bret Taylor and Clay Bavor, alongside Meta's engineering leadership, unveiled the Personal Agent Protocol—code-named **Poppy**. Announced initially with a coalition including Shopify, Stripe, Walmart, Genesys, Rocket Companies, and Instinct, the open standard quickly expanded three days later with 35 additional design partners, including OpenAI, Bank of America, Visa, Mastercard, PayPal, Okta, Cloudflare, Target, and United Airlines.

Poppy was not created to replace tool-calling standards or lower-level agent frameworks. Instead, it was established as an open **consent, session, and identity layer** positioned directly between a user's personal agent and the enterprise systems of a business.

```
+-----------------------------------------------------------------------+
|                         PERSONAL AI AGENT                             |
|               (Acts for User via Mobile, Web, or App)                 |
+-----------------------------------------------------------------------+
                                   |
                         [ Poppy Protocol Layer ]
             (Discovery, OAuth Consent, Session Continuity)
                                   |
         +-------------------------+-------------------------+
         |                         |                         |
         v                         v                         v
  +--------------+          +--------------+          +--------------+
  |   WEBSITE    |          |  OPENAPI/MCP |          |  ENTERPRISE  |
  |  (Browsing)  |          |    (APIs)    |          |  AI AGENT    |
  +--------------+          +--------------+          +--------------+
```

### The Five Core Building Blocks of Poppy

The architecture of Poppy establishes five sequential building blocks that transform automated traffic into authenticated, auditable sessions:

1. **Discovery via Standardized Manifest**: An enterprise publishes a machine-readable document at a standard URI: `/.well-known/poppy.json`. This discovery manifest advertises available interaction channels, supported OpenAPI and Model Context Protocol (MCP) endpoints, authentication mechanisms, and organizational capabilities.
2. **Session Initiation and Identity**: A personal agent initiates a session on behalf of its user. Crucially, the interaction can begin in a **guest state**—allowing the agent to perform public, read-only tasks such as checking product availability, reviewing store hours, or reading return policies without requesting account credentials. Throughout every interaction, the agent explicitly identifies itself with a signed header, ensuring host systems recognize the request source.
3. **OAuth Sign-In & Explicit Scope Granting**: When a task requires access to a customer's personal account (e.g., initiating a refund or booking a flight), the customer authenticates through standard OAuth flows (such as Authorization Code with PKCE or Device Authorization) directly on the business's own domain. The user retains complete authority over the scope of access granted—selecting read-only or read-write permissions—while the personal agent never sees or stores the user's primary account credentials.
4. **Cross-Channel Session Continuity**: Poppy introduces a cryptographically signed session token (`Poppy-Session` JWT). This single token persists across all channels offered by the enterprise. An initial inquiry made on a public website, an API payload submitted via MCP, and a follow-up conversation held with the company's internal customer-service agent share the exact same visit session. The customer is never forced to re-authenticate, and the business maintains an integrated, end-to-end view of the customer journey.
5. **Multi-Channel Execution Routes**: Enterprises determine precisely which backend channels to expose to incoming personal agents based on operational capabilities:
   - **Regular Web Pages**: Agents navigate structured site pages with user permissions applied dynamically.
   - **Direct APIs**: Agents invoke OpenAPI or MCP tools for high-efficiency structured data exchanges.
   - **Enterprise AI Agents**: Personal agents communicate directly with company-hosted AI agents to resolve complex, conversational workflows such as warranty claims or loan pre-approvals.

---

> **[IMAGE REFERENCE 2: The Personal Agent Protocol Front Door Architecture]**  
> *Recommended Image Source: `image.webp` or `hero.webp`*  
> *Description: Diagram depicting a mobile device running a personal agent connecting through the Poppy Front Door via OAuth, branching into Website, API, and Business Agent channels across brand platforms.*

---

### The Five Granular Permission Tiers

To give businesses precise control over automated interactions, Poppy codifies five distinct operational tiers:

| Permission Level | Operational Boundary | Functional Scope & Actions |
| :--- | :--- | :--- |
| **Blocked** | Complete Restriction | Autonomous agent access is prohibited across all channels. |
| **Search Only** | Public Read-Only | Agents may query public product catalogs, FAQs, and store locators. |
| **View Private Info** | Authenticated Read-Only | Agents read user-specific orders, delivery status, or balances after OAuth sign-in. |
| **Execute Actions** | Scoped Read/Write | Agents execute pre-approved transactions (e.g., booking a room) with explicit consent. |
| **Full Access** | Unrestricted Delegation | Agents act fully on the customer's behalf across all account capabilities. |

This tiered structure eliminates the binary choice between blocking agents entirely or granting unconstrained web access.

---

## Act II: Hardening Execution Boundaries and Code Mode Efficiency

While protocol manifests establish who an agent is and what it may do, maintaining operational stability at the execution boundary requires structured containment at the infrastructure layer. Autonomous agent execution must be bounded by deterministic security proxies and isolated runtimes.

### Micro-VM Isolation and Egress Controls

To ensure system stability, production agent architectures decouple the central planning model from the task execution environment. All tool calls, web rendering, and dynamic code executions run inside ephemeral **micro-VM sandboxes** (such as AWS Firecracker).

```
+-------------------------------------------------------------------------+
|                         ORCHESTRATOR HOST                               |
|          (Receives User Prompt -> Compiles Execution Graph)             |
+-------------------------------------------------------------------------+
                                     |
                         [ Ephemeral Task Execution ]
                                     v
+-------------------------------------------------------------------------+
|                  MICRO-VM SANDBOX (Firecracker)                         |
|  - Non-Root User Execution     - Ephemeral / Isolated File System       |
|  - Active Autonomous Agent     - Lifetime: Single Sub-Task              |
+-------------------------------------------------------------------------+
                                     |
                          [ Outbound Tool Call ]
                                     v
+-------------------------------------------------------------------------+
|                        EGRESS SECURITY PROXY                            |
|  - Filters Mutating Verbs (POST/PUT/DELETE) during browsing mode         |
|  - Enforces Domain Whitelisting & Path Validation                       |
+-------------------------------------------------------------------------+
                         /                       \
        (Blocked Path)  /                         \  (Allowed Path)
                       v                           v
      +---------------------------------+  +-------------------------------+
      | Unlisted Forms & Editors        |  | Target Platform Server        |
      | [ UNAUTHORIZED STATE MUTATION ] |  | (/.well-known/poppy.json)     |
      +---------------------------------+  +-------------------------------+
```

The egress security proxy acts as a network gatekeeper. During exploratory browsing tasks, the proxy automatically restricts mutating HTTP verbs (`POST`, `PUT`, `DELETE`, `PATCH`), preventing an exploratory loop from modifying database state or submitting unverified forms. Outbound requests are validated against domain whitelisting rules, while machine-readable schemas returned by host servers are cryptographically verified using Ed25519 client assertions before any API call is allowed to complete.

---

> **[IMAGE REFERENCE 3: Ephemeral Micro-VM Sandbox and Egress Security Proxy Architecture]**  
> *Recommended Image Source: `1_Ir0RyLBWPNmcFWQLxZfXwg.png`*  
> *Description: Technical architectural diagram illustrating an autonomous agent inside a Firecracker micro-VM sandbox issuing tool calls through an Egress Security Proxy with verb filtering and OAuth assertion signing.*

---

### Telemetry Proof: Unconstrained Loops vs. Hardened Egress

The operational impact of implementing Poppy manifests alongside egress proxies is demonstrated in edge gateway telemetry data. Unconstrained headless agent loops parsing raw HTML exhibit high error rates and database latency spikes. Once Poppy manifests and direct JSON-LD schemas replace unstructured web scraping, system performance stabilizes dramatically.

---

> **[IMAGE REFERENCE 4: Grafana Edge Gateway Telemetry — Pre- vs. Post-Hardening Metrics]**  
> *Recommended Image Source: `1_Jk16de7mWv8IdFeIbjAsug.png`*  
> *Description: Dual-panel Grafana dashboard showing unconstrained pre-hardening agent failure rates of 34.2% and P99 latency of 4,850ms dropping to 0.01% failure rate and 42ms latency post-hardening with Poppy manifests.*

---

### The Code Mode Efficiency Revolution

Beyond isolation, modern agentic protocols address the critical bottleneck of **context window bloat**. In classic direct tool calling, an AI model receives every tool schema in its system prompt, selects one tool, waits for the response to travel back through its context window, and repeats the cycle sequentially.

Under **Code Mode**, the model is provided with a typed API interface and writes a single executable script containing loops, conditionals, and data filters. The script executes locally inside the secure sandbox, returning only the final processed outcome back to the language model.

```
DIRECT TOOL CALLING (High Context Overhead):
[ Model ] ---> (Call Tool A) ---> [ Tool A ] ---> (Full Response Back to Model)
[ Model ] ---> (Call Tool B) ---> [ Tool B ] ---> (Full Response Back to Model)
[ Model ] ---> (Call Tool C) ---> [ Tool C ] ---> (Full Response Back to Model)
Result: 3 Round Trips, Large Payload Bloat in Context Window.

CODE MODE (Sandbox Execution Efficiency):
[ Model ] ---> (Generates 1 Script) ---> [ Micro-VM Sandbox ]
                                             |-> Executes Tool A
                                             |-> Executes Tool B
                                             |-> Executes Tool C
[ Model ] <--- (Returns Final Result Only) <-+
Result: 1 Round Trip, Up to 99.9% Reduction in Token Consumption.
```

Empirical benchmarks demonstrate significant token savings when shifting from direct tool calling to Code Mode execution across different workload complexities:

- **Simple Tasks (Cloudflare Demo)**: 32% token reduction by executing local loops rather than individual requests.
- **AIMultiple Benchmark (Net Average)**: 77.4% net token reduction across standard operational benchmarks.
- **31-Event Calendar Task**: 81% reduction in context window consumption.
- **Bifrost Gateway (500+ Tools)**: 92.8% token savings achieved via context-gated tool discovery.
- **Anthropic Code Execution Workflows**: Token footprint reduced from ~150,000 tokens down to 2,000 tokens (98.7% reduction).
- **Cloudflare Full API Suite (2,500 Endpoints)**: 99.9% token reduction, compressing a 1.17-million-token documentation context into two core sandbox methods (`search` and `execute`).

---

> **[IMAGE REFERENCE 5: Reported Token Savings of Code Mode vs. Direct Tool Calling]**  
> *Recommended Image Source: `1_1qKvS1aKNQSiQJJnrm-I9w.webp` or `1_qQ5jrSi-Da6Y3qpWROUC8w.webp`*  
> *Description: Bar chart displaying token reduction percentages across Cloudflare, AIMultiple, Bifrost, and Anthropic benchmarks, alongside a structural comparison of Direct Tool Calling versus Code Mode sandbox execution.*

---

## Act III: The Personal Agent Landscape: Muse, Dots, and PACT

The arrival of the Personal Agent Protocol coincided with a broader wave of consumer and enterprise personal AI agent deployments across 2026. Personal agents evolved beyond basic conversational chatbots into persistent digital partners capable of autonomous task execution.

```
               THE 2026 PERSONAL AI AGENT ECOSYSTEM
 +-------------------+ +-------------------+ +-------------------+ +-------------------+
 |     META MUSE     | |    OPENAI DOTS    | |     GROK BOT      | |     INSTINCT      |
 |  (Consumer/Admin) | |   (Work Projects) | |  (AI Teammates)   | |  (Text/Call Ops)  |
 +-------------------+ +-------------------+ +-------------------+ +-------------------+
 | - Muse Secure VM  | | - GPT-6 Astra     | | - SuperGrok       | | - No Native App   |
 | - Sentinel Agent  | | - 4,000+ Apps     | | - Multi-Bot Group | | - Proactive Calls |
 | - Stripe Link Pay | | - Custom Rules    | | - Local Routines  | | - Text Messaging  |
 +-------------------+ +-------------------+ +-------------------+ +-------------------+
```

### 1. Meta Muse: The Cloud-VM Personal Assistant
Launched in September 2026, Meta Muse is an always-on personal AI agent accessible via WhatsApp, iOS, Android, and web. Muse operates on a dedicated **Muse Secure VM**—an isolated cloud computer containing the agent's browser, data, and credentials. 

Security is enforced through a dual-agent structure: a separate system-level **Sentinel Agent** monitors outbound actions. Nothing generated by Muse reaches the internet unless approved by Sentinel. Furthermore, payment checkout is handled via **Link by Stripe**, which issues single-use virtual card numbers to protect credit card details, backed by built-in agent purchase protections.

### 2. OpenAI Dots: The Enterprise Task Force
Unveiled at OpenAI DevDay in September 2026, OpenAI Dots are personal AI agents built for workplace execution. Powered by GPT-6 Astra, Dots operate inside ChatGPT, Slack, and Microsoft Teams, connecting to over 4,000 enterprise applications. Security governance is managed via **Custom Rules**, allowing organizations to explicitly define allowed, approval-required, or blocked agent actions.

### 3. Decagon & PACT (Personal Agent Consent & Trust Protocol)
Developed by Decagon in collaboration with Instinct, **PACT** is an open protocol built on top of Agent2Agent (A2A) and OAuth 2.0. PACT establishes a structured three-step interaction model for business-to-agent communication:
1. **Discover & Connect**: Personal agents locate an enterprise's A2A Agent Card to inspect available permission scopes (e.g., `orders:read`, `orders:cancel`).
2. **Authenticate & Consent**: Account delegation is established via OAuth device authorization flows. The user logs in directly on the enterprise domain, granting explicit consent while keeping credentials completely isolated from the agent.
3. **Delegated Execution**: Requests carry both the personal agent's identity and a short-lived customer delegation token. The host system verifies authorization before executing business workflows within permitted boundaries.

---

> **[IMAGE REFERENCE 6: The 2026 Landscape of Personal AI Agents]**  
> *Recommended Image Source: `1_xXSR7gLMAO5FXXtS_RipfA.webp`*  
> *Description: Creative illustration depicting the distinct personas and operational focus areas of modern personal AI agents including Meta Muse, OpenAI Dots, Grok Bot, and Instinct.*

---

### User Permission Controls: You Hold the Keys

Across all modern personal agent protocols, consumer trust depends on explicit, granular permission toggles. Users manage access levels directly from their personal agent dashboard, choosing whether an agent can view orders, modify reservations, or save delivery addresses.

---

> **[IMAGE REFERENCE 7: Granular User Consent and Permission Control Interface]**  
> *Recommended Image Source: `1_0jVg9P3q1mVsFY5ZbBd6Jg.webp`*  
> *Description: Mobile UI mockup titled "You Hold the Keys" showing user permission toggles for store account access, including options to view orders, cancel orders, or save address details.*

---

## Act IV: The Economics of Data Settlement in Agentic Commerce

As personal agents assume responsibility for e-commerce transactions, commercial enterprises face a subtle financial and analytical challenge: **the divergence between financial settlement and data settlement**.

Every retail sales channel settles two distinct assets upon completing a transaction:
1. **Financial Settlement**: The monetary transaction (revenue, payment terms, take rates, fee schedules).
2. **Data Settlement**: The behavioral, identity, and permission signals returned to the merchant alongside the sale.

While commerce teams negotiate financial terms rigorously, data settlements are frequently accepted in whatever default structure a third-party platform provides.

```
THE FOUR SETTLEMENT DIMENSIONS OF A COMMERCE TRANSACTION
 +-------------------------------------------------------------------------+
 | 1. BEHAVIORAL SIGNAL : Browsing path, comparative views, dwell time.     |
 | 2. IDENTITY SIGNAL   : Durable customer profile key across visits.       |
 | 3. TRANSACTION DATA  : Order line items, financial total, timestamp.     |
 | 4. PERMISSION ACCESS : Direct consent and contact channel rights.        |
 +-------------------------------------------------------------------------+
```

### Analysis of Six E-Commerce Checkout Channels

Consider a single product—a $180 jacket—purchased across six distinct e-commerce channels. While every transaction books $180 in financial revenue, the data settlement varies dramatically based on channel rules:

| E-Commerce Channel Surface | Behavioral Data | Identity Signal | Transaction Data | Permission Access |
| :--- | :---: | :---: | :---: | :---: |
| **1. Merchant Direct Site** | Full Returned | Full Returned | Full Returned | Full Granted |
| **2. Instagram (Off-Site Checkout)** | Partial (Opt-in) | Full Returned | Full Returned | Full Granted |
| **3. TikTok Shop (Native Checkout)** | Withheld | Withheld | Full Returned | Withheld |
| **4. Personal Agent (Single-Use Card)** | Withheld | Withheld | Full Returned | Withheld |
| **5. Merchant-Operated Agent** | Full Returned | Full Returned | Full Returned | Full Granted |
| **6. Agent-to-Agent (Poppy Session)** | Full Returned | Full Returned | Full Returned | Full Granted |

When a personal agent purchases on behalf of a user using a single-use virtual credit card (such as Stripe Link), the merchant receives financial payment but loses behavioral history, customer profile resolution, and long-term contact permissions. The payment card number—historically used as an identity resolution key in retail analytics—no longer persists.

---

> **[IMAGE REFERENCE 8: Six Sales Channels, Six Data Settlements Comparison Matrix]**  
> *Recommended Image Source: `1_GEh2PvU568JWyV5D2jAOYw.webp`*  
> *Description: Comparative matrix diagram titled "Same Sale. Six Settlements" showing data capture variations (Behavior, Identity, Transaction, Permission) across six retail checkout channels.*

---

### Analytical Distortions and LTV Metrics

This divergence introduces specific analytical distortions into corporate metrics:

- **The Flattering LTV Mirage**: As off-site agent transactions grow, unjoinable revenue increases. If lifetime value (LTV) models calculate metrics using only visible, account-registered customers, measured LTV drifts upward while actual customer retention falls. The sample becomes self-selecting, creating a dashboard that looks increasingly flattering while obscuring business realities.
- **Conversion Rate Denominator Errors**: Dividing total order volumes (which include background agent transactions) by direct website browsing sessions inflates conversion rate figures, masking underlying changes in human site traffic.
- **Task-Based Metrics vs. Pageviews**: Agent interactions cannot be evaluated through traditional pageview volume. A personal agent making ten API retries to modify a reservation represents a single task execution, not ten instances of deep brand engagement. Modern analytics must evaluate **completed task rate, task duration, and escalation rates**.

---

## Act V: Protocol Synergy: How Poppy, MCP, and A2A Coexist

A common point of confusion in agentic systems is how newly introduced protocols interact with existing standards. Poppy does not compete with or replace the Model Context Protocol (MCP) or Google's Agent2Agent (A2A) protocol. Instead, each standard operates at a distinct layer in the agentic technology stack.

```
+-------------------------------------------------------------------------+
|                        POPPY PROTOCOL (Front Door)                      |
|      Identifies Personal Agent, Establishes OAuth Session & Consent     |
+-------------------------------------------------------------------------+
                                     |
         +---------------------------+---------------------------+
         |                                                       |
         v                                                       v
+----------------------------------+           +----------------------------------+
|   MODEL CONTEXT PROTOCOL (MCP)   |           |    AGENT-TO-AGENT (A2A / PACT)   |
|   (Agent-to-Tool Interface)      |           |    (Horizontal Delegation)       |
| - Executes structured API calls  |           | - Peer agent task delegation     |
| - Accesses database resources    |           | - Multi-agent collaboration      |
+----------------------------------+           +----------------------------------+
```

### Comparative Analysis of Protocol Layers

| Technical Dimension | Personal Agent Protocol (Poppy) | Model Context Protocol (MCP) | Agent2Agent Protocol (A2A) |
| :--- | :--- | :--- | :--- |
| **Primary Focus** | Consent, Session & Identity Layer | Tool & Data Access Layer | Agent-to-Agent Delegation Layer |
| **Target Relationship** | Personal Agent ↔ Enterprise Business | Agent Client ↔ Host Tool Server | Autonomous Agent ↔ Peer Remote Agent |
| **Discovery Mechanism** | `/.well-known/poppy.json` | MCP Server Capabilities Listing | A2A Agent Card Specification |
| **State Management** | Cross-Channel Customer Session | Protocol-Level Stateless (July 2026) | Stateful Task Lifecycle Management |
| **Authorization Model** | OAuth 2.1 + DPoP Scoped Tokens | Tool-Level Resource Scopes | Signed Task Assertion Tokens |
| **Execution Surface** | Web, OpenAPI, MCP, Company Agent | Direct Tools, Resources, Prompts | Remote Agent Skills & Reasoning |

### The Layered Banking Workflow Example

To understand how these three protocols operate in unison, consider a practical banking scenario:

1. **Poppy (Identity & Consent Layer)**: A customer instructs their personal agent to review account statements and reconcile an unexpected billing transaction. The personal agent locates the bank's `poppy.json` manifest, initiates an OAuth session, and obtains a customer-approved token scoped to `disputes:write`.
2. **MCP (Tool & Data Layer)**: Operating within the authorized Poppy session, the personal agent invokes the bank's internal MCP tool server to fetch transaction ledgers and balance histories.
3. **A2A (Peer Delegation Layer)**: The personal agent delegates the technical financial review to an independent financial audit agent using A2A task-passing specifications.
4. **Execution & Audit**: The bank verifies the signed `Poppy-Session` token and delegation claims before executing the transaction review inside its backend core.

---

## Epilogue: The Five-Point Agent Deployment Standard

As autonomous personal agents transform digital commerce and enterprise operations, software engineering teams must adopt disciplined systems standards to deploy agentic interfaces safely.

### The 5-Point Architectural Deployment Checklist

1. **Strict Egress Method Stripping**: Configure agent execution runtimes to restrict outbound `POST`, `PUT`, `DELETE`, and `PATCH` HTTP verbs during exploratory web browsing mode. Reserve write access strictly for validated API tools.
2. **Transparent Identity Declarations**: Structure outbound request headers to declare agent identity, operating platform, and emergency contact URIs (e.g., `User-Agent: PersonalAgent/1.0 (+agent-policy; ops@company.com)`).
3. **Poppy Manifest Pre-Flight Probing**: Implement mandatory pre-flight checks for `/.well-known/poppy.json` before initiating DOM rendering. When structured manifests are present, execute tasks via direct JSON schemas to eliminate unstructured web scraping.
4. **Ephemeral Micro-VM Sandbox Isolation**: Isolate dynamic code execution and browser rendering inside disposable micro-VM containers with strict execution timeouts (max 15 seconds) to prevent cross-session memory contamination.
5. **Dual Revenue & Settlement Tracking**: Ingest e-commerce order streams into analytics engines with explicit channel surface attributes, publishing both total revenue and identifiable revenue metrics to maintain accurate customer LTV models.

The emergence of the Personal Agent Protocol marks the transition of the agentic web from an unconstrained frontier into a governed, authenticated ecosystem. By establishing a shared front door based on mutual identity, explicit user consent, and standardized multi-channel sessions, Poppy ensures that autonomous personal agents and enterprise platforms can interact with confidence, security, and trust.
