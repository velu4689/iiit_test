# DataHub Context Engineering — Technical Deep Dive Question Sheet

*Prepared for enterprise architecture review. Grounded in DataHub Cloud's current public positioning (Context Platform, MCP Server, Agent Context Kit) as of Aug 2026 — validate all specifics live with the specialists.*

---

## 1. Context Platform — Current State & Known Gaps

DataHub markets itself as the platform for the "context layer" between agents and enterprise data (ingestion, semantic extraction from query history, expert validation). Push on where this is still maturing:

- What percentage of your customer base has moved from the open-source catalog to the full Context Platform (Context Ingestion, Context Intelligence, Context Hub, MCP delivery) vs. still using DataHub as a traditional catalog?
- Context Intelligence claims to auto-generate context documents from query logs, BI dashboards, and dbt projects — what's your measured precision/recall on auto-generated business definitions, and how often do humans need to correct them?
- What's the actual latency from a schema/metadata change to that change being reflected in context served to an agent? Is this truly real-time or batch/near-real-time?
- Where does context freshness break down today — e.g., unstructured sources (Confluence/Notion), semi-structured BI layers, or streaming sources?
- What are the top 3 unresolved product gaps your own roadmap is targeting in the next 2–3 releases?
- How do you handle conflicting context (e.g., two teams with different definitions of "revenue") — is conflict resolution automated, human-arbitrated, or left to the consuming agent?
- What is your track record / benchmark for reducing hallucination or incorrect SQL generation when agents use DataHub context vs. no context layer — can you share the methodology, not just the headline number?
- Are there any customer-reported production incidents (bad context served, stale lineage, incorrect access decisions) you can speak to, and what changed as a result?
- What's the current maximum scale (metadata graph size, entities, QPS) you've proven in production, and where does performance degrade?

## 2. Agents Directly Interacting with DataHub

