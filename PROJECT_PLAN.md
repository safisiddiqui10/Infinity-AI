# Infinity AI — Project Plan

> **Planning status:** Updated after the initial architecture brainstorm on 22 October 2026. This is a working plan, not a claim that the proposed work has been implemented. We will continue brainstorming and refine the scope before starting each implementation phase.

## 1. Vision

Evolve Infinity AI from a functional prototype into a reliable, extensible AI assistant that can later become a public, multi-user SaaS product.

The product direction is to support two distinct experiences:

- **Normal Mode:** general questions, research, document Q&A, and suitable tools.
- **Coding Mode:** code explanation, review, debugging, generation, and—once appropriate permissions and safeguards exist—project-aware coding workflows.

The project will be developed locally first. We will improve the existing chatbot before replacing the current UI or introducing the new API architecture. Each phase should leave behind tested, reusable components rather than a large rewrite.

## 2. Decisions Agreed So Far

- **Development approach:** local-first, with a hosted product planned for later.
- **Eventual audience:** public, multi-user SaaS.
- **Workspace model:** personal workspace first; design the membership model so team workspaces can be added later.
- **Authentication direction:** managed authentication, with **Clerk selected as the v1 candidate** because developer experience and polished React authentication are priorities. Validate its integration before committing it throughout the codebase.
- **Application data:** Infinity AI should own its application data model and authorization rules. The identity provider must not become the source of truth for chat ownership or workspace permissions.
- **Current application:** preserve working Streamlit, LangGraph, Groq, FAISS, and persistence functionality where it is sound. Audit before refactoring.
- **Implementation style:** brainstorm and agree on a phase first; inspect existing code; make small changes; test; then continue.
- **No premature infrastructure:** begin with a modular application, not microservices or Kubernetes. Introduce infrastructure when the product's real needs justify it.

## 3. Current Prototype — Existing Foundation

The original project plan records the following capabilities as completed. Their current behavior will be verified during the audit rather than assumed to be bug-free.

- [x] LangGraph-based state management.
- [x] LLM integration through Groq.
- [x] Basic tool calling.
- [x] Per-thread PDF RAG using FAISS.
- [x] Streamlit frontend and thread persistence.
- [x] Basic Search, Stocks, and Calculator tools.

---

# Phase 1 — Improve the Existing Chatbot

**Goal:** Make the current application more reliable, testable, trustworthy, and easier to extend before building the React UI and FastAPI backend.

**Phase 1 rule:** Do not begin with a large rewrite. Start with a read-only audit and baseline tests. Implement only the upgrades we agree are useful after examining the existing code.

## Phase 1A — Read-Only Repository Audit and Baseline

- [ ] Map the current request flow: Streamlit → LangGraph → LLM/tools/RAG → persistence → response.
- [ ] Inspect the current project structure, configuration, dependencies, and run instructions.
- [ ] Identify how threads, checkpoints, messages, uploaded files, and FAISS indexes are stored.
- [ ] Review existing prompts, graph nodes, tool schemas, and error handling.
- [ ] Reproduce and document known issues before changing code.
- [ ] Create a small evaluation set of approximately 30–50 realistic questions and tasks.
- [ ] Record baseline results for answer quality, RAG retrieval, tool success, errors, latency, and token/usage data when available.
- [ ] Produce an audit report with findings grouped into: keep, fix, investigate, and defer.

**Exit condition:** We understand the current code paths and have repeatable baseline tests. No code changes are required during the initial audit.

## Phase 1B — Agent and Tool Reliability

- [ ] Review and improve tool descriptions and input schemas.
- [ ] Validate tool arguments and handle invalid input and unavailable services gracefully.
- [ ] Add sensible timeouts, bounded retries, and a maximum number of agent/tool steps.
- [ ] Avoid repeated tool calls or unbounded agent loops.
- [ ] Log selected tools, execution duration, success/failure, and safe diagnostic metadata.
- [ ] Add success and failure tests for each existing tool.
- [ ] Ensure tool failures return understandable messages instead of crashing the conversation.

