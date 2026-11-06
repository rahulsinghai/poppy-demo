

Specification created

17-page technical document



I've created an implementation-ready specification for a fictional retail bank called Harbourline Bank, demonstrating how an independent Personal Agent interacts with a bank using Poppy.

## 1. Download the deliverables

Technical specification (DOCX)

17 pages · Editable Word document

Primary

Download the technical specification

Complete source bundle

DOCX, Markdown, architecture diagram, Graphviz source and README

Download the complete ZIP bundle

Markdown specification

Technical source for version control and future implementation

Download Markdown source

Architecture diagram

View deployment architecture

## 2. Proposed technology stack

| Layer          | Technology                  | Responsibility                                            |
| -------------- | --------------------------- | --------------------------------------------------------- |
| Bank website   | Java 21 + Spring Boot       | Banking UI, accounts, payments, statements                |
| OAuth server   | Spring Authorization Server | Customer authentication, consent and tokens               |
| Poppy gateway  | Rust + Axum                 | Session JWT verification, delegated keys, API security    |
| Personal Agent | Python + FastAPI            | Discovery, OAuth client, Poppy sessions and orchestration |
| Database       | OCI MySQL HeatWave          | Bank ledger, customer, account and agent state            |
| Hosting        | OCI Ampere A1 VM            | Docker Compose deployment                                 |
| Reverse proxy  | Caddy                       | HTTPS and hostname routing                                |

## 3. What the demo will actually demonstrate

Phase 1

Bank website and synthetic data

A familiar retail banking experience: login, dashboard, current and savings accounts, transaction history, statements, cards and beneficiaries.

Phase 2

Poppy discovery and authentication

The Personal Agent discovers the bank through `poppy.json`, verifies issuer metadata, publishes its client metadata and JWKS, and connects using Direct Sign-In and OAuth consent.

Phase 3

Signed Sessions and banking APIs

The agent generates delegated keys and request-bound `Poppy-Session` JWTs, then accesses accounts and transactions with authorised scopes.

Phase 4

Agent conversations and money movement

Company Agent conversations, simulated £45 transfers, explicit customer approval using Operations v2, balanced ledger entries and duplicate-request protection.

Phase 5–6

Browser joining and MCP interoperability

Demonstrate a shared Session across API and browser interactions, then optionally add a read-only MCP server.

## 4. Two important architectural decisions

Hosting: OCI Always Free provides an Ampere A1 allowance equivalent to 2 OCPUs and 12 GB RAM, plus a standalone MySQL HeatWave Always Free database with 50 GB storage and 50 GB additional backup storage. Availability and capacity depend on your home region and tenancy.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.oracle.com\&sz=32)

Oracle Docs+1



Protocol version: The specification targets Poppy draft 0.4, checked on 6 November 2026. It remains a draft and may change incompatibly. The design pins the version and includes protocol conformance tests.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://personalagentprotocol.org\&sz=32)

Personal Agent Protocol+1



All banking information, customers, cards and transactions are synthetic.

Recommended next step: Build the runnable Git monorepo scaffold from this specification, starting with Docker Compose, MySQL migrations, the Rust Poppy gateway, Java bank services and Python Personal Agent.