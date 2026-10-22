# Infinity AI - Phase 1A Evaluation Set

This document contains the test cases used to establish the baseline performance of Infinity AI.

## 🧪 Test Categories & Cases

### 1. RAG (Retrieval-Augmented Generation)
*Assuming a standard PDF is uploaded for these tests.*
- **Direct Fact:** "What is the main topic of this document?"
- **Specific Detail:** "Find the specific value/date mentioned on page [X]."
- **Synthesis:** "Summarize the three key takeaways from the second section."
- **Out-of-Bounds:** "What does this document say about [Topic NOT in PDF]?" (Expected: AI should state it's not in the document).
- **Cross-Reference:** "Compare the conclusion on page [X] with the introduction on page [Y]."

### 2. Tool Integration
- **Search:** "What is the current news regarding [Current Event]?"
- **Finance:** "What is the current stock price of NVDA?"
- **Math:** "Calculate 1234.56 * 789.10 / 45.2."
- **Multi-Tool (Complex):** "Find the current stock price of AAPL and tell me if it's higher than the average price mentioned in my uploaded document."

### 3. Thread & Persistence
- **Isolation:** Start Thread A (Upload PDF A) $\rightarrow$ Start Thread B (Upload PDF B) $\rightarrow$ Ask Thread A about PDF B. (Expected: No leak).
- **Resumption:** Chat in Thread A $\rightarrow$ Restart App $\rightarrow$ Load Thread A $\rightarrow$ Ask "What were we just talking about?".
- **Reset:** Start Chat $\rightarrow$ Use "New Chat" $\rightarrow$ Verify `thread_id` changed and history cleared.

### 4. Edge Cases & Reliability
- **Empty PDF:** Upload a blank or nearly blank PDF $\rightarrow$ Ask a question.
- **Large PDF:** Upload a 50+ page document $\rightarrow$ Test retrieval latency.
- **Invalid Math:** Ask the calculator to divide by zero.
- **Tool Failure:** Simulate/Request a search for a non-existent topic to check error handling.
