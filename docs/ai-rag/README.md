# AI and RAG Docs

Use this folder for practical AI features that improve the commerce experience without weakening security or architecture.

## Planned Documents

| Document | Purpose |
| --- | --- |
| `rag-architecture.md` | Retrieval-augmented generation design and request flow. |
| `vector-index-strategy.md` | Vector DB choice, schema, indexing triggers, and reindexing strategy. |
| `semantic-search-plan.md` | Hybrid keyword/vector product search behavior. |
| `assistant-guardrails.md` | Authentication, authorization, safety, confidence thresholds, and logging. |
| `rag-evaluation.md` | Test questions, retrieval quality checks, hallucination checks, and support metrics. |

## AI Defaults

- Vector DB: Amazon OpenSearch Serverless vector search.
- Alternative: pgvector if PostgreSQL is selected later.
- LLM and embeddings: Amazon Bedrock.
- First use cases: semantic product search, support assistant, admin knowledge assistant, and recommendations.
