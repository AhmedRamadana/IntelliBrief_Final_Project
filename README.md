# IntelliBrief — Multi-Agent Business Intelligence & Reporting System

**Final Project — AI Engineering Course**

A fully functional multi-agent AI system that automates business intelligence reporting end-to-end, using **real public datasets**: it gathers data from three real sources, analyzes it with an LLM, writes a professional report, validates it for reliability, and automates its delivery — all through a simple web chat/dashboard interface.

---

## 📋 Project Overview

**IntelliBrief** simulates a real-world business workflow: instead of an analyst manually pulling sales numbers, reading a market report, and checking customer feedback to write a report, this system does it automatically through a coordinated team of specialized AI agents — grounded in **real, downloadable data**, not synthetic examples.

Give it a topic (e.g. *"Q3 Sales Performance"*) and, optionally, the market report PDF — it will:

1. **Retrieve** data from three real sources (a retail sales dataset, a customer reviews dataset, and RAG-based retrieval from an industry report PDF)
2. **Analyze** the combined data using an LLM, producing structured, validated output (not free text)
3. **Write** a polished, professional Markdown report
4. **Validate** the report for reliability before anything is sent (a Guardrail/QA layer)
5. **Automate** the delivery: save the report, send it by email (simulated), and log it to a dashboard
6. Let the user **chat** with the latest report for follow-up questions, using conversation memory

---

## 📊 Data Sources

All three sources are real, publicly available, and were chosen so they relate to a single coherent business scenario: a retail company assessing its performance in the context of real customer sentiment and real fashion/retail industry trends.

| # | Data Source | Type | Download Link |
|---|---|---|---|
| 1 | **Superstore Sales Dataset** — thousands of real retail order records (2015–2018): revenue, quantity, category, region | CSV | https://www.kaggle.com/datasets/vivek468/superstore-dataset-final |
| 2 | **Women's E-Commerce Clothing Reviews** — ~23,000 real customer reviews with ratings (1–5) and free text | CSV | https://www.kaggle.com/datasets/nicapotato/womens-ecommerce-clothing-reviews |
| 3 | **The State of Fashion 2022** — a real ~130-page industry report by McKinsey & Business of Fashion | PDF | https://www.mckinsey.com/~/media/mckinsey/industries/retail/our%20insights/state%20of%20fashion/2022/the-state-of-fashion-2022.pdf |

### How to set them up
1. Download the two CSV files from Kaggle (requires a free Kaggle account) and the PDF from the McKinsey link above.
2. Upload the CSVs to your Colab/Kaggle session (e.g. to `/content/`).
3. Update `SALES_CSV_PATH` and `FEEDBACK_CSV_PATH` in **Sections 6 and 8** of the notebook to match your uploaded file names.
4. The PDF does **not** need a fixed path — it is uploaded directly through the Gradio file-upload component in the web interface.

---

## 🎯 Why This Project

This project was designed specifically to demonstrate, in one coherent system, everything covered across the course:

| Course Topic | Where It's Used in This Project |
|---|---|
| **Output Parser** | The Analyst Agent returns strictly structured JSON (summary, insights, risks, recommendation) via LangChain's `StructuredOutputParser` |
| **AI Agent with Tools** | Every agent calls API-like tool functions (`fetch_sales_data_api`, `fetch_feedback_sheet_api`, save/send/log APIs) |
| **RAG** | The market report PDF is chunked, embedded, and retrieved via FAISS |
| **LangChain** | `PyPDFLoader`, `FAISS`, `HuggingFaceEmbeddings`, `PromptTemplate`, `StructuredOutputParser` |
| **Memory** | Follow-up Q&A chat about the latest report uses a bounded conversation memory |
| **Multi-Agent** | Four specialized, single-responsibility agents (Retriever, Analyst, Writer, QA) coordinated by an orchestrator |
| **Automation & APIs** | `save_report_api`, `send_email_api`, `update_dashboard_api` — with error handling and a QA gate before sending |
| **Web Interface** | Gradio app with two tabs: run analysis, and chat about results |
| **Quantization & Deployment** | Model loaded in 4-bit (NF4 + double quantization); deployable via Streamlit + Ngrok as an alternative front end |

---

## 🤔 The Problem It Solves

