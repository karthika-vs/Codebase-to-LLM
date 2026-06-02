# Codebase-to-LLM
1. The Parsing Engine (AST Extraction)
You do not use regular expressions (grep) to read the code, as they cannot understand scope or syntax. Instead, your backend utilizes an Abstract Syntax Tree (AST) parser. The industry standard for this is Tree-sitter, because it is extremely fast and supports over 60 programming languages.

Your pipeline feeds every source file into Tree-sitter, which strips away the actual logic (loops, variable assignments, inline comments) and extracts only the structural skeleton:

Definitions: Classes, function signatures, interfaces, and enums.

Signatures: Input parameters and return types.

Edges: Import statements and module requirements.

2. Dependency Resolution (Building the Graph)
Once the AST extracts the symbols from individual files, your backend must stitch them together. If auth.py imports verify_token from crypto.py, the system draws a directed edge between them.

Advanced indexers (like those powering Cursor or Claude Code) use a multi-strategy approach to resolve these connections:

Explicit Imports: Directly linking files based on explicit paths (e.g., import ./utils/logger).

LSP Type Resolution: For complex languages like C++ or Go, the indexer might query a Language Server Protocol (LSP) to accurately map method receivers and pointer indirections that an AST might miss.

3. The Compression & Formatting Layer
The resulting graph is still a complex data structure. The final step of Stage 1 is converting this graph into a text format the LLM can easily read, known as the Repo Map.

The Indexer traverses the graph and generates a condensed Markdown or JSON manifest. To optimize token usage, the Indexer often prioritizes files based on their centrality in the graph (e.g., core utility files get slightly more detailed summaries than isolated unit tests).

4. Modern Delivery: Model Context Protocol (MCP)
As of 2026, the modern way to expose this map to the LLM is via the Model Context Protocol (MCP). Instead of injecting a massive text map directly into the initial prompt, your indexer runs as an MCP Server. The LLM acts as the MCP Client, dynamically querying the Indexer for specific sub-graphs or file summaries exactly when it needs them, drastically reducing token consumption and API costs.


##2. The LLM Summarization Fallback (The "Slow Path")
If heuristics fail (or if the file is a complex custom configuration, like a massive YAML or XML file), the Indexer uses AI to do what the AST couldn't.

The Process: The backend sends the raw file to a fast, cheap LLM (like Gemini 1.5 Flash or Claude 3 Haiku).

The Prompt: "Read this unformatted file. Summarize its purpose in one sentence and list any other files, databases, or services it references."

Integration: The AI-generated summary is injected directly into the Repo Map alongside the AST-generated summaries of the other files. It is slightly more expensive, but it ensures custom DSLs are accurately mapped.
