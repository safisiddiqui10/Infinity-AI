# Infinity AI — Small Steps Implementation Plan

**Purpose:** Turn the long-term architecture into small, reviewable tasks.  
**Last updated:** 2026-10-22  
**Long-term decisions:** [`LONGTERM_PLAN.md`](LONGTERM_PLAN.md)  
**Original roadmap:** [`PROJECT_PLAN.md`](PROJECT_PLAN.md)

---

## How we will work

For each task:

1. **Inspect** the relevant files and current behavior.
2. **Discuss** the problem, options, trade-offs, and expected result.
3. **Agree** on the proposed change before editing code.
4. **Implement** the smallest useful change.
5. **Test** the changed behavior and nearby regressions.
6. **Review** the result and update this checklist.

**Initial rule:** The first phase is read-only. Do not modify code during the audit. Later edits should remain small and phase-by-phase; do not rewrite the full project just to fit the target architecture.

Use checkboxes to track progress. A task is complete only after its “Done when” condition passes. If the audit reveals that a task is already complete, verify it and record the evidence instead of rebuilding it.

---

## Phase 0 — Agree on scope and current decisions

- [ ] **0.1 Read the plans together.** Confirm `PROJECT_PLAN.md`, `LONGTERM_PLAN.md`, and this file are consistent.
  - Done when: any contradictions or missing requirements are listed.
- [ ] **0.2 Confirm the first release's minimum user journeys.** Start with chat, persistent threads, document questions, and existing tools; define what must work before more features are added.
  - Done when: the first-release user journeys are written down.
- [ ] **0.3 Create an initial test-question set.** Write 30–50 representative chat, multi-turn, tool, document, and failure scenarios; include expected behavior rather than requiring a single exact response where multiple answers may be correct.
  - Done when: every scenario has a category and a way to judge the result.
- [ ] **0.4 Record the open decisions.** Keep unresolved hosting, storage, billing, vector database, and team-workspace decisions visible.
  - Done when: none of these are treated as silently finalized.

## Phase 1 — Read-only repository audit

**No code changes in this phase.**

- [ ] **1.1 Map the repository.** List the entry points, folders, important modules, configuration files, dependency files, and tests.
  - Done when: the purpose of each key file is known.
- [ ] **1.2 Trace one normal chat request.** Follow the request from the Streamlit interface through LangGraph and Groq to the displayed response.
  - Done when: the call flow and state transitions are documented.
- [ ] **1.3 Trace thread persistence.** Identify how thread IDs are created, stored, loaded, and resumed; inspect the checkpoint mechanism and application history store.
  - Done when: we know which component owns each piece of chat state.
- [ ] **1.4 Trace the existing PDF RAG pipeline.** Follow upload, extraction, chunking, embeddings, FAISS indexing, retrieval, and answer generation.
  - Done when: the complete document-question flow is mapped.
- [ ] **1.5 Inspect the tool registry.** Review Search, Stocks, Calculator, and any other actual tool code; record errors, timeouts, and permission assumptions.
  - Done when: each tool's inputs, outputs, dependencies, and side effects are known.
- [ ] **1.6 Inspect persistence and configuration.** Identify actual database/storage use, environment variables, secrets handling, and how local configuration is loaded.
  - Done when: no assumptions remain about the current database or configuration path.
- [ ] **1.7 Inspect dependencies and test coverage.** Review requirements/dependency files, version constraints, available tests, and run instructions without upgrading packages.
  - Done when: the current environment can be described and known test gaps are listed.
- [ ] **1.8 Write a current-state architecture map.** Record what works, what is fragile, and what should not be changed without cause.
  - Done when: the audit findings are reviewed before implementation begins.

## Phase 2 — Establish a safe baseline

- [ ] **2.1 Write down how to run the existing app locally.** Record current commands and necessary environment variables without exposing secrets.
  - Done when: the app's normal startup sequence is reproducible.
- [ ] **2.2 Run the current app and core workflows.** Test a normal chat, a second message in the same thread, a new thread, each existing tool, and a PDF question.
  - Done when: results and failures are recorded.
