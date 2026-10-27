# RAG QA Bot

A single-file Gradio application that accepts a PDF and answers a question using IBM watsonx models and a Chroma vector store.

## System Design

### Requirements

- Load one uploaded PDF, split its pages into text chunks, retrieve relevant chunks, and return a generated answer.

**Constraints**
- The vector index is created in process for each request; no persistent document store or result database is configured.
- The launcher enables a Gradio share link.

### Pipeline

```text
PDF upload → PyPDFLoader → 1,000-character chunks / 50 overlap
           → watsonx Slate embeddings → in-memory Chroma
Question   → retriever → RetrievalQA → watsonx Granite response → Gradio
```

`qabot.py` configures `ibm/slate-125m-english-rtrvr` embeddings and `ibm/granite-3-2-8b-instruct` for generation.

### Trade-offs

| Decision | Benefit | Cost |
| --- | --- | --- |
| Rebuild a compact Chroma index in the request | Simple, self-contained demo flow | Repeats PDF parsing and embedding work for every question |
| Return only the answer text | Minimal interface | Retrieved source documents are not shown |

### What to Revisit

- Reuse an index for repeated questions on the same document.
- Return retrieved passages so users can check grounding.
- Add a dependency manifest and decide whether uploaded PDFs should be shared through the Gradio tunnel.

## Run

Install the packages imported by `qabot.py` in a Python environment, configure watsonx access, then run:

```bash
python qabot.py
```

The app binds to `127.0.0.1:7860` and requests a Gradio share link.
