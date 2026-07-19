| No. | Question | Answer |
|-----|----------|--------|
| 1 | What occurs during the MCP initialization phase? | **A.** The client and server exchange protocol versions and supported capabilities. |
| 2 | Which approaches are supported by OCI Enterprise AI Agents? | **C.** Call the Responses API directly or deploy a hosted agent application. |
| 3 | What is long-term memory in OCI Enterprise AI Agents? | **C.** Durable memory shared across conversations. |
| 4 | Which statement describes the OCI Responses API? | **B.** It provides an OpenAI-compatible API. |
| 5 | What is short-term memory compaction in OCI Enterprise AI Agents? | **A.** A summarization process for long conversations. |
| 6 | What is the purpose of the Vector Stores API? | **B.** Indexing and retrieving data by meaning. |
| 7 | In the given LangChain chain, what is the role of `StrOutputParser()`? | **D.** It extracts plain text from the model response. |
| 8 | Which native column type does Oracle AI Database use for storing vector embeddings? | **D.** VECTOR |
| 9 | What is the default distance metric for `VECTOR_DISTANCE` in Oracle for non-BINARY vectors? | **C.** COSINE distance |
| 10 | Why are docstrings especially important when defining LangChain tools with the `@tool` decorator? | **C.** They tell the LLM when and how to use the tool. |
| 11 | Which tasks is handled automatically by LangChain when using `agent.invoke()`? | **B.** Managing conversation state, formatting API requests, and routing tool execution across the agent loop. |
| 12 | In the OpenAI Agents SDK, when are input guardrails and output guardrails evaluated? | **B.** Input guardrails run before the agent processes input; output guardrails run before the response is returned. |
| 13 | In the OpenAI Agents SDK, what is the role of the Runner? | **D.** It executes the agent loop. |
| 14 | In the OpenAI Agents SDK, how does a Handoff differ from the Manager pattern? | **A.** A Handoff transfers control; a Manager keeps control. |
| 15 | In the context of MCP, what does the USB-C for AI analogy emphasize? | **C.** MCP is a standardized interface. |
| 16 | What integration problem does MCP address? | **B.** N × M custom-connector problem between AI apps and tools. |
| 17 | Which statement best describes a key advantage of semantic search over traditional keyword search? | **A.** It retrieves results based on meaning rather than exact words. |
| 18 | In the OpenAI Agents SDK, how does the model select which tool to call? | **C.** It uses tool names, descriptions, and schemas. |
| 19 | What does the `@function_tool` decorator do in the OpenAI Agents SDK? | **A.** Converts a regular Python function into a tool the agent can call. |
| 20 | Which SQL function computes distance between vectors in Oracle AI Vector Search? | **C.** `VECTOR_DISTANCE()` |
| 21 | Which four behaviors does every Select AI Agent perform? | **C.** Plan, Use tools, Reflect, and Remember. |
| 22 | Compared with a standalone LLM call, an AI-agent architecture commonly adds which capabilities? | **A.** Tool access, memory handling, and iterative execution. |
| 23 | Which behavior is NOT a characteristic of modern LLM-based AI agents? | **A.** Requiring every execution path to be predefined. |
| 24 | Which OCI services are used for observability and auditing of deployed AI agents? | **A.** OCI Logging, OCI Monitoring, and OCI Audit. |
| 25 | Why is chunking necessary before generating embeddings for large documents? | **C.** To overcome token limits for large documents. |
| 26 | Which statement describes the purpose of the OpenAI Responses API? | **B.** It sends input to an OpenAI model and returns generated output. |
| 27 | Which three model categories are available through OCI Enterprise AI Models? | **C.** Chat models, Embed models, and Rerank models. |
| 28 | What is the purpose of OCI Enterprise AI Governance? | **C.** Applying guardrails, identity controls, and network security controls to AI workloads. |
| 29 | Which statement describes an MCP Host? | **C.** It coordinates the LLM, user interaction, and MCP client connections to servers. |
| 30 | According to the MCP architecture model, which statement describes the client-server relationship? | **A.** An MCP client maintains a dedicated connection to an MCP server. |
| 31 | What is the strategic theme behind agentic AI capabilities in Oracle AI Database? | **D.** Bringing AI capabilities natively into the database. |
| 32 | How are tool calls handled between the LLM and the application? | **B.** It generates a tool-call request for the application to validate and run. |
| 33 | Which Python package is installed first for a simple OCI Responses API setup? | **D.** `openai` |
| 34 | Which authentication approach should be used for production-grade access to OCI Enterprise AI services? | **C.** OCI IAM authentication with signed requests and IAM policies. |
| 35 | Which prompt addition is used for zero-shot Chain-of-Thought prompting? | **A.** "Let's think step by step." |
| 36 | What is Oracle AI Database Private Agent Factory? | **C.** A no-code platform for building agents. |
| 37 | Which standard MCP transport supports remote or network-accessible deployments where multiple clients may connect to the same server? | **C.** Streamable HTTP |
| 38 | Which message format does MCP use for client–server communication? | **D.** JSON-RPC 2.0 |
| 39 | Which description defines memory poisoning in AI-agent systems? | **B.** Malicious content inserted into persistent memory stores. |
| 40 | Which behavior best describes the purpose of reranking in a Retrieval-Augmented Generation (RAG) pipeline? | **B.** It improves the ordering of retrieved documents by relevance before sending them to the LLM. |