- [ ] **2.3 Record known bugs.** Separate blockers, correctness problems, usability gaps, and optional improvements.
  - Done when: issues are prioritized and reproducible.
- [ ] **2.4 Add or repair one small test at a time.** Start with the highest-risk core behavior; do not refactor unrelated code merely to make tests convenient.
  - Done when: at least the key existing workflows have repeatable checks.
- [ ] **2.5 Add controlled errors to the highest-risk path.** Only after tracing it, handle one concrete failure mode (for example, provider failure or invalid tool input) with a clear error and test.
  - Done when: the failure no longer crashes the user workflow unexpectedly.

## Phase 3 — Stabilize the core and create useful boundaries

- [ ] **3.1 Define a chat use case.** Identify the minimal service function that sends a message without depending on Streamlit UI objects.
  - Done when: its inputs and outputs are documented and tested.
- [ ] **3.2 Move one piece of UI-independent logic.** Extract only the chosen logic; retain the existing UI behavior.
  - Done when: existing chat tests still pass.
- [ ] **3.3 Centralize runtime configuration.** Decide how the LLM provider, model name, storage paths, and app settings are loaded.
  - Done when: secrets are not hardcoded and configuration failures are understandable.
- [ ] **3.4 Bound tool execution.** Review one tool at a time for timeout, input validation, output shape, and retry needs.
  - Done when: no tool can retry indefinitely or run without a defined limit.
- [ ] **3.5 Bound agent loops.** Set and test a maximum number of graph/tool steps where applicable.
  - Done when: a looping agent run terminates safely.
- [ ] **3.6 Document checkpoint versus transcript behavior.** Decide how user-visible messages relate to LangGraph checkpoint state and how failures/retries are represented.
  - Done when: duplicate sends and partial failures have an explicit handling strategy.

## Phase 4 — Clerk proof of concept and identity design

**Do the smallest proof of concept before a broad integration.**

- [ ] **4.1 Review Clerk's current React and Python setup guides.** Confirm the current SDKs and required configuration before installing anything.
  - Done when: the intended SDK versions and setup commands are recorded.
- [ ] **4.2 Define development configuration.** Identify frontend publishable-key configuration, backend-only secret configuration, allowed origins, and local URLs.
  - Done when: no secret is required in a browser bundle or source file.
- [ ] **4.3 Test a minimal sign-in flow in an isolated proof of concept.** Do not replace Streamlit or refactor the full application for this task.
  - Done when: a test user can sign in locally using Clerk's documented flow.
- [ ] **4.4 Test FastAPI token verification.** Add a minimal protected test endpoint in the proof of concept and verify valid, invalid, missing, and expired tokens where testable.
  - Done when: unauthorized requests are rejected and valid requests produce trusted identity data.
- [ ] **4.5 Define internal identity mapping.** Finalize `users` and `auth_identities` keys and the uniqueness rule for `(provider, provider_subject)`.
  - Done when: the application can keep its own stable user ID without using email as a primary key.
- [ ] **4.6 Review the proof of concept before integrating.** Record usability, local workflow, provider lock-in, cost assumptions, and any gaps.
  - Done when: the integration design is explicitly approved.

## Phase 5 — Database model and SaaS authorization foundations

- [ ] **5.1 Audit the current SQLite schema and thread metadata.** Identify what must be retained or migrated.
  - Done when: the proposed schema accounts for existing data and IDs.
- [ ] **5.2 Finalize the minimum schema.** Start with users, auth identities, workspaces, memberships, conversations, and messages. Add other tables when their use cases are clear.
  - Done when: keys, foreign keys, uniqueness rules, and deletion semantics are written down.
- [ ] **5.3 Define private-workspace creation.** Specify how the first workspace is created and how retries avoid duplicates.
  - Done when: account creation is idempotent and ownership is unambiguous.
- [ ] **5.4 Define centralized authorization checks.** Write one documented rule for checking membership and resource scope, then apply it to one endpoint at a time.
  - Done when: protected operations do not rely on client-provided IDs as proof of access.
