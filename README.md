# Neural Employee: 44-FZ Tender Consultant

🌐 **English** | [Русский](README.ru.md)

A RAG agent that advises suppliers on government procurement under Russian Federal Law 44-FZ. It answers questions from a knowledge base of 34 documents, cites the relevant articles of the law, and honestly says "I don't know" when information is missing. Built for Google Colab on the Russian-language model saiga_mistral_7b.

## Installation

1. Open `neiro_sotrudnik_44fz_rag_agent.ipynb` in Google Colab (File → Upload notebook or from GitHub).
2. Add your Hugging Face token to Colab secrets (the key icon in the left panel) and reference the secret name in the `userdata.get(...)` cell.
3. Run the cells in order - dependencies are installed automatically in the first cell (`!pip install`).

## Features

| Technique | Purpose |
|---|---|
| saiga_mistral_7b (4-bit NF4 + LoRA) | Russian-language answer generation |
| Dense + BM25 ensemble with RRF | Hybrid search: semantics + keywords |
| HyDE | Better recall for complex queries |
| Cross-encoder reranker | Precise selection of relevant chunks |
| LongContextReorder | Optimal chunk order in the context |
| Knowledge Graph RAG | Answers about entity relationships |
| Phoenix tracing | Transparency of every pipeline step |
| Response Validation | Deterministic hallucination control |
| LRU cache | Faster repeated queries |
| Security | Topic guardrail, blacklist, query length limit |
| System prompt | Citations, clarification, disclaimer |
| Knowledge Maps (DDD) | Routing to domain experts |

## Phoenix tracing

Every query is traced in the Phoenix UI: embedding, retrieval, reranking, generation. This helps find bottlenecks and hallucination symptoms.

Project latency dashboard:

![Project latency dashboard](img/phoenix_project_overview_latency_dashboard.jpg)

Trace of a grounded response (bid security, referencing articles 44 and 96):

![RAG request trace](img/phoenix_rag_trace_grounded_response.jpg)

## Project structure

```
├── neiro_sotrudnik_44fz_rag_agent.ipynb   - main notebook (all pipeline stages)
├── Knowledge_graph_44fz.html              - interactive knowledge graph visualization (pyvis)
├── img/
│   ├── phoenix_project_overview_latency_dashboard.jpg
│   └── phoenix_rag_trace_grounded_response.jpg
├── README.md
├── README.ru.md
└── LICENSE
```

`storage_44fz/` (the vector index) is created when the notebook runs and is not committed.

## How it works

- **Request pipeline**: security → LRU cache → hybrid retrieval (Dense + BM25, RRF) → reranking → generation → validation → post-processing.
- **Hybrid retrieval**: dense vector search and BM25 results are merged with Reciprocal Rank Fusion.
- **Reranking**: the cross-encoder `BAAI/bge-reranker-base` reorders the chunks; `LongContextReorder` places the most relevant ones at the start and end of the context.
- **Validation**: a deterministic check that every article and number mentioned in the answer exists in the context; on a mismatch the answer is replaced with "I don't know".
- **Knowledge graph**: triplets (subject-predicate-object) are extracted from the documents and visualized with Pyvis.
- **Tracing**: Phoenix collects the query span tree (embedding, retrieval, reranking, generation) via OpenInference.
- **Security**: query topic guardrail, a blacklist of forbidden topics, and a 1000-character query length limit.

## Disclaimer

Answers are for informational purposes only and do not constitute legal advice. For a final decision please consult a qualified lawyer.

## License

[MIT](LICENSE)
