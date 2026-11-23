# Corrective RAG System

A Corrective Retrieval-Augmented Generation (CRAG) system that answers questions from documents.
Before generating an answer it **grades the retrieved chunks**, and when they are weak or irrelevant
it **rewrites the question and retrieves again**. If the documents still do not contain the answer,
the system says so instead of guessing.

## Features
- Ingestion of 4 file types: PDF, DOCX, TXT, CSV (extract, clean, split into section-based chunks)
- Embeddings with `BAAI/bge-base-en-v1.5` stored in ChromaDB
- Hybrid retrieval: vector search + BM25 keyword search, merged with reciprocal rank fusion
- Reranking with `cross-encoder/ms-marco-MiniLM-L-6-v2`
- LLM grader that keeps only the chunks that help answer the question
- Query rewriting (synonyms and formal terms) and re-retrieval when the context is weak
- Answers generated only from the verified context, with sources (file and section)
- Refusal for questions that are not covered by the documents
- Gradio interface that also shows when the question was rewritten

## Pipeline
```
Question
   |
   v
Hybrid retrieval (BGE + BM25) --> Cross-encoder rerank --> LLM grader
                                                              |
                         relevant chunks found? ----- yes --> Generate answer + sources
                                   |
                                   no (or the answer was "not found")
                                   v
                       Rewrite the question (3 new queries)
                                   |
                                   v
                       Retrieve again --> Grade again
                                   |
                  relevant chunks found? --- yes --> Generate answer + sources
                                   |
                                   no
                                   v
                     "I could not find the answer in the documents."
```

## Dataset
Synthetic documents for a fictional company, **NovaTech Solutions**, and its helpdesk product **NovaDesk**.
The files were written to make retrieval hard on purpose:
- similar numbers in different contexts (30 days for refunds, expense claims and data deletion; 14 vs 10 character passwords for employees and customers)
- similar rules for different things (a lost laptop must be reported in 2 hours, a lost badge in 24 hours)
- different words for the same idea (vacation / annual leave / time off)

| File | Type | Content |
|---|---|---|
| `HR_policy.pdf` | PDF | working hours, leave, probation, remote work, notice period |
| `Product_guide.pdf` | PDF | system requirements, API limits, SLA, security |
| `Employee_handbook.docx` | DOCX | expenses, training, performance reviews, salary |
| `IT_security_policy.txt` | TXT | passwords, lost devices, USB, phishing |
| `Support_FAQ.txt` | TXT | trial, refunds, cancellation, support hours |
| `Pricing.csv` | CSV | pricing plans |
| `Offices_directory.csv` | CSV | department heads and extensions |

## Evaluation
`eval_questions.csv` has 26 questions with the expected source and keywords:
- 10 **direct** questions
- 10 **needs_rewrite** questions (the wording is different from the documents)
- 6 **unanswerable** questions (the system must refuse)

Plain RAG (retrieve and answer) is compared with Corrective RAG on the same questions.

| Question type | Plain RAG | Corrective RAG |
|---|---|---|
| direct | XX% | XX% |
| needs_rewrite | XX% | XX% |
| unanswerable | XX% | XX% |
| **all** | **XX%** | **XX%** |

## Project structure
```
.
├── README.md
├── requirements.txt
├── .gitignore
├── corrective_rag.ipynb     # full pipeline, evaluation and Gradio app
├── app.py                   # standalone Gradio app
├── eval_questions.csv       # evaluation questions (kept outside the documents folder)
├── novatech_data/           # the 7 source documents
└── screenshots/
```

## How to run

### Option 1: Notebook (Kaggle / Colab / Jupyter)
1. Add the `novatech_data` folder and `eval_questions.csv` to the notebook (on Kaggle turn on Internet too)
2. Get a free API key from [Groq](https://console.groq.com)
3. Run the cells in order and enter the key when asked
4. The last cell starts the Gradio app

### Option 2: Standalone app
```bash
pip install -r requirements.txt
export GROQ_API_KEY="your_key_here"      # Windows PowerShell: $env:GROQ_API_KEY="your_key_here"
python app.py
```
Then open the link printed in the terminal.

## Tech stack
Python, ChromaDB, Sentence-Transformers, rank-bm25, Groq (`openai/gpt-oss-120b`), Gradio, pypdf, python-docx, pandas

## Screenshots
![Direct question](screenshots/direct-question.png)
![Rewritten question](screenshots/rewritten-question.png)
![Question without an answer](screenshots/no-answer.png)
