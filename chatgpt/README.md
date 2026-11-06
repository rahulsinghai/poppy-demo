# Poppy (Personal Agent Protocol): Architecture, MCP and A2A comparison

Released October 2026

Emerging standard – draft specification

## The key distinction for senior leadership:

- MCP standardises how an AI agent accesses tools, data and capabilities.
- A2A standardises how independent AI agents communicate, delegate and collaborate.
- Poppy standardises how a person's personal AI agent discovers a business, establishes an authorised session and acts on the person's behalf, within permissions granted by the customer and the business.

Poppy is not intended to replace MCP or A2A. It addresses an important gap in agentic AI: identity, customer consent and delegated authority when an AI agent interacts with an organisation.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://personalagentprotocol.org\&sz=32)

Personal Agent Protocol+2

## 1. What exactly is Poppy?

Poppy was announced by Sierra and Meta on 6 October 2026, with industry partners including OpenAI subsequently participating in its design-partner programme. It is an open protocol intended to establish a standard relationship between:

1. Personal Agent: An AI agent representing an individual customer.
2. Company: An organisation providing products, services or account-based capabilities.
3. Company Agent: The organisation's own AI agent, which can handle customer requests.

The underlying problem is straightforward.

Today, when a personal AI agent tries to interact with a bank, airline, retailer or other business, the business has two imperfect choices:

- Block automated agents because they cannot reliably identify or control them.
- Allow agents to interact through the customer's normal login, potentially giving them excessive access.

Poppy introduces a standardised mechanism for agent discovery, identity, customer-authorised access and controlled interaction.

Its objective is to make an AI agent a recognised, permissioned participant rather than an automated browser pretending to be a human.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://personalagentprotocol.org\&sz=32)

Personal Agent Protocol+1



## 2. How Poppy works: architecture and flow

