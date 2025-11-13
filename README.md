<<<<<<< HEAD
# Vacature Skill Mapping

A project that scrapes job postings, extracts skills and maps them to a taxonomy.

Table of Contents

- [Overview](#overview)
- [Repository layout](#repository-layout)
- [Quick start (development)](#quick-start-development)
- [Architecture](#architecture)
- [Full project tree](#full-project-tree)
- [Notes & troubleshooting](#notes--troubleshooting)
- [Contributing & License](#contributing--license)

## Overview

This repository contains:
- a Python backend with scrapers and an API,
- a React + Vite frontend for viewing and exporting results,
- a small NLP pipeline to normalize and extract skills from job descriptions.

Use the backend CLI to populate the database and run the scrapers, then use the frontend to explore results or run the NLP pipeline to extract skills.

## Repository layout

- `backend/` — Python API, scrapers and database models.
  - `backend/app/run.py` — CLI (Typer) to initialize DB and run scrapers.
  - `backend/scrapper/` — scraper implementations (Stepstone, Indeed, writers).
  - `backend/db/` — SQLAlchemy models, seed data and SQL.
- `frontend/` — Vite + React TypeScript app (UI for searching, exporting, and viewing results).
- `job_skill_pipeline/` — NLP code for normalizing skills and running the pipeline.
- `data_scrapping/` — example scraping scripts and sample outputs.
- `initdb/` — SQL used for initial DB setup.
- `docker-compose.yaml` — service definition to run backend + frontend with containers.

## Quick start (development)

Prerequisites
- Python 3.10+ (recommend using a virtualenv)
- Node.js 16+ and npm or yarn (for the frontend)
- (Optional) Docker & Docker Compose — to run services in containers

Backend (local)

1. Create and activate a virtual environment, then install dependencies:

```cmd
python -m venv .venv
.venv\Scripts\activate
pip install -r backend/requirements.txt
```

2. (Optional) Create a `.env` file in `backend/` if you want to override DB connection or other settings.

3. Initialize the database tables:

```cmd
python backend/app/run.py initdb
```

4. Run the scraper (examples):

```cmd
python backend/app/run.py scrape stepstone --query "data engineer" --location "Belgium" --pages 2
python backend/app/run.py scrape indeed --query "data scientist" --pages 1
```

Frontend (local)

1. Install dependencies and start dev server:

```cmd
cd frontend
npm install
npm run dev
```

2. Open the URL shown by Vite (typically `http://localhost:5173`).

Docker (optional)

```cmd
docker-compose up --build
```

Job skill pipeline

Run the pipeline:

```cmd
python job_skill_pipeline/run_pipeline.py
```

## Architecture

```
   [ CLI / Scheduler ]
	   |
	   v
   +-----------------+          +----------------+          +--------------------------+          +-------------+
   |   Scrapers      |  ---->   |   Database     |  ---->   |   Job Skill Pipeline     |  ---->   |  Frontend   |
   | (Stepstone,     |          | (SQLAlchemy)   |          | (normalize & extract)    |          | (React/Vite)|
   |  Indeed, writers)|          |  backend/app/db|          |  job_skill_pipeline/     |          |  frontend/  |
   +-----------------+          +----------------+          +--------------------------+          +-------------+
	   |                          ^   |
	   |                          |   | (reads/writes results)
	   +--> (writer_csv / writer) +---+
```

## Full project tree

```
.
├── .gitmodules
├── README.md
├── docker-compose.yaml
├── requirements.txt
├── initdb/
│   └── 01_init_.sql
├── data_scrapping/
│   ├── vdab_1.py
│   ├── vdabvdab.py
│   ├── vdab_jobs_playwright_full_data.csv
│   ├── vdab_jobs_playwright_full_data.json
│   ├── vdab_jobs_playwright_preview_New5.csv
│   └── vdab_jobs_playwright_preview_New5.json
├── backend/
│   ├── Dockerfile
│   ├── requirments.txt
│   └── app/
│       ├── run.py                      # Typer CLI: initdb, scrape
│       ├── api/
│       │   ├── routes_chatbot.py
│       │   ├── routes_export.py
│       │   └── routes_jobs.py
│       ├── core/
│       │   ├── config.py
│       │   └── utils.py
│       ├── db/
│       │   ├── database.py
│       │   ├── datawarehouse.sql
│       │   ├── models.py
│       │   └── seed_data.py
│       └── scrapper/                    # scrapers and writers
│           ├── __init__.py
│           ├── base.py
│           ├── indeed.py
│           ├── normalizer.py
│           ├── stepstone.py
│           ├── writer.py
│           └── writer_csv.py
├── frontend/
│   ├── README.md
│   ├── index.html
│   ├── package.json
│   ├── package-lock.json
│   ├── vite.config.ts
│   ├── tsconfig.app.json
│   ├── tsconfig.node.json
│   ├── tailwind.config.js
│   ├── eslint.config.js
│   ├── public/
│   │   └── vite.svg
│   └── src/
│       ├── index.html
│       ├── main.tsx
│       ├── index.css
│       ├── App.tsx
│       ├── App.css
│       ├── Mobile.tsx
│       ├── Chatbot.tsx
│       ├── Dashboard.tsx
│       ├── Export.tsx
│       ├── pages/
│       │   ├── Assistant.tsx
│       │   ├── Dashboard.tsx
│       │   ├── export_model.tsx
│       │   ├── Landing.tsx
│       │   └── mobile_view.tsx
│       ├── components/
│       │   ├── Landing/
│       │   │   ├── ArrowMotif.tsx
│       │   │   ├── Hero.tsx
│       │   │   └── QuickSearch.tsx
│       │   ├── layout/
│       │   │   ├── Header.tsx
│       │   │   └── NavBar.tsx
│       │   └── UI/
│       │       ├── Button.tsx
│       │       ├── CeforaLogo.tsx
│       │       └── SearchFilter.tsx
│       ├── i18n/
│       │   └── LanguageProvider.tsx
│       ├── assets/
│       │   ├── CEVORA-CEFORA_Logo_white_RGB.png
│       │   ├── CEVORA-CEFORA_Logo+baseline_RGB.png
│       │   └── CEVORA-CEFORA_Logo+baseline_RGB.svg
│       └── postcss.config.js
└── job_skill_pipeline/
    ├── requirements.txt
    ├── run_pipeline.py
    ├── pipeline/
    │   └── __init__.py
    └── nlp/
	├── normalize_skills.py
	├── placeholders.py
	├── skill_extractor.py
	└── taxonomy_loader.py

```

## Notes & troubleshooting

- Watch for a `scraper` vs `scrapper` naming mismatch in imports if you see ModuleNotFoundError. Adjust `PYTHONPATH` or rename accordingly.
- If the frontend cannot reach the backend, confirm the backend is running and check CORS/proxy settings in `frontend/vite.config.ts`.

## Contributing & License

Contributions welcome — open issues or PRs. Consider adding a `LICENSE` file (e.g., MIT) if you plan to publish.

---
If you'd like I can also:
- add per-folder `README.md` files with focused instructions,
- add a `CONTRIBUTING.md` and `LICENSE`, or
- create a small SVG architecture diagram and commit it to the repo.
=======

>>>>>>> 9e3396d4a3625ae3496287bdf8c97e025b638373