Manually producing a business report requires:
- Digging through raw sales numbers
- Reading long market reports for relevant sections
- Reviewing scattered customer feedback
- Writing a coherent summary
- Double-checking the summary is actually accurate
- Manually sending it and updating tracking sheets

This is slow, repetitive, and error-prone when done every week/month for multiple topics. **IntelliBrief automates the entire pipeline** on top of **real data**, while keeping a **human-auditable trail** (saved report file, dashboard log, and a QA pass/fail status) — so automation doesn't mean losing control over quality.

---

## 🏗️ Architecture

```
                    +----------------------+
                    |     Web Interface     |
                    |  (Gradio: Run + Chat)  |
                    +-----------+----------+
                                |
                                v
                    +----------------------+
                    |      Orchestrator      |
                    |     (run_pipeline)      |
                    +-----------+----------+
                                |
     +-------------+------------+------------+-------------+
     v             v            v            v             v
Retriever      Analyst      Writer        QA        Automation
  Agent         Agent        Agent      Agent        (save/send/log)
     |             |            |          |             |
     v             v            |          |             |
 Sales CSV     Structured       |     Validates    save_report_api
 Feedback CSV   JSON Output     |     analysis &   send_email_api
 RAG (PDF)     (Output Parser)  |     report        update_dashboard_api
                                v
                          Markdown Report
                                |
                                v
                    Memory (chat follow-up Q&A)
```

For the full data-flow diagram and design rationale, see **Section 22 (Architecture)** at the end of the notebook, and the detailed walkthrough in `شرح_مشروع_IntelliBrief.md`.

---

## 📂 Project Files

| File | Description |
|---|---|
| `IntelliBrief_MultiAgent_System.ipynb` | Main notebook — full implementation, runnable end-to-end in Colab/Kaggle, split into 22 clearly commented sections |
| `README.md` | This file — project overview |
| `شرح_مشروع_IntelliBrief.md` | Detailed, line-by-line explanation of every part of the code (in Arabic) |

---

## 🚀 How to Run

1. Download the two CSVs and the PDF from the links in the **Data Sources** section above.
2. Open `IntelliBrief_MultiAgent_System.ipynb` in Google Colab or Kaggle (GPU runtime recommended).
3. Upload the two CSV files and update the paths in Sections 6 and 8.
4. Run all cells from top to bottom.
5. The second-to-last cell launches a Gradio app with a public link (`share=True`).
6. In the **"Run New Analysis"** tab: enter a topic (e.g. `Q3 Sales Performance`), upload the State of Fashion PDF, and click **Run**.
7. In the **"Chat About Latest Report"** tab: ask follow-up questions about the generated report.
8. Generated reports are saved as `report_*.md` files, and every run is logged to `dashboard.csv`.

### Optional: Production-style Deployment (Streamlit + Ngrok)
The core agent functions in this notebook (`run_pipeline`, `chat_with_report`, etc.) can be reused inside a Streamlit `app.py` and exposed publicly with Ngrok, exactly as covered in the Quantization & Deployment session — a natural next step for turning this from a demo into a small internal tool.

---

## 🧪 Testing & Evaluation

The system includes a built-in reliability layer:
- **QA Agent**: checks that all required fields are present, non-empty, and correctly typed before allowing the email step to run.
- **Guarded automation**: if QA fails, the report is still saved and logged (for auditing), but the email send is skipped — preventing unreliable content from reaching stakeholders.
- **Dashboard log** (`dashboard.csv`): every run's topic, insight/risk counts, and QA pass/fail status are recorded, enabling manual review of the system's reliability over time.
- **Independent data-source failure handling**: each of the 3 sources (sales CSV, feedback CSV, PDF) can be missing or fail to load without crashing the whole pipeline — mirroring real-world "one system is down" situations.

---

## 🔮 Future Improvements

- Replace the simulated email step with a real email API (e.g. SendGrid) and the sales CSV with a live REST API or database connection
- Add a fine-tuned model (Session 3: FineTuning) specialized for business report writing style
- Add retry-with-corrected-prompt logic when the QA Agent fails, instead of just skipping the email
- Persist chat memory and past reports across sessions using a database
- Filter the sales dataset by a specific product category once a clothing/apparel-specific internal sales dataset is available, for an even tighter thematic match with the reviews and market report
