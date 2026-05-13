# UK Pension Guidance RAG Assistant

A Retrieval-Augmented Generation (RAG) pipeline built to answer questions 
grounded in publicly available UK pension guidance documents.

Built as a personal project to demonstrate RAG pipeline design, LLM 
evaluation frameworks, and responsible AI principles in a regulated 
financial services context.

---

## Architecture
PDF Documents → Text Extraction → Chunking → Embedding → ChromaDB
                                                              ↓
User Question → Embedding → Semantic Search → Retrieved Chunks
                                                              ↓
                        Prompt (question + context) → Llama 3.1
                                                              ↓
                                          Grounded Answer + Sources

## Key Design Decisions

**1. RAG over fine-tuning**
Chose RAG because the knowledge base needs to be updatable without 
retraining. In a production pension context, regulatory guidance changes 
frequently — RAG allows knowledge updates by adding documents, not 
retraining the model.

**2. Uncertainty over hallucination**
The prompt explicitly instructs the model to admit when it lacks sufficient 
context rather than generate a plausible-sounding but ungrounded answer. 
In a regulated environment, a confidently wrong answer causes real harm.

**3. Local vector store**
ChromaDB runs in-process — no data is sent to an external service. 
Important design consideration for regulated environments where data 
residency and security controls are non-negotiable.

**4. Open-weight model**
Using Llama 3.1 via Groq rather than a proprietary model. The architecture 
is model-agnostic — swapping to OpenAI, Anthropic, or a self-hosted model 
requires changing one line. Avoids vendor lock-in.

**5. Evaluation built in from day one**
A lightweight retrieval relevance scoring framework is included from the 
start — not added after deployment. Systematic evaluation is a governance 
requirement, not an afterthought.

---

## Evaluation Results

| Question | Retrieval Score | Behaviour |
|----------|----------------|-----------|
| What is pension drawdown and risks? | 5/5 | Grounded answer with sources |
| Employer contribution requirements? | 1/5 | Correctly admits uncertainty |
| Pension on death before retirement? | 1/5 | Correctly admits uncertainty |
| Can I take pension as lump sum? | 5/5 | Grounded answer with sources |
| Defined benefit vs contribution? | 1/5 | Correctly admits uncertainty |

Scores of 1/5 reflect knowledge base gaps — not model failure. 
The correct fix is adding documents, not changing the architecture.

---

## Stack

| Component | Technology | Rationale |
|-----------|-----------|-----------|
| LLM | Llama 3.1 8B via Groq | Free, fast, OpenAI-compatible API |
| Embeddings | all-MiniLM-L6-v2 | Lightweight, no API cost, runs locally |
| Vector store | ChromaDB | Local, no external dependencies |
| PDF parsing | pypdf | Simple, reliable |
| Environment | Google Colab | Reproducible, no local setup required |

---

## Knowledge Base

- FCA Non-Advised Drawdown Pension Sales Review
- Defined Contribution Pension Schemes (The Pensions Regulator)
- How Your Employer's Pension Scheme Works (The Pensions Regulator)

---
## Sample Output — Agent Test Run
*Full implementation in `02_pension_agent.ipynb`*

| Query | Category | Confidence | Escalated | Reason |
|-------|----------|------------|-----------|--------|
| What is pension drawdown? | ANSWERABLE | HIGH | No | Answered from knowledge base |
| Can I take 25% tax free? | ANSWERABLE | HIGH | No | Answered from knowledge base |
| Drawdown vs annuity — health condition? | COMPLEX | N/A | Yes | Regulated financial advice |
| 58 years old, £200k — what should I do? | ANSWERABLE | LOW | Yes | Insufficient context |
| DB pension if employer goes bust? | ANSWERABLE | LOW | Yes | Insufficient context |
| Best mortgage deal right now? | COMPLEX | N/A | Yes | Out of scope |
---
**Three escalation paths demonstrated:**
- COMPLEX classification → immediate escalation before retrieval (Q3, Q6)
- ANSWERABLE + LOW confidence → escalation after generation (Q4, Q5)  
- ANSWERABLE + HIGH confidence → answer returned to user (Q1, Q2)

## Limitations & Next Steps

- **Knowledge base**: 3 documents — production would require comprehensive 
  coverage of pension rules, employer obligations, and death benefits
- **Evaluation**: lightweight scoring — production would use RAGAS framework 
  for faithfulness, answer relevance, and context precision metrics
- **Persistence**: ChromaDB resets each Colab session — production would use 
  a persistent vector store (Pinecone, Weaviate, or self-hosted Chroma)
- **Agentic layer**: see `02_pension_agent.ipynb` — LangGraph agent with 
  classification, conditional routing, confidence scoring, and human-in-the-loop 
  escalation across three trigger paths

---

## Running This Notebook

1. Open `01_pension_rag_pipeline.ipynb` in Google Colab
2. Add your Groq API key (free at console.groq.com) to Cell 2
3. Upload the pension PDF documents when prompted
4. Run all cells in order
