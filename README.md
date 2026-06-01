# Healthcare Prior Authorization Q&A

A RAG-based system that lets you ask plain English questions about Medicare coverage policies and get cited, grounded answers.

Built this because prior auth is one of the most painful parts of healthcare admin — a billing coordinator can spend 20+ minutes digging through CMS policy documents to answer something like "is this MRI covered?" This brings that down to a few seconds.

## What it does

Takes a clinical question, retrieves the relevant CMS coverage policy, and returns a structured answer with the policy ID cited.

```
Q: My patient has had back pain for 3 weeks with no other symptoms. Is an MRI covered?

NOT COVERED

MRI for lumbar spine requires one of the following:
- Radiculopathy with neurological signs
- Suspected spinal stenosis
- Red flag symptoms (unexplained weight loss, fever, history of cancer, etc.)
- Failed conservative therapy >= 6 weeks

Since symptoms are under 6 weeks with no red flags, MRI is not covered at this time.
Policy: LCD-L34091
```

## Stack

- **Embeddings** — `sentence-transformers/all-MiniLM-L6-v2` running locally
- **Vector store** — FAISS
- **LLM** — Groq (llama-3.1-8b-instant)
- **Data** — CMS Local Coverage Determinations (public, no auth required)

No LangChain chains — just direct FAISS retrieval + Groq API call. Kept it simple.

## Setup

```bash
git clone https://github.com/yourusername/healthcare-rag-assistant
cd healthcare-rag-assistant
pip install -r requirements.txt
```

Add your Groq API key (free at console.groq.com):
```python
GROQ_API_KEY = "your-key-here"
```

Run the notebook:
```bash
jupyter notebook notebooks/healthcare_rag_assistant.ipynb
```

## Results

Retrieval accuracy on 4-policy prototype:
```
✓ 'obstructive sleep apnea CPAP coverage'  → LCD-L33393 ✓
✓ 'lumbar spine imaging criteria'           → LCD-L34091 ✓
✓ 'botox injection for migraine'            → LCD-L36158 ✓
✓ 'PT treatment plan documentation'         → LCD-L34028 ✓

retrieval accuracy: 4/4 = 100% (top-3 recall)
```

Note: this is a small prototype benchmark — 4 policies, queries I wrote myself. Real production eval would need 100+ adversarial queries from actual billing staff.

## Policies covered

| Policy ID | Title |
|---|---|
| LCD-L33393 | CPAP Therapy |
| LCD-L34091 | Lumbar Spine MRI |
| LCD-L35050 | Cardiac Rehabilitation |
| LCD-L36158 | Botulinum Toxin Injections |
| LCD-L34028 | Physical Therapy Guidelines |

## What's next

- Ingest full CMS LCD database (700+ policies) via public API
- Healthcare-specific embeddings (ClinicalBERT) for better clinical term matching
- Hybrid search: dense retrieval + BM25 for ICD code lookup
- Structured JSON output for integration with EHR workflows
- HIPAA-compliant deployment with audit logging
