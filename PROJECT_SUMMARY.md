# Infinity AI - Project Summary (Phase 1)

## 🚀 Project Overview
**Infinity AI** is an advanced AI-powered chatbot application that combines the power of Large Language Models (LLMs), Retrieval-Augmented Generation (RAG), and agentic tool-use. It is designed to provide a seamless user experience for chatting with documents, querying live data, and maintaining persistent conversation histories.

## 🛠️ Core Features
- **RAG (Retrieval-Augmented Generation):** Users can upload PDF documents which are indexed on-the-fly. The AI uses a specialized `rag_tool` to retrieve relevant context from these documents to provide accurate, grounded answers.
- **Agentic Tool Use:** The system is not just a chatbot; it is an agent capable of choosing and executing tools:
    - `TavilySearchResults`: For real-time web search.
    - `get_stock_price`: For fetching live financial data via Alpha Vantage.
    - `calculator`: For precise arithmetic operations.
    - `rag_tool`: For document-specific queries.
- **Persistent Memory & Threading:** 
    - Powered by **LangGraph** and **SqliteSaver**, the app supports multiple independent chat threads.
    - Conversations are persisted in a SQLite database (`chatbot.db`), allowing users to return to past chats.
- **Streaming UI:** A modern **Streamlit** frontend that supports real-time message streaming and visual status updates when tools are being executed.

## 🏗️ Tech Stack
- **Frontend:** Streamlit
- **Orchestration:** LangGraph (StateGraph, Checkpointers)
- **LLM:** ChatGroq (GPT-OSS-120B)
- **Embeddings:** HuggingFace (`sentence-transformers/all-MiniLM-L6-v2`)
- **Vector Store:** FAISS (for efficient document retrieval)
- **Database:** SQLite (via LangGraph SqliteSaver)
- **Tools:** Tavily API, Alpha Vantage API

## 📈 Progress Summary
The project has successfully moved from a basic chatbot to a sophisticated agentic system. Key milestones achieved include:
1. Implementation of the basic LangGraph state machine.
2. Integration of multi-tool support.
3. Development of a per-thread RAG pipeline for PDFs.
4. Implementation of persistent thread management and conversation loading.
5. Refinement of the UI for a professional "AI Assistant" feel.