**Exit condition:** Each existing tool has tests for normal and failure cases, and agent runs terminate predictably.

## Phase 1C — RAG Quality and Document Management

- [ ] Test PDF extraction and identify missing, garbled, or duplicated text.
- [ ] Review chunk size, overlap, metadata, and retrieval configuration against evaluation examples.
- [ ] Preserve source metadata such as document name and page number where available.
- [ ] Show source citations or retrieved passages when answering document questions.
- [ ] Prevent duplicate indexing when the same file is uploaded again.
- [ ] Define reliable delete and re-index behavior for document files, chunks, and FAISS indexes.
- [ ] Add visible document-processing status and useful error messages.
- [ ] Add a simple document manager to list and delete indexed files if the audit confirms it fits the current design.
- [ ] After measuring baseline retrieval, evaluate TXT, DOCX, and CSV ingestion.
- [ ] Only then compare hybrid keyword + semantic search, reranking, and query expansion one change at a time.

**Exit condition:** We can reproduce retrieval tests, inspect the sources used by an answer, and correctly manage document indexing and deletion.

## Phase 1D — Conversation and Persistence Reliability

- [ ] Test that separate threads remain isolated.
- [ ] Verify that a previous conversation resumes with the expected state.
- [ ] Test new-chat, reset, and restart behavior.
- [ ] Investigate duplicate messages, partial writes, or inconsistencies between conversation history and LangGraph checkpoints.
- [ ] Define how failed runs and retries should be represented.
- [ ] Ensure one thread's state cannot leak into another thread's prompt or response.

**Exit condition:** Thread behavior is predictable across normal interactions, errors, and application restarts.

## Phase 1E — Observability and Current UI Improvements

- [ ] Add useful structured local logs for errors, model calls, tool execution, and request duration.
- [ ] Keep LangSmith optional; maintain a useful local debugging path.
- [ ] Avoid logging secrets, raw credentials, or unnecessary sensitive user content.
- [ ] Improve the existing Streamlit UI's Markdown and code rendering where needed.
- [ ] Add useful user-facing loading, empty, error, and document-processing states.
- [ ] Consider lightweight thumbs-up/thumbs-down feedback to help identify poor answers.
- [ ] Track quality and reliability results against the Phase 1 baseline.

**Exit condition:** Common failures are diagnosable, users receive understandable status messages, and regression tests remain green.

## Phase 1F — Prepare for Phase 2 Without Premature Rewrites

- [ ] Identify core logic currently coupled to Streamlit session state or UI rendering.
- [ ] Separate reusable chat, RAG, tool, and persistence logic only where necessary and safe.
- [ ] Establish clear service boundaries that can later be called by FastAPI.
- [ ] Centralize configuration and secret handling.
- [ ] Define acceptance criteria for Normal Mode and Coding Mode.
- [ ] Document the future API needs for chat, threads, documents, feedback, and mode selection.
- [ ] Document desired MCP use cases before choosing whether Infinity AI will act as an MCP client, expose MCP servers, or both.

**Exit condition:** We have a stable prototype and a documented plan for exposing core functionality through an API. Avoid abstract layers that do not solve a current or clearly anticipated problem.

### Phase 1 Priorities

1. Audit and baseline tests.
2. Agent and tool reliability.
3. Conversation and persistence reliability.
4. RAG quality, citations, and document lifecycle.
5. Logging and current UI improvements.
6. Reusable service boundaries and Phase 2 design.

The detailed order may change after the repository audit. Do not implement every proposed feature just because it appears here; use tests and actual findings to decide.

---

# Phase 2 — New Product Architecture and Dual-Mode Experience

**Goal:** Introduce a React frontend, FastAPI backend, explicit Normal/Coding modes, and carefully designed MCP support while reusing proven Phase 1 functionality.

## Phase 2A — Confirm Architecture and API Contracts

