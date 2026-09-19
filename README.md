# Mini AI Knowledge Assistant (RAG)

A simple RAG-based application that answers questions grounded in a provided
set of documents.

## Approach

1. **Document ingestion** (`ingest.py`) — PDFs in `data/` are loaded with
   `PyPDFLoader`, split into ~800-character chunks with 150-character overlap
   using `RecursiveCharacterTextSplitter`, embedded with the free local
   `sentence-transformers/all-MiniLM-L6-v2` model, and stored in a persistent
   **Chroma** vector store.
2. **Retrieval + generation** (`rag_chain.py`) — a user's question is embedded
   with the same model, the top-k most similar chunks are retrieved from
   Chroma, and passed as context to **Gemini 1.5 Flash** along with a strict
   prompt instructing it to answer only from the given context.
3. **Interface** (`app.py`) — a Streamlit chat interface that accepts
   questions, displays generated answers, keeps conversation history for the
   session, and shows the exact source chunks each answer was grounded in.

## Setup

```bash
pip install -r requirements.txt

# add your PDF(s) to the data/ folder, then:
python ingest.py

# set your free Gemini API key (https://aistudio.google.com/apikey)
export GOOGLE_API_KEY=your_key_here   # or paste it into the sidebar

streamlit run app.py
```

## Design decisions

- **Chroma over FAISS/Pinecone**: simplest to set up locally with zero
  external dependencies, persists to disk automatically.
- **Local embeddings (MiniLM) over API embeddings**: free, fast, no rate
  limits — keeps the pipeline usable without any paid service.
- **Gemini 1.5 Flash**: free tier available, fast inference, good enough
  quality for grounded Q&A over a small document set.
- **Strict grounding prompt**: the model is explicitly instructed to say it
  doesn't know rather than hallucinate when the answer isn't in the
  retrieved context — directly addresses the "answers primarily based on
  the provided knowledge source" requirement.

## Bonus features implemented

- ✅ Source citations (expandable per-answer, shows file + page + excerpt)
- ✅ Conversation history (persists for the Streamlit session)
- ⬜ Multi-document support (already works — just drop more PDFs in `data/`
  before running `ingest.py`)
- ⬜ Deployment (not done — can be deployed to Streamlit Community Cloud if
  needed)

## Results

*(Fill in after testing: 3-5 example questions, the answers generated, and
whether they were correctly grounded in the source documents.)*

## AI tool disclosure

This project was built with assistance from Claude (Anthropic) for code
scaffolding, architecture decisions, and documentation.