- [ ] **5.5 Choose a local PostgreSQL setup method.** Keep it simple; document local installation/configuration and migration commands. Do not introduce Docker unless the project owner explicitly changes the current no-Docker constraint.
  - Done when: the database can be run and reset reproducibly in the approved local environment.
- [ ] **5.6 Add versioned schema migrations.** Choose the migration approach after examining existing dependencies and schema.
  - Done when: a clean database can be created from migrations and changes can be applied predictably.
- [ ] **5.7 Decide whether to use PostgreSQL RLS.** Evaluate how request/workspace context is established, which database role the app uses, and how connection pooling/reset behavior works.
  - Done when: the decision and corresponding tests are documented; do not assume RLS is configured just because it is discussed.
- [ ] **5.8 Add cross-user authorization tests.** Test conversations and every currently supported private resource with two separate users/workspaces.
  - Done when: one user's ID manipulation cannot expose another user's private data.

## Phase 6 — FastAPI application boundary

- [ ] **6.1 Inventory existing request/response logic.** Determine which current operations are already suitable for an API.
  - Done when: we avoid building duplicate business logic.
- [ ] **6.2 Define the first API contract.** Start with health/status and one chat operation; agree request, response, and error schemas.
  - Done when: contracts are documented before frontend integration.
- [ ] **6.3 Add FastAPI without removing Streamlit.** Keep both interfaces able to call shared application services as appropriate.
  - Done when: the API can be tested independently and the existing app still works.
- [ ] **6.4 Add authentication dependency.** Integrate the approved Clerk token-validation flow with FastAPI.
  - Done when: protected endpoints require verified identity.
- [ ] **6.5 Add authorization to the first protected resource.** Use conversations as the first end-to-end example, then repeat the tested pattern for other resources.
  - Done when: read/update/delete operations enforce ownership and workspace policies.
- [ ] **6.6 Validate API input and output.** Add size/type checks and safe error responses for the first endpoints.
  - Done when: malformed input is rejected without exposing secrets or internal traces.

## Phase 7 — Conversation persistence and thread isolation

- [ ] **7.1 Map conversations to server-managed LangGraph thread IDs.** Keep thread IDs internal or treat them only as identifiers, never as permissions.
  - Done when: the backend authorizes a conversation before resolving/running its graph thread.
- [ ] **7.2 Define message/checkpoint consistency behavior.** Address retries, duplicate submissions, and partial failures.
  - Done when: failures result in a known, testable state rather than confusing duplicate messages.
- [ ] **7.3 Test thread isolation across two users.** Attempt to read and resume another user's thread.
  - Done when: access is denied and private state is not returned.
- [ ] **7.4 Define conversation deletion.** Include product-facing messages and related checkpoint state where applicable.
  - Done when: deletion behavior and any retention exceptions are documented and tested.

## Phase 8 — Documents and RAG, one capability at a time

- [ ] **8.1 Add ingestion tests for the existing PDF flow.** Cover extraction, chunking, metadata, and indexing.
  - Done when: a known document produces expected searchable content.
- [ ] **8.2 Add source-aware answers.** Define how retrieved passages and citations appear in responses.
  - Done when: responses can be traced to retrieved source passages when appropriate.
- [ ] **8.3 Define a document lifecycle and status.** Use states such as uploaded, queued, processing, ready, and failed where supported.
  - Done when: users can distinguish indexing success from failure.
- [ ] **8.4 Implement safe document deletion.** Handle original file, chunks, vector entries, and indexing jobs.
  - Done when: deleted content no longer appears in retrieval and cleanup outcomes are visible.
- [ ] **8.5 Add TXT support.** Implement and test only this format first.
  - Done when: a TXT file can be uploaded, indexed, queried, and deleted.
- [ ] **8.6 Add DOCX support.** Reuse shared ingestion boundaries where suitable.
  - Done when: DOCX workflow tests pass.
- [ ] **8.7 Add CSV support.** Decide how tabular rows, headers, and large files are represented before implementing.
  - Done when: the chosen behavior is documented and tested.