- [ ] Review the Phase 1 audit and decide what core modules should be reused or refactored.
- [ ] Define request/response schemas and API error conventions.
- [ ] Define thread, message, mode, document, and feedback contracts.
- [ ] Choose the MCP implementation after deciding whether Infinity AI needs an MCP client, MCP server, or both.
- [ ] Keep regular REST API responsibilities separate from MCP protocol responsibilities.
- [ ] Choose and pin compatible versions of the selected FastAPI and MCP packages.

## Phase 2B — FastAPI Backend

- [ ] Add a FastAPI application around reusable application services.
- [ ] Expose endpoints for health checks, chat, threads, document operations, and mode selection as required.
- [ ] Validate incoming data and return consistent errors.
- [ ] Add request logging, timeouts, and tests for the API.
- [ ] Keep LLM, RAG, and tool logic out of route handlers where it belongs in application services.
- [ ] Define the authentication dependency boundary; integrate Clerk at the appropriate point in the SaaS rollout.

## Phase 2C — React + Vite Frontend

- [ ] Build the React application and connect it to FastAPI.
- [ ] Implement chat history, new conversation, message display, loading states, and error states.
- [ ] Support readable Markdown and syntax-highlighted code.
- [ ] Add document upload/management and citations if not already covered in the UI.
- [ ] Add the Normal Mode / Coding Mode selector.
- [ ] Keep API and authentication configuration out of hard-coded source code.
- [ ] Test the frontend against API failures, slow responses, and expired sessions.

## Phase 2D — Normal Mode and Coding Mode

### Normal Mode
- [ ] General conversation and explanations.
- [ ] Search and appropriate existing tools.
- [ ] Document question-answering with sources.
- [ ] Clear separation between retrieved evidence and model-generated explanations.

### Coding Mode
- [ ] Code explanation and review.
- [ ] Debugging and error analysis.
- [ ] Code generation and refactoring suggestions.
- [ ] Project-aware context through explicitly scoped file-reading tools when implemented.
- [ ] Test-generation and code-review workflows.
- [ ] Require permission checks and confirmation before consequential file writes or command execution.

### Shared mode requirements
- [ ] Define separate instructions and tool allowlists where appropriate.
- [ ] Test mode selection, correct routing, context handling, and tool choice.
- [ ] Avoid treating a mode label or prompt alone as a security boundary.

## Phase 2E — MCP / FastMCP Integration

- [ ] Specify the exact use cases for MCP before selecting dependencies.
- [ ] Decide which tools/resources should remain internal and which should be exposed through MCP.
- [ ] If connecting to external MCP servers, define connection configuration and trust boundaries.
- [ ] Validate tool inputs and results; do not treat tool output as trusted instructions.
- [ ] Add timeouts, permission checks, bounded execution, and safe error handling.
- [ ] Test MCP integration independently from the normal REST API.
- [ ] Keep credentials and server configuration on the backend.

**Phase 2 exit condition:** React uses FastAPI contracts, both modes behave distinctly and pass tests, and any MCP integration has defined permissions and failure handling.

---

# Phase 3 — Multi-User SaaS Foundations

**Goal:** Prepare Infinity AI for private, multi-user hosted use without losing control of user data.

## Identity, Users, and Workspaces

- [ ] Integrate Clerk for authentication after validating the proof of concept and current plan requirements.
- [ ] Map Clerk's external subject to an internal Infinity AI user ID.
- [ ] Keep identity-provider code behind a small authentication boundary.
- [ ] Create a personal workspace for each new user.
- [ ] Add workspace membership records now, but defer shared team functionality until required.
- [ ] Implement authorization in FastAPI; frontend route guards are not a security boundary.
- [ ] Enforce ownership and access rules across conversations, documents, vectors, memories, checkpoints, and tool operations.

## Data and Security

