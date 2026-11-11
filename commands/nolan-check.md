---
description: Check that the Nolan knowledge base connection works and report which tenant it is reading.
disable-model-invocation: true
---

Run a one-shot health check of the Nolan knowledge base connection and report the result
concisely. Do not attempt any fixes as part of this check.

1. Are the `nolan-knowledge-base` MCP tools present in this session? If not, stop and report that
   the server is not loaded.
2. Call `search_knowledge_base` with the query `company overview`, which works without depending
   on any particular page existing.
3. Report:
   - **Connected** — the titles of a few pages it returned, which identify the knowledge base
     being read. No matching pages still proves the connection works; say so rather than
     reporting a failure.
   - **Not connected** — the HTTP status and the server's error message verbatim. Do not
     paraphrase it; `401`, `403`, and `429` mean different things and the wording distinguishes
     them.

If the check fails, name the `connect-nolan-knowledge-base` skill as the next step rather than
walking through setup here.