- [ ] **8.8 Add a document manager.** Show the current user's permitted files, indexing state, and delete action.
  - Done when: the UI uses authorized backend endpoints and handles failed deletion clearly.
- [ ] **8.9 Measure baseline retrieval quality.** Run the document evaluation set and save results.
  - Done when: baseline retrieval/answer behavior is recorded.
- [ ] **8.10 Test hybrid search.** Compare keyword-plus-semantic retrieval to the saved baseline.
  - Done when: we keep it only if the results justify the complexity.
- [ ] **8.11 Test reranking.** Compare with the current best retrieval configuration.
  - Done when: quality and latency trade-offs are recorded.
- [ ] **8.12 Consider query expansion.** Add only for query types where tests show a measurable benefit.
  - Done when: the change passes the same evaluation and does not introduce unacceptable latency or irrelevant results.
- [ ] **8.13 Evaluate hosted vector storage.** Compare pgvector and dedicated vector options using expected workload, cost, operations, and tenant filtering.
  - Done when: there is an evidence-based choice before migration.

## Phase 9 — Long-term memory

- [ ] **9.1 Separate memory requirements from chat history.** List which facts/preferences are useful to save and what must not be saved automatically.
  - Done when: memory categories and save rules are documented.
- [ ] **9.2 Define memory ownership.** Distinguish user-private and workspace-scoped memories.
  - Done when: each memory has exactly one intended scope.
- [ ] **9.3 Build the minimal memory store/service.** Keep it separate from conversation checkpoints and document chunks.
  - Done when: memories can be stored and retrieved under an explicit scope.
- [ ] **9.4 Test cross-thread recall.** Verify a permitted user preference can be used in another conversation by the same user.
  - Done when: relevant memory is retrieved and unrelated users' memories are never returned.
- [ ] **9.5 Add inspect/edit/delete controls.** Provide understandable behavior for users.
  - Done when: deletion is respected by the retrieval path and any derived storage cleanup is defined.
- [ ] **9.6 Add expiration/source details where useful.** Avoid keeping stale or untraceable facts indefinitely.
  - Done when: the policy is documented and tested.

## Phase 10 — UI and multimodal product experience

- [ ] **10.1 Design the React application shell.** Decide navigation, chat layout, responsive behavior, and component conventions.
  - Done when: a small UI prototype is reviewed before broad implementation.
- [ ] **10.2 Add Clerk React authentication UI.** Integrate signup, sign-in, sign-out, and profile controls through the supported SDK.
  - Done when: the React app signs in locally and protected requests reach FastAPI.
- [ ] **10.3 Build the normal chat workflow.** Connect the UI to the approved chat API and handle loading, errors, retries, and streaming if supported.
  - Done when: core chat scenarios work end to end.
- [ ] **10.4 Add Markdown and code rendering.** Include safe rendering behavior and syntax highlighting where appropriate.
  - Done when: common Markdown and code examples render correctly.
- [ ] **10.5 Add thread navigation and management.** Include only the agreed create, rename, resume, and delete operations.
  - Done when: list and detail APIs enforce workspace authorization.
- [ ] **10.6 Define Coding Mode versus Normal Mode.** Specify differences in prompts, tool availability, response style, and safety rules.
  - Done when: mode differences are documented and covered by test cases.
- [ ] **10.7 Add the two modes in the UI/backend.** Keep mode selection explicit and validated on the backend.
  - Done when: each mode invokes the intended behavior.
- [ ] **10.8 Define the first multimodal capability.** Pick one supported input/output type based on actual need and provider/model capability.
  - Done when: capability, limits, privacy, and tests are agreed before implementation.
- [ ] **10.9 Implement and test one multimodal path.** Avoid adding several media types at once.
  - Done when: supported and unsupported file/response cases are handled clearly.

## Phase 11 — Evaluation, usage limits, and observability

- [ ] **11.1 Add structured logs.** Include request/run IDs and safe status metadata; avoid secrets and unnecessary content.
  - Done when: a failure can be traced across API, service, and agent layers.