[Introducing Personal Agent Protocol | Sierra](https://images.openai.com/static-rsc-4/iGBsAR0s3AfaQj5IjhtlW_w9GCICqG5O_sTBiPkwyLoen87yMd8ZGXtxiNmYNW5Y3N1rF0bYdCHblIdtnlQrRV_Ls4bDYCgaG6rcINsILToNK7AviTp-AbiYIvgpI0jTReX4D8C39k6PYM_e7SDXuZ07EHlZXMPe1kDoJhXSOhQ?purpose=inline)

[sierra.ai](https://sierra.ai/blog/introducing-personal-agent-protocol?utm_source=chatgpt.com)

The protocol can be understood as six stages:

1\. Discovery

The company publishes a machine-readable discovery document:

`https://company.com/.well-known/poppy.json`

This describes:

- Supported interaction channels.
- API endpoints (OpenAPI and MCP).
- Company Agent endpoints.
- Authentication and sign-in capabilities.
- Other supported protocol extensions.

2\. Session establishment

The Personal Agent establishes a session representing a particular user interacting with a particular company.

A session can begin without the customer signing in. This allows anonymous activities such as checking product availability or asking about policies.

3\. Authentication and authorisation

When account access is required, the customer authenticates using standard OAuth flows.

The important distinction is that authentication and authorisation are separate:

- Authentication establishes access to the customer's account.
- Authorisation determines what the agent is allowed to do.

The company defines the available scopes and the customer approves the requested access.

4\. Session continuity

Poppy introduces a signed `Poppy-Session` JWT, identifying the Personal Agent, user and session.

This allows the company to maintain continuity across API calls, web browsing and conversations.

The user identifier is pseudonymous and company-specific, rather than a globally shared customer identifier.

5\. Execution through different channels

The company decides which interaction mechanisms it supports:

- Website: The agent browses the company's website within the session.
- APIs: The agent invokes OpenAPI or MCP endpoints.
- Company Agent: The Personal Agent communicates with the company's own agent.

6\. Handoff and human intervention

The conversation mechanism supports requesting human assistance when necessary. The customer can also participate directly in a conversation.

An important architectural detail: Poppy sessions and OAuth account access are separate. A session identifies the interaction; OAuth grants access to the account. Neither should be confused with the other.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://personalagentprotocol.org\&sz=32)

Personal Agent Protocol+1



## 3. Poppy vs MCP vs A2A — detailed comparison

| Dimension         | Poppy                                                  | MCP                                                         | A2A                                                                                  |
| ----------------- | ------------------------------------------------------ | ----------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| Primary purpose   | Customer-agent authorisation and interaction           | Agent-to-tool integration                                   | Agent-to-agent collaboration                                                         |
| Main participants | Person, personal agent, company                        | MCP host/client and server                                  | A2A client and remote agent                                                          |
| Discovery         | `poppy.json`                                           | MCP server configuration/capability negotiation             | Agent Card                                                                           |
| Core interaction  | Customer-authorised sessions and business interactions | Tools, resources and prompts                                | Messages, tasks and artifacts                                                        |
| Identity          | Personal agent and pseudonymous user identity          | Client identity and optional OAuth authorisation            | Agent identity and authentication                                                    |
| Authorisation     | Customer-approved OAuth scopes, enforced by company    | Server-defined access controls and OAuth framework for HTTP | Agent/server authentication; operation-level authorisation is implementation-defined |
| State management  | Cross-channel customer session                         | Protocol request/transport state; not a customer session    | Stateful task lifecycle                                                              |
| Typical use case  | Agent managing a customer's bank account               | Agent querying a banking API                                | Fraud agent delegating investigation to another agent                                |

[image](https://www.google.com/s2/favicons?domain=https://personalagentprotocol.org\&sz=32)

Personal Agent Protocol+3

### The most important similarities

1\. Open interoperability

All three aim to reduce bespoke integrations between AI systems and services.

2\. Discovery and capability advertisement

- Poppy: `poppy.json`
- MCP: Server capabilities and tool/resource discovery.
- A2A: Agent Cards and declared skills.

3\. Security and controlled interactions

All three have security considerations, but they address different trust boundaries. Poppy makes customer-authorised access and cross-channel session continuity central to its design.

### The most important differences

The biggest difference is the unit of interoperability:

- MCP interoperates at the capability level.
- A2A interoperates at the agent/task level.
- Poppy interoperates at the customer/business relationship level.

A2A also has a richer asynchronous task lifecycle, including status updates, streaming, artifacts and push notifications. Poppy instead focuses on a customer-facing session, account access and business interaction.

An important nuance: Poppy has its own Company Agent conversation interface. However, its current draft does not define a native A2A binding. Therefore, the two should not be treated as interchangeable conversation protocols.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1



## 4. How the three protocols work together

Consider a practical banking example.

A customer asks their personal AI agent:

"Review my recent transactions, identify suspicious payments, and raise a dispute for any fraudulent transaction."

Customer

Requests a transaction review

POPPY — Customer trust and access

Discover bank · Authenticate customer · Grant permitted scopes · Maintain session

MCP

Agent → tools

Retrieve transactions, account balances and payment information.

A2A

Agent → agent

Delegate fraud analysis to an independent fraud investigation agent.

Bank's execution and control layer

Policy checks · Customer consent · Fraud controls · Audit logging · Human approval where required

Outcome

Dispute raised within the customer's authorised permissions.

This is an illustrative architecture, not a prescribed integration mandated by Poppy.

Critical security point: Poppy does not automatically make a requested transaction safe or authorised. The bank must enforce scopes and business rules at the execution boundary. A write permission should not automatically imply permission to perform every financial operation.

## 5. Implications for enterprise AI architecture

For a regulated financial institution, I would view Poppy as an emerging external-facing trust and access layer.

| Architecture area | Potential implication                                                                   |
| ----------------- | --------------------------------------------------------------------------------------- |
| API strategy      | Publish agent-consumable OpenAPI and MCP interfaces                                     |
| Identity          | Establish agent identity and pseudonymous customer mapping                              |
| Access management | Fine-grained OAuth scopes, least privilege and revocation                               |
| Security          | JWT verification, key management, token audience validation and agent allow/block lists |
| Governance        | Consent records, transaction-level controls and auditability                            |
| Operations        | Session monitoring, abuse detection, rate limiting and human escalation                 |

Three things I would watch before adopting it:

1. Specification maturity: The current draft 0.3 can change incompatibly before 1.0.
2. Enterprise adoption: Published support is not the same as production adoption. Design partnerships do not guarantee deployed interoperability.
3. Financial transaction controls: Payment execution, transaction approvals and liability boundaries require additional controls; Poppy does not solve these automatically.&#x20;

   [image](https://www.google.com/s2/favicons?domain=https://personalagentprotocol.org\&sz=32)

   Personal Agent Protocol+2



## 6. Recommendation for senior leadership

My recommendation would be monitor, evaluate and prepare for adoption rather than immediately implement Poppy.

For an enterprise AI Innovation Lab, I would position the protocols as complementary building blocks:

- MCP: Standardise access to enterprise tools, data and services.
- A2A: Enable collaboration and delegation across independently developed agents.
- Poppy: Prepare for trusted interactions between external personal agents and the organisation.

The strategic opportunity is a future in which customers interact with their bank through their preferred personal AI agents, while the bank retains control over identity, authorisation, execution and audit.

## 7. Executive one-page slide

Created · One slide

I've created a PowerPoint slide suitable for sharing with senior leadership, including the architecture, protocol comparison, strategic implications and maturity caveat.

Poppy vs MCP vs A2A

Executive briefing · Single-slide presentation

&#x20;PowerPoint PDF

### Further reading — primary sources

1. [Poppy – Official Introduction](https://personalagentprotocol.org/docs/introduction?utm_source=chatgpt.com) and [Technical Specification (Draft 0.3)](https://personalagentprotocol.org/docs/0.3/spec?utm_source=chatgpt.com).
2. [MCP – Official Specification](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2026-07-28/basic/index.mdx?utm_source=chatgpt.com).
3. [A2A – Official Specification (1.0)](https://github.com/a2aproject/A2A/blob/main/docs/specification.md?plain=1\&utm_source=chatgpt.com).

My overall takeaway: MCP gives agents capabilities, A2A gives agents collaborators, and Poppy gives personal agents a governed way to act for customers. Together, they point towards a more interoperable agentic ecosystem.
