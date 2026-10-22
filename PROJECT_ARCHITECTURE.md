# Infinity AI - Project Architecture

## 📐 System Architecture Overview
Infinity AI follows a **Modular Agentic Architecture**. It separates the user interface, the orchestration logic (the "brain"), the memory system, and the tool execution layer.

### 1. High-Level Component Diagram
`User` $\rightarrow$ `Streamlit Frontend` $\rightarrow$ `LangGraph Orchestrator` $\rightarrow$ `LLM` $\rightarrow$ `Tools` $\rightarrow$ `External APIs/Vector DB`

---

## 🧩 Component Deep Dive

### A. Frontend (Streamlit)
- **Role:** Handles user input, displays streaming AI responses, and manages session-based thread selection.
- **Key State:** Maintains `thread_id` and `message_history` to sync with the backend.
- **PDF Upload:** Forwards file bytes to the backend for immediate indexing.

### B. Orchestration Layer (LangGraph)
The "Brain" of the system. It uses a **StateGraph** to manage the flow of conversation.
- **Nodes:**
    - `chat_node`: The LLM decides whether to respond directly or call a tool.
    - `tools`: A `ToolNode` that executes the requested tool and returns the result to the LLM.
- **Edges:**
    - `START` $\rightarrow$ `chat_node`
    - `chat_node` $\rightarrow$ `tools` (Conditional: if tool call is present)
    - `tools` $\rightarrow$ `chat_node` (Loop: back to LLM to synthesize the answer)

### C. Memory System
- **Short-term Memory:** Managed via `ChatState` (a TypedDict) that tracks the current conversation messages.
- **Persistence:** `SqliteSaver` acts as a checkpointer, saving the graph state to `chatbot.db` after every turn. This allows the agent to "remember" conversations across restarts.
- **Thread Isolation:** Every conversation has a unique `thread_id`, ensuring data and context do not leak between chats.

### D. RAG Pipeline (Retrieval-Augmented Generation)
- **Ingestion:** `PDF` $\rightarrow$ `PyPDFLoader` $\rightarrow$ `RecursiveCharacterTextSplitter` $\rightarrow$ `FAISS Vector Store`.
- **Retrieval:** The `rag_tool` takes a user query $\rightarrow$ performs similarity search in the thread's FAISS index $\rightarrow$ returns the top-k relevant chunks.
- **Scope:** The vector store is **ephemeral and per-thread**, meaning documents uploaded in one chat are not accessible in another.

### E. Tool Integration
- **Web Search:** Tavily API for real-time information.
- **Finance:** Alpha Vantage API for stock quotes.
- **Logic:** Python-based calculator for deterministic math.
- **Knowledge:** The internal RAG pipeline for PDF content.

---

## 🛠️ Data Flow
1. **Input:** User sends a message in a specific thread.
2. **Reasoning:** LLM analyzes the message and decides: *"Do I need a tool?"*
3. **Execution:** If yes, the orchestrator routes to the `tools` node $\rightarrow$ Tool executes $\rightarrow$ Result returns to LLM.
4. **Synthesis:** LLM combines the tool output with its internal knowledge to generate a final response.
5. **Persistence:** The final state is saved to the SQLite database.
6. **Output:** Streamlit streams the response to the user.