- [ ] **11.2 Add token/usage measurement.** Track usage by user/workspace where provider data allows it.
  - Done when: usage can be summarized without exposing private prompts.
- [ ] **11.3 Add request and tool timeouts.** Start with the slowest or most failure-prone operations identified in the audit.
  - Done when: requests terminate predictably when a dependency stalls.
- [ ] **11.4 Add rate and size limits.** Set initial development values, then tune based on real workload before launch.
  - Done when: oversized or excessive requests are rejected clearly.
- [ ] **11.5 Automate the evaluation set.** Run the saved cases after relevant changes.
  - Done when: regressions can be compared against a known baseline.
- [ ] **11.6 Decide on tracing tooling.** Evaluate LangSmith or an alternative; keep local logs/tests useful without a paid trace service.
  - Done when: chosen observability tools and limits are documented.

## Phase 12 — Integrations and autonomous agents

Do not start this phase until the core application, permission model, and tests are reliable.

- [ ] **12.1 Choose one external integration.** Compare need, permissions, API access, maintenance, and privacy; do not build Slack, Discord, Email, Notion, and Drive all at once.
  - Done when: one integration is selected with a narrowly defined use case.
- [ ] **12.2 Design and store integration credentials safely.** Define account/workspace ownership, revocation, and secret handling.
  - Done when: credentials are never exposed to the frontend or logs.
- [ ] **12.3 Implement one read-only integration workflow first.** Add write actions only when required and authorized.
  - Done when: failures, retries, and permission boundaries are tested.
- [ ] **12.4 Design a bounded plan-and-execute workflow.** Define max steps, deadlines, tool allowlists, checkpoints, cancellation, and approval rules.
  - Done when: the plan cannot run indefinitely or perform consequential actions without the required authorization.
- [ ] **12.5 Test agent recovery and cancellation.** Cover tool failure, timeout, repeated action, and interrupted execution.
  - Done when: each case produces a known end state.

## Phase 13 — Hosting and launch readiness

- [ ] **13.1 Choose hosting after estimating workload.** Compare operational simplicity, cost, supported Python/React deployment, region, and provider limits.
  - Done when: a deployment design and cost assumptions are written down.
- [ ] **13.2 Prepare production environment variables and secrets.** Separate development and production credentials.
  - Done when: production secrets are not present in the repository or frontend bundle.
- [ ] **13.3 Prepare database migration and backup procedures.** Verify restore, not just backup creation.
  - Done when: a test database can be restored using documented steps.
- [ ] **13.4 Configure HTTPS, allowed origins, health checks, and error monitoring.**
  - Done when: deployment and basic recovery behavior are tested.
- [ ] **13.5 Run a tenant-isolation security suite.** Test chat, file download/delete, RAG, memory, checkpoints, and tools with multiple test users.
  - Done when: no cross-tenant access test succeeds improperly.
- [ ] **13.6 Document data deletion and retention.** Cover account deletion, documents, vectors, messages, checkpoints, and memories.
  - Done when: expected behavior and exceptions are documented and tested.
- [ ] **13.7 Run a small private beta.** Use limited users/usage and collect concrete failures before public launch.
  - Done when: high-severity issues are resolved or launch is paused.
- [ ] **13.8 Complete a launch review.** Review reliability, security, privacy, cost, support, and backup readiness.
  - Done when: the team makes an explicit go/no-go decision.

## First three tasks to do next

1. **Task 1:** Read-only repository map and entry-point inventory.
2. **Task 2:** Trace one chat request from Streamlit through LangGraph/Groq to the final response.
3. **Task 3:** Trace thread persistence and checkpoint behavior.

Do not integrate Clerk, migrate databases, or rewrite the UI before these tasks establish the actual current architecture.

---

## Progress notes

| Date | Task/phase | Findings or decision | Next step |
|---|---|---|---|
| 2026-10-22 | Architecture brainstorming | Public multi-user SaaS; local development then hosted release; personal workspaces first; Clerk selected for v1 candidate; independent application database; React later, retain Streamlit during initial audit; PostgreSQL target; modular monolith | Start read-only repository audit |