- Beyond the MCP Server, what direct SDK/API surface exists for a custom agent to query, write back to, or subscribe to changes in the metadata graph (the "Agent Context Kit")?
- Can agents **write** to DataHub (e.g., log usage feedback, propose corrections, create new glossary terms) or is agent access strictly read-only today?
- How is agent identity distinguished from human identity in your access model — do agents get service accounts, scoped tokens, or a separate principal type?
- Is there a rate-limiting / cost-control mechanism to prevent a runaway agent from hammering the context graph?
- How do you support long-running or stateful agent sessions (memory across turns) vs. stateless single-query retrieval?
- What does an agent do when requested context doesn't exist or is stale — is there a defined "context not found" / fallback contract, or does it silently degrade?
- Do you support agent-to-agent context sharing (e.g., one agent's derived context becoming trusted context for another), and how is provenance tracked in that chain?

## 3. MCP Server Support

- Is the DataHub MCP Server open-source or Cloud-only? What tools/resources does it expose today (search, lineage, glossary, ownership, query history, etc.)?
- What authentication/authorization model does the MCP server enforce per-call — does it inherit the calling user's DataHub permissions, or does it run under a shared service identity?
- How do you scope MCP tool access down to row/column-level or domain-level permissions, not just "can call this tool"?
- Is the MCP server stateless per request, and how do you handle multi-turn context within an MCP session?
- What's your position on the MCP protocol's evolution (versioning, breaking changes) — how do you manage backward compatibility for enterprise integrators?
- Can we self-host the MCP server in our VPC, or does it require a call out to DataHub Cloud?
- What audit logging exists specifically for MCP tool calls — can we see which agent/user asked for what context and when, for compliance purposes?
- Have you benchmarked MCP server latency/throughput under concurrent multi-agent load?
- Which agent frameworks/runtimes have you validated against in production (LangChain, LlamaIndex, Databricks Genie, Snowflake Intelligence, custom agents, Claude, etc.)?

## 4. Identity, Okta & SSO/Access Control

- Confirm current state: DataHub supports OIDC-based SSO natively. Is SAML now supported, or still OIDC-only? (Public docs previously stated SAML/LDAP were not yet supported — has this changed?)
- For Okta specifically: is SCIM provisioning fully automated end-to-end today, or does it still require the SWA-app workaround alongside a separate OIDC app integration?
- How are Okta group memberships mapped to DataHub policies/roles — real-time sync or periodic reconciliation?
- How does authorization propagate to **agents** acting on a user's behalf — does an agent inherit the requesting user's Okta-derived entitlements, or does it use a separate service-account policy?
- Do you support attribute-based access control (ABAC) tied to Okta claims (e.g., department, clearance level) for row/column-level metadata restrictions?
- What happens to an agent's active session/context access when a user's Okta access is revoked mid-session — is revocation propagated immediately?
- Is there support for Okta Fine Grained Authorization / Okta's newer identity-for-AI-agents offerings, or non-human identity management generally?
- How do you handle break-glass/emergency access and audit trails for both human admins and agent service accounts?

## 5. Broader Architecture, Security & Operations

- Deployment model: is Context Platform available for in-VPC/private deployment, or is any component (embedding, semantic extraction) processed in DataHub's multi-tenant cloud?
- Where is embedding/vectorization done, what models are used, and is our proprietary metadata/data ever used to train shared models across tenants?
- What's the data residency story — can context ingestion, embedding, and serving all stay within a specific region/cloud for regulatory reasons (GDPR, data sovereignty)?
- SOC 2 / HIPAA compliance is claimed — is there a current SOC 2 Type II report we can review, and does it cover the newer Context Platform components specifically or just the legacy catalog?
- What's the actual TCO difference between self-hosted OSS DataHub with community MCP support vs. DataHub Cloud Context Platform, at our scale (data volume, users, agents)?
- What's your DR/HA SLA for the metadata graph itself — if DataHub is down, do dependent agents fail closed (no answer) or fail open (stale/no context silently)?
- How do version upgrades/migrations work for self-hosted deployments that have heavily customized the metadata model — what's the typical upgrade pain point customers hit?
- Can you share 2–3 reference customers running agentic use cases at production scale (not pilot) who'd be willing to talk about what broke and what worked?

## 6. Industry-Wide Challenges, Unstructured Data & Context Federation

Broader questions to test whether DataHub is honest about category-wide limitations, not just their own product gaps:

**Industry challenges (use to gauge how self-aware they are):**
- What do you see as the biggest unsolved problems for cataloging/context platforms as a category right now — not just what DataHub has solved?
- How do you prevent AI-generated context (auto-extracted definitions, inferred lineage) from creating a feedback loop of confidently wrong "trusted" context?
- Your own research shows a gap between claimed maturity and production impact (88% claim operational context platforms, yet 61% still delay AI initiatives for lack of trusted data) — what's actually driving that gap for your customers?
- How do you handle semantic conflicts between teams (different definitions of the same business term) at scale — is this ever fully automated, or always human-arbitrated?
- What's your approach to keeping access control granular and current for autonomous/long-running agent sessions, as opposed to human sessions?

**Unstructured data:**
- Beyond text-based sources like Confluence/Notion (which you chunk and embed for semantic search), what's your support for binary unstructured content — PDFs, images, scanned documents, audio, video?
- Is binary/multimodal ingestion on the roadmap, or do we need to pre-process (OCR, transcription) before ingestion?
- For text sources you do support, how do you handle document versioning and staleness — if a Confluence page changes, how fast does the embedded/chunked version in DataHub catch up?
- How do you evaluate/measure semantic search quality on unstructured content, and can we see benchmarks?

**Context federation:**
- Is DataHub's model live query federation across other catalogs, or full metadata ingestion/duplication into DataHub's own graph? Confirm which.
- If we already run another catalog (Collibra, Purview, Unity Catalog, Alation) in parts of the org, does DataHub federate with it live, or require full ingestion?
- If it's ingestion-based, how do you handle two-way sync — if a definition is edited in DataHub, does it flow back to the source system, or does DataHub become the new system of record?
- For multi-region/multi-cloud enterprises, can the context graph itself be federated (e.g., regional instances federated into one logical layer), or is it a single centralized deployment?
- How do you handle the "system of record" question when metadata about the same asset exists in multiple places (e.g., a glossary term defined in both Collibra and DataHub)?

---

**Suggested use in the call:** Lead with Section 1 (drawbacks) to set an honest tone before the vendor goes into feature pitch mode, then move to Sections 2–4 as your core evaluation criteria, use Section 5 to pressure-test operational readiness for your environment, and close with Section 6 to see how candidly they discuss category-wide limits versus their own product's specific gaps.