- [ ] Migrate to PostgreSQL using versioned database migrations when appropriate.
- [ ] Define tenant ownership fields, constraints, and safe deletion semantics.
- [ ] Evaluate PostgreSQL Row-Level Security as defense in depth, with correctly restricted database roles and tests.
- [ ] Use private object storage for hosted uploads.
- [ ] Ensure vector retrieval is scoped to authorized workspace resources.
- [ ] Add rate limits, upload limits, resource limits, secret management, backup, restore, and account deletion workflows.
- [ ] Test cross-user access denial before public launch.

## Evaluation and Operations

- [ ] Maintain an automated evaluation set for chat, tools, and RAG.
- [ ] Track quality, latency, failures, and model usage.
- [ ] Keep LangSmith optional and provide local or self-managed diagnostics as appropriate.
- [ ] Add production monitoring, health checks, backup validation, and an incident-response checklist.
- [ ] Verify current hosting, authentication, storage, and LLM-provider costs before public release.

**Phase 3 exit condition:** Authentication, private workspaces, data isolation, deletion behavior, usage limits, observability, and recovery are tested in a staging environment before opening public signup.

---

# Phase 4 — Ecosystem and Advanced Capabilities

- [ ] Evaluate integrations such as Google Drive, Notion, Slack, Discord, and email based on actual user needs.
- [ ] Add external knowledge-base ingestion with appropriate permissions and sync behavior.
- [ ] Introduce background workers for indexing and long-running work when needed.
- [ ] Consider hybrid search, reranking, and more advanced RAG patterns where evaluations show a benefit.
- [ ] Add bounded plan-and-execute workflows after individual tools and agent runs are reliable.
- [ ] Add team workspaces, invitations, and role-based membership after defining the product requirements.
- [ ] Evaluate dedicated vector infrastructure only when load, retrieval, or operations justify it.

Do not assume every item in this phase is required. Prioritize based on user value, measured quality, security, and cost.

---

## 4. Engineering Principles

1. **Inspect before changing:** understand the existing implementation and reproduce issues before editing code.
2. **Small changes:** implement one agreed task at a time.
3. **Test every phase:** keep regression tests and record results.
4. **Measure quality:** compare RAG, tool selection, reliability, and latency against a baseline.
5. **Security outside the LLM:** the model never decides its own authorization or tool permissions.
6. **Tenant isolation:** all future user-owned data and AI operations must respect verified ownership and workspace scope.
7. **Privacy-conscious memory:** define what can be stored, why, for how long, and how it can be reviewed or deleted.
8. **Keep provider boundaries:** authentication, LLM, storage, and MCP integrations should be replaceable where it is practical.
9. **Avoid premature complexity:** no microservices, Kubernetes, or complex autonomy without a demonstrated need.
10. **Preserve working code:** refactor only when the benefit is clear and tests protect existing behavior.
11. **Keep plans honest:** a planned checkbox is not complete until the behavior is implemented and tested.

## 5. Working Method

For each small task:

1. Explain the problem and options.
2. Agree on the change and its acceptance criteria.
3. Inspect the relevant code before editing it.
4. Implement only the agreed scope.
5. Run focused tests and relevant regression tests.
6. Record the result and update this plan.
7. Move to the next task only after reviewing the outcome.

## 6. Current Next Step

**Start with Phase 1A: read-only repository audit and baseline.** No code changes should be made until the existing project structure and behavior have been reviewed and the first set of improvements is agreed upon.

## 7. Open Decisions for Future Brainstorming

- Which evaluation cases and quality thresholds should define success?
- Which existing code should be retained, refactored, or replaced after the audit?
- What exact behavior and tool permissions distinguish Normal Mode from Coding Mode?
- Will Infinity AI act as an MCP client, expose its own MCP server, or both?
- Which embedding model and hosted vector approach, if any, meet quality, privacy, and cost requirements?
- Which hosting provider and deployment topology fit the first public release?
- When should shared team workspaces, paid plans, and advanced autonomous workflows be introduced?

These decisions remain open and should be resolved through further brainstorming before implementation.
