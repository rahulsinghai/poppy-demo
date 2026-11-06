The Demo Flow
The story writes itself, and it shows every Poppy building block in ~3 minutes:

Agent fetches poppy.json — discovery (prove it live with the community validator)

Agent opens a session and asks a guest question — sessions + JWT identity

"I need to see your orders" → company's own OAuth page opens, user approves orders:read — sign-in + scopes

Agent proposes a return, user approves the irreversible action — consent in action

Same session continues across API and company-agent chat — one session, every channel

Gotchas Worth Knowing
Spec churn: 0.2→0.3 shipped mostly breaking changes within two weeks. Pin your SDK version and expect to re-pin.

DPoP: draft 0.2+ requires companies to accept ES256 DPoP proofs — the SDKs handle this, hand-rolling it is painful.

Domain: the spec requires poppy.json on a domain you control over HTTPS — a DuckDNS subdomain works free, or spend £5 on a real domain for polish.

Oracle A1 capacity: popular regions often show "out of capacity" — pick a quieter region or retry; the instance itself is comfortably oversized for this.

Demo insurance: keep a screen recording as backup and persist all state in Postgres so a Kaggle restart resumes mid-flow.