# Handoff: local Markdown retrieval for Pi

## Request for the next session

Implement a local content index for this Obsidian/Markdown repository using **LlamaIndex's persisted simple vector store**. Pi should retrieve relevant passages from the index, read the original notes when needed, and compose answers grounded in those notes. The user requested this handoff only in the current session: no index, scripts, dependencies, or Pi skill have been installed or created yet.

This document records decisions and context, **not an implementation plan**. Confirm any unspecified design choices with the user rather than treating earlier suggestions as commitments.

## Decisions and constraints from the conversation

- Use LlamaIndex's built-in `SimpleVectorStore`, persisted to local disk. Do not introduce Chroma, SQLite, or a custom Go RAG program for the initial version.
- No continuously running local server, model-serving app, Docker container, Ollama, or llama.cpp process. Short-lived Python commands called by Pi are acceptable.
- Use a **local embedding model**. Pi is the language model that reasons over retrieved passages and writes the final answer; the retrieval program does not need a separate answer-generating model or a hosted embedding API.
- The user wants a Pi skill to make use of the index when answering questions about the notes. The skill should retrieve relevant passages and verify assertions against source Markdown rather than treating a similarity score as proof.
- Notes include English and Greek. `BAAI/bge-m3` was recommended as an initial multilingual embedding model, **not yet benchmarked or finally approved**. It is relatively large and must be downloaded on first use; a faster/smaller multilingual option remains open if startup or resource use is unacceptable. The same embedding model must be used for indexing and queries; changing it requires rebuilding embeddings.
- The index should carry source paths and enough context to verify/cite the original note. Precise line citations need to be deliberately implemented or checked against source files; LlamaIndex's basic file loader does not automatically guarantee them.
- Exclude generated output and non-note material, especially `graphify-out/`, `.git/`, `.trash/`, and `.obsidian/`. Check other sensitive or irrelevant folders with the user before indexing. A copied index contains note text and should be kept local and out of Git.
- A simple rebuild-on-demand was floated as an initial freshness mechanism; incremental updates, deletion handling, automatic refresh, citation format, and performance thresholds were **not decided**. Do not present these as completed or agreed features.

## Repository facts checked in this session

- Working directory: `/Users/vatikiot/Notes`. The repository is an Obsidian vault with Markdown at the root and in nested folders, plus some Greek-language notes.
- A filesystem count found **1,086 `.md` files** outside `.git/` and `graphify-out/` at the time of inspection. Graphify's existing report lists roughly **1,386,366 words** and 1,087 detected files; these counts may change.
- Existing `graphify-out/graph.json` maps files and headings, but does **not** store paragraph content. At inspection it contained 1,086 page nodes, 4,852 heading nodes, and no stored paragraph text. Its report lists 0 semantic-extraction tokens. Treat it as complementary navigation, not the planned passage index.
- `.gitignore` already ignores `graphify-out/`, but has no rule for the proposed retrieval index. Do not write an index into tracked content without adding an appropriate ignore rule or using an external location.
- `python3` currently resolves to `/usr/bin/python3`, **Python 3.9.6**. Check the installed interpreter and current LlamaIndex package compatibility before installing anything; a separate modern Python environment may be necessary. Node.js is installed, but the chosen path is Python.
- There is an existing unrelated modification to `.obsidian/snippets/supercharged-links-gen.css`. Leave it untouched.
- No RAG implementation or dependencies were installed during this conversation.

## Technical distinctions worth preserving

- The `pip` package `llama-index-embeddings-huggingface` installs the integration and Python runtime dependencies. A model such as `BAAI/bge-m3` is fetched separately when first instantiated, then cached for reuse. Offline operation requires that first download to have succeeded.
- LlamaIndex defaults can use OpenAI if an embedding model or LLM is not set explicitly. Configure a local embedding model for **both** index construction and retrieval. For Pi to answer, use `index.as_retriever(...)` and return evidence, rather than using an answer-generating `query_engine` with an implicit external LLM.
- LlamaIndex's simple vector store persists with `index.storage_context.persist(persist_dir=...)`. In a later process it can be reopened via `StorageContext.from_defaults(persist_dir=...)` and `load_index_from_storage(...)` using the same embedding model. Persistence is not automatic file watching.
- Semantic search finds potentially related passages; it may miss exact names, numbers, and codes. Consider source-file or keyword search as a complementary check, but no hybrid design was chosen.
- A short-lived query process will reload the embedding model for each invocation. Measure that startup time before promising interactive performance.
- The earlier time estimate was **a few hours for a small proof of concept**, and approximately **1–2 days for a usable vault-wide version with exclusions, source tracking, and tests**. These were rough estimates, not a commitment.

## Primary references consulted

- [Simple vector store and persistence example](https://developers.llamaindex.ai/python/examples/vector_stores/simpleindexdemo/)
- [Local Hugging Face embedding configuration and model-change caveat](https://developers.llamaindex.ai/python/framework/module_guides/models/embeddings/)
- [Markdown loading, recursion, exclusions, and file metadata](https://docs.llamaindex.ai/en/stable/module_guides/loading/simpledirectoryreader/)
- [Retriever API, without an answer-generating model](https://developers.llamaindex.ai/python/framework/module_guides/querying/retriever/)
- [Markdown header-based parser](https://developers.llamaindex.ai/python/framework-api-reference/node_parsers/markdown/)
- [BAAI/bge-m3 model card](https://huggingface.co/BAAI/bge-m3)
- [Pi skill format and discovery](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent/docs) (prefer the installed Pi documentation at `/opt/homebrew/lib/node_modules/@earendil-works/pi-coding-agent/docs/skills.md` when implementing)

## Status

Handoff file only. No implementation, dependency installation, model download, index build, or skill creation has occurred.
