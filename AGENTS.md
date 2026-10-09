# Integrated Multi-Tool Student Agent: Architecture & Specification

## Project Overview
The **Integrated Multi-Tool Student Agent** is an end-to-end full-stack agentic AI application designed for academic tasks without requiring an external LLM API key. It uses a deterministic, rule-based, and heuristic NLP agent pipeline that accurately identifies student intent, decomposes requests, selects appropriate tools, executes them safely, evaluates results, and produces clear structured answers with an audit trace.

---

## 7-Stage Core Workflow Pipeline
```
[1. User Query]
       │
       ▼
[2. Intent Detection] (IntentClassifier: Academic Q&A, Document Q&A, Study Planner, Course Queries, Data Analysis, Report Generation, Ambiguous/Fallback)
       │
       ▼
[3. Task Decomposition] (TaskDecomposer: breaks down user goals into concrete subtasks with validation)
       │
       ▼
[4. Tool Selection] (ToolRegistry: maps subtasks to registered tool handlers and validates parameter schemas)
       │
       ▼
[5. Execution] (AgentOrchestrator: safely invokes tool handlers, handles files, isolates errors)
       │
       ▼
[6. Result Evaluation] (ResultEvaluator: verifies completeness, checks data integrity, verifies citations/sources)
       │
       ▼
[7. Final Response] (Unified structured output: answer, metadata, charts, download links, execution trace)
```

---

## Modular Components
1. **`IntentClassifier`**:
   - Classifies query into one of 7 intents: `academic_qa`, `document_qa`, `study_planner`, `course_query`, `data_analysis`, `report_generation`, or `clarification` (ambiguous).
   - Computes intent confidence and extracts domain keywords/entities.

2. **`TaskDecomposer`**:
   - Breaks query into sequential subtasks (e.g. for study planner: parse requirements -> compute study hours -> build schedule; for data analysis: validate CSV -> compute summary statistics -> extract chart series).

3. **`ToolRegistry`**:
   - Registers all 6 tools with descriptions, parameter schemas, input types, and handler bindings.
   - Provides tool discovery and parameter validation.

4. **`AgentOrchestrator`**:
   - Manages stage-by-stage progression.
   - Records execution trace with timestamps, stage status (`success`, `partial`, `failure`, `clarification`), inputs, and outputs.
   - Executes tools securely and handles file uploads (PDF, DOCX, CSV) and sanitization.

5. **`ResultEvaluator`**:
   - Evaluates execution outputs: completeness check, schema validation, citation/source verification.
   - Strictly enforces truthfulness: no hallucinated citations or synthetic facts.

6. **Tool Handlers**:
   - **Academic Q&A Tool**: Query against structured academic knowledge base (CS, Mathematics, Physics, Writing/Citation, Study Skills) with keyword relevance ranking, topic matching, and real citation references.
   - **Document Q&A Tool**: Extracts raw text page-by-page from PDF (PyMuPDF) and section-by-section from DOCX (python-docx). Answers queries via semantic/lexical overlap and provides exact page/paragraph references.
   - **Study Planner Tool**: Schedules subjects based on exam deadlines, difficulty/priority, available daily hours, and rest intervals. Generates day-by-day timetable and recommendations.
   - **Course Catalog Tool**: Searches rich local curriculum dataset with code, name, department, credits, prerequisites, syllabus, and instructor details.
   - **Data Analysis Tool**: Parses CSV files with Pandas, calculates summary statistics (mean, std, quartiles, missing values, skewness, correlations), detects column types, and structures data for Recharts visualization.
   - **Report Generation Tool**: Compiles task findings (study plan, data analysis, course syllabus) into downloadable PDF (ReportLab) and DOCX (python-docx) files.

---

## Backend Endpoints
- `POST /api/agent/run`: Core pipeline execution accepting query, optional tool override, and uploaded file/session context.
- `POST /api/upload`: Handles file uploads (PDF, DOCX, CSV) with validation (size <= 10MB, allowed extensions), temporary storage, and secure cleanup.
- `GET /api/reports/download/{filename}`: Secure download endpoint for generated reports.
- `GET /api/health`: System health and tool registry status.
- `GET /api/courses`: Direct course catalog search and filtering.
- `GET /api/kb/topics`: Academic knowledge base topics list.

---

## Frontend Architecture (React + Vite + Tailwind CSS + Recharts)
- **Dashboard Layout**:
  - Sidebar: Active tool modes, conversation history, sample prompt library.
  - Main Agent Interface: User query bar, file uploader, mode switcher.
  - Visual 7-Stage Execution Trace Drawer: Real-time status cards showing every step of the pipeline.
  - Interactive Result Views:
    - Academic Q&A: Formatted markdown responses with verified references.
    - Document Q&A: Split-view showing question, answer, and extracted document page excerpt citations.
    - Study Planner: Visual calendar/timetable grid, workload distribution metrics.
    - Course Catalog: Filterable course cards with badges for prerequisites and difficulty.
    - Data Analysis: Summary stat cards, missing value badges, interactive Recharts (bar, line, scatter).
    - Report Generator: PDF/DOCX preview and direct download action cards.
