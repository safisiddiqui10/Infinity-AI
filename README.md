# ♾️ Infinity AI

Infinity AI is an advanced agentic chatbot that combines Large Language Models (LLMs) with Retrieval-Augmented Generation (RAG) and real-time tool use to provide a sophisticated, document-aware AI assistant.

## 🚀 Key Features

- **📚 Document-Aware Chat (RAG):** Upload PDF documents and chat with them. The system uses FAISS for efficient vector search, allowing the AI to retrieve grounded facts from your files.
- **🛠️ Agentic Tool Use:** The AI isn't just a chatbot; it's an agent that can autonomously decide to use:
    - **Web Search:** Real-time internet access via Tavily.
    - **Financial Data:** Live stock prices via Alpha Vantage.
    - **Calculator:** Precise arithmetic for complex calculations.
    - **RAG Tool:** Specialized retrieval from uploaded PDFs.
- **🧠 Persistent Memory:** Powered by **LangGraph** and **SQLite**, the application supports multiple independent chat threads that persist across sessions.
- **⚡ Modern UI:** A streamlined **Streamlit** interface with real-time streaming responses and tool-execution status updates.

## 🏗️ Technical Architecture

- **Orchestration:** [LangGraph](https://github.com/langchain-ai/langgraph) (StateGraph)
- **LLM:** ChatGroq (GPT-OSS-120B)
- **Vector Store:** FAISS (on-disk persistence)
- **Embeddings:** HuggingFace (`all-MiniLM-L6-v2`)
- **Frontend:** Streamlit
- **Database:** SQLite (via SqliteSaver)

## 🛠️ Installation & Setup

### Prerequisites
- Python 3.11+
- A Groq API Key
- A Tavily API Key
- An Alpha Vantage API Key

### Setup Steps

1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-username/Infinity-AI.git
   cd Infinity-AI
   ```

2. **Create a virtual environment:**
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

4. **Configure Environment Variables:**
   Create a `.env` file in the root directory:
   ```env
   GROQ_API_KEY="your_groq_key"
   TAVILY_API_KEY="your_tavily_key"
   ALPHA_VANTAGE_API_KEY="your_alpha_vantage_key"
   ```

5. **Run the application:**
   ```bash
   streamlit run app.py
   ```

## 🗺️ Project Roadmap

- [x] **Phase 1A:** Repository Audit & Baseline $\checkmark$
- [x] **Phase 1B:** Agent & Tool Reliability $\checkmark$
- [ ] **Phase 1C:** RAG Quality & Document Management (Upcoming)
- [ ] **Phase 1D:** Conversation & Persistence Reliability
- [ ] **Phase 1E:** Observability & UI Improvements
- [ ] **Phase 2:** New Product Architecture (React + FastAPI)

## 📄 License
Distributed under the MIT License. See `LICENSE` for more information.
