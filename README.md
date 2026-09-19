# Marblebee Automation Suite

> A production-grade **n8n automation ecosystem** for e-commerce businesses — connecting Google Drive, Google Sheets, Shopify, Pinterest, and email into a fully automated product management pipeline.

[![n8n](https://img.shields.io/badge/Built%20with-n8n-EA4B71?style=flat-square)](https://n8n.io)
[![Shopify](https://img.shields.io/badge/Integrates-Shopify-96BF48?style=flat-square)](https://shopify.com)
[![Google Drive](https://img.shields.io/badge/Google-Drive%20%7C%20Sheets-4285F4?style=flat-square)](https://drive.google.com)
[![Python](https://img.shields.io/badge/Scraper-Python-3776AB?style=flat-square)](https://python.org)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

---

## Overview

This repository contains the complete automation suite built for **Marblebee** — a marble and natural stone product business selling on Shopify, Etsy, and Houzz. The suite eliminates manual data entry and repetitive tasks across product image management, content generation, email communication, and social media publishing.

**What gets automated:**
- 📸 Google Drive images → Shopify CDN, zero manual uploads
- 🤖 AI-generated product titles, descriptions, and SEO content
- 🛒 Google Sheets product data → Shopify product listings
- 📧 Full inbound and outbound email pipeline, plus warmup
- 📬 AXE AI: Gmail/Outlook conversations logged per client into Drive + Sheets, with an AI/RAG responder
- 📌 Pinterest pin creation, scheduling (via Metricool) and publish verification
- 🕷️ Houzz market data scraping

**Tech stack:** n8n, Google Drive API, Google Sheets API, Shopify GraphQL Admin API, OpenAI, Google Gemini, Microsoft Graph (Outlook), Gmail API, Supabase (pgvector), Cohere reranker, Metricool API, Python

---

## Table of Contents

- [Workflows](#workflows)
  - [1. Google Drive → Shopify Image Sync](#1-google-drive--shopify-image-sync)
  - [2. AI Product Title & Description Generator](#2-ai-product-title--description-generator)
  - [3. Shopify Product Sync Pipeline](#3-shopify-product-sync-pipeline)
  - [4. Wall Coverings Classifier](#4-wall-coverings-classifier)
  - [5. Email Automation Pipeline](#5-email-automation-pipeline)
  - [6. Pinterest Publishing Pipeline](#6-pinterest-publishing-pipeline)
  - [7. AXE AI Email Data Pipeline](#7-axe-ai-email-data-pipeline)
- [Houzz Scraper](#8-houzz-scraper)
- [Repository Structure](#repository-structure)
- [Setup & Installation](#setup--installation)
- [Credentials & Security](#credentials--security)

---

## Workflows

All workflows are built in **n8n** and exported as importable `.json` files. Each can be imported independently into any n8n instance.

---

### 1. Google Drive → Shopify Image Sync

**Files:** `workflows/image-sync/drive_shopify_image_sync.json`, `workflows/image-sync/image_url_health_audit.json`

Automatically syncs product images from Google Drive to Shopify CDN on a daily schedule. No manual uploads needed — just drop images in Drive with the correct naming convention and the workflow handles everything.

**Trigger:** Daily at 9:00 AM (configurable cron schedule)

**How it works:**

| Phase | What happens |
|-------|-------------|
| **Phase 1 — Collect** | Scans all 30 Google Sheets product tabs, collects rows where `IMAGE URL1` is empty |
| **Phase 2 — Process** | For each product, finds its folder in Drive (Root → Category → ProductNo → LOGO), classifies images, uploads to Shopify CDN via Staged Upload API, writes CDN URLs back to the sheet |

**Image naming convention:**

| Filename pattern | Result |
|-----------------|--------|
| `ProductNo-P.jpg` or `ProductNo_P.jpg` | Primary image → `IMAGE URL1` + Picture thumbnail |
| `ProductNo-S.jpg` or `ProductNo_S.jpg` | Secondary image → `IMAGE URL2` |
| Any other image | `IMAGE URL3`, `IMAGE URL4` … in order |
| No `-P` file found | Row is skipped until someone renames a file |

> Naming is case-insensitive: `-P`, `-p`, `_P`, `_p` all work.

**Shopify upload flow:**
1. `stagedUploadsCreate` GraphQL mutation → gets GCS signed URL
2. Multipart form POST to Google Cloud Storage
3. `fileCreate` GraphQL mutation → registers file in Shopify Files
4. 15s wait → `nodes` query → retrieves final CDN URL
5. Writes `=IMAGE("cdn_url", 1)` formula to the `Picture` column for thumbnail preview

**Safety features:**
- **Self-healing:** existing `IMAGE URL1-10` values are HEAD-checked on the Shopify CDN; dead links are re-checked after 45s (CDN propagation delay) and only re-queued for a fresh upload from Drive if still dead
- **Upload safety:** oversized images are downscaled before upload, and URLs are only written once the file is READY and the exact URL is confirmed live on the CDN

**Image URL Health Audit** (`image_url_health_audit.json`): manual, report-only workflow that scans every uploaded `IMAGE URL1-11` across all 30 sheets and logs dead links (404/410) to an `Image Health Report` tab. It never writes to image columns or re-queues rows.

**Supported categories (30 sheets):**
2026Instock, Animal, Balustrade, Bathtub, Bench, BookMatchedSlabs, Carving, Columns, DoorSurround, Exterior-Wall-Decoration, Fireplace, Fountain, Lamppost, Marble+ Basin, Marble+ ConsoleTable, MarbleMedallion, MarbleRangeHood, MarbleSlab, Moulding, OutdoorCornice, Planter, Sink, Stairs, Statue, StoneMosaicTile, Table, Vanity, Wainscoting, Wall-Coverings, WindowSurround

---

### 2. AI Product Title & Description Generator

**File:** `workflows/product-management/AI Product Title & Description Generator.json`

Reads product rows from Google Sheets and uses AI to generate complete Shopify-ready content — automatically written back to the sheet.

**Generates:**
- Shopify product title
- Shopify product description (HTML-ready)
- SEO meta title and SEO meta description
- Product tags
- Product type
- Etsy-optimised title and description

---

### 3. Shopify Product Sync Pipeline

**Files:**
- `workflows/product-management/Shopify Merge product sync Pipeline (GraphQL).json` — main pipeline
- `workflows/product-management/Shopify Merge product sync Pipeline (GraphQL) - Process Product (sub-workflow).json` — per-product sub-workflow

Syncs product data from Google Sheets into Shopify via the **GraphQL Admin API** across all 30 category sheets. Handles product create/update, variants, slot-based image sync, collection assignments, metafields, SEO fields and Online Store publishing.

**Sub-workflow architecture:** the main pipeline loops over sheets and reads/filters rows, then calls the Process Product sub-workflow once per product via an *Execute Sub-workflow* node (*Run once for each item*). This sidesteps an n8n Split-In-Batches bug where a node triggered more than once per execution silently stops processing items. A dedupe step guards against the sheet loop's *Done* branch firing more than once.

**Also handled by the main pipeline:**
- **CAD → IMAGE URL11 sync:** copies `CAD Drawing URL` into `IMAGE URL11` (skipped if that exact URL already sits in another image slot)
- **Publish reconciliation:** recently created products missing from the Online Store channel are re-published

**Import note:** import both files, then open the main pipeline's `Process Product (per item)` node and select the sub-workflow from its dropdown.

---

### 4. Wall Coverings Classifier

**File:** `workflows/product-management/wall_coverings_classifier_workflow.json`

Classifies wall covering product images with Gemini into a 4-category system, generates a descriptive panel name, and fills the matching Shopify collection IDs in the sheet.

---

### 5. Email Automation Pipeline

**Directory:** `workflows/email-automation/`

Interconnected n8n workflows covering the complete email lifecycle:

| Workflow | Purpose |
|---------|---------|
| `Email Receiving Automation` | Processes inbound emails, parses content, routes to correct handler |
| `Email Sending Automation` | Sends outbound emails based on workflow triggers |
| `Email To Interested` | Automated follow-up sequence for interested leads |
| `Email To Manager` | Escalation notifications with context to managers |
| `Email Warmup` | Runs at 9 AM and 3 PM, round-robining warmup templates across 13 sending workspaces |
| `Global Error Handler` *(in `workflows/shared/`)* | Catches errors across **all** workflows system-wide and sends alert emails |

---

### 6. Pinterest Publishing Pipeline

**Directory:** `workflows/pinterest/`

Workflows automating the Pinterest content cycle from product/blog classification to a verified live pin. Pins are scheduled through the **Metricool API**.

| Workflow | Purpose |
|---------|---------|
| `product_classifier_workflow` | AI-classifies products by image into Category + Subcategory |
| `pinterest_product_photo_prepare_workflow` *(WF0)* | Scans the Drive product-photo folder, uploads new images to Shopify Files, AI-writes pin copy, and queues pins linking to the Shopify product page |
| `pinterest_prepare_workflow` *(WF1)* | Reads newly posted blogs, extracts images, plans pins, AI-writes copy and auto-assigns boards |
| `pinterest_publish_workflow` *(WF2)* | Schedules approved pins from both pin tabs in Metricool, with retry and reschedule handling |
| `pinterest_verify_workflow` *(WF3)* | Polls Metricool and writes back the live Pinterest URL once a pin is published |
| `shopify_blog_n8n_workflow` | Publishes blog posts to Shopify |
| `metricool_list_boards_helper` | Pulls Pinterest board names/IDs from Metricool into the boards tab |

---

### 7. AXE AI Email Data Pipeline

**Directory:** `workflows/axe-ai/`

Logs Gmail and Outlook conversations per client into Drive + Sheets, and answers questions over that data with RAG.

| Workflow | Purpose |
|---------|---------|
| `email-ingestion/Email Intake Orchestrator` | Reads accounts from a control sheet, searches Gmail/Outlook, reuses or creates each client's Drive folder, attachments subfolder and conversation sheet, and queues message IDs (with a `last_synced_at` watermark) |
| `message-processors/Gmail Processor` | Every 10 min: reads queued Gmail messages, extracts the latest message text, uploads attachments to Drive, and appends a row to the client conversation sheet |
| `message-processors/Outlook Processor` | Same as above for Outlook (Microsoft Graph) |
| `message-processors/Sort All Conversation Sheets by Timestamp` | Manual utility that sorts every client conversation sheet by timestamp |
| `ai-responder/Email Receiver and Replier` | Monitors mailboxes, stores cases/messages/files in Supabase with embeddings, and drafts RAG-based replies |
| `rag/RAG Query Interface` | Webhook Q&A endpoint over the Supabase vector store with Cohere reranking |

Flow: Intake Orchestrator → queue sheet → Gmail/Outlook Processors → per-client conversation sheets and Drive attachment folders.

---

### 8. Houzz Scraper

**Directory:** `houzz-scraper/` *(linked as git submodule → [bilalhaider11/houzz_scrapper](https://github.com/bilalhaider11/houzz_scrapper))*

Async Python scraper that extracts product listings and contractor profiles from Houzz for market research and lead generation.

**Features:**
- Async HTTP with rate limiting and retry logic
- Database pipeline (SQLite / PostgreSQL)
- Phone number formatting and deduplication
- Configurable scraping state (resume from last position)

**Stack:** Python 3, aiohttp, SQLAlchemy, custom pipeline

---

## Repository Structure

```
marblebee/
├── workflows/
│   ├── axe-ai/
│   │   ├── ai-responder/
│   │   │   └── Email Receiver and Replier.json
│   │   ├── email-ingestion/
│   │   │   └── Email Intake Orchestrator.json
│   │   ├── message-processors/
│   │   │   ├── Gmail Processor.json
│   │   │   ├── Outlook Processor.json
│   │   │   └── Sort All Conversation Sheets by Timestamp.json
│   │   └── rag/
│   │       └── RAG Query Interface.json
│   ├── image-sync/
│   │   ├── drive_shopify_image_sync.json           # Google Drive → Shopify image sync
│   │   └── image_url_health_audit.json             # Report-only dead image link audit
│   ├── product-management/
│   │   ├── AI Product Title & Description Generator.json
│   │   ├── Shopify Merge product sync Pipeline (GraphQL).json
│   │   ├── Shopify Merge product sync Pipeline (GraphQL) - Process Product (sub-workflow).json
│   │   └── wall_coverings_classifier_workflow.json
│   ├── shared/
│   │   └── Global Error Handler.json               # System-wide — covers ALL workflows
│   ├── email-automation/
│   │   ├── Email Receiving Automation.json
│   │   ├── Email Sending Automation.json
│   │   ├── Email To Interested.json
│   │   ├── Email To Manager.json
│   │   ├── Email Warmup.json
│   │   └── README.md
│   └── pinterest/
│       ├── metricool_list_boards_helper.json
│       ├── pinterest_prepare_workflow.json
│       ├── pinterest_product_photo_prepare_workflow.json
│       ├── pinterest_publish_workflow.json
│       ├── pinterest_verify_workflow.json
│       ├── product_classifier_workflow.json
│       └── shopify_blog_n8n_workflow.json
├── houzz-scraper/                                   # Submodule → houzz_scrapper repo
├── .gitignore
└── README.md
```

---

## Setup & Installation

### Prerequisites

- [n8n](https://n8n.io) (self-hosted, v1.0+)
- Google Cloud project with Drive and Sheets APIs enabled
- Shopify store with Admin API access
- Python 3.9+ (for Houzz scraper only)

### Importing n8n Workflows

1. Open your n8n instance
2. Go to **Workflows → Import from file**
3. Select any `.json` file from the `workflows/` directory
4. Connect the required credentials (see below)
5. For scheduled workflows, toggle the **Active** switch ON

### Required Credentials

| Credential | Used by |
|-----------|---------|
| Google Drive OAuth2 | Image Sync, all Drive lookups |
| Google Sheets OAuth2 | All sheet read/write operations |
| Shopify OAuth2 | Image Sync, Product Sync |
| OpenAI API | AI Title & Description Generator, Pinterest, AXE AI |
| Google Gemini API | Wall Coverings Classifier, AXE AI responder |
| Metricool (Header Auth, `X-Mc-Auth`) | Pinterest Publishing Pipeline |
| Gmail OAuth2 / SMTP | Email Automation, AXE AI |
| Microsoft Outlook OAuth2 | AXE AI Outlook ingestion and processor |
| Supabase / Cohere | AXE AI responder and RAG |

### Houzz Scraper

```bash
cd houzz-scraper
python -m venv venv
source venv/bin/activate       # Windows: venv\Scripts\activate
pip install -r requirements.txt
python main.py
```

---

## Credentials & Security

OAuth client secret files (`client_secret_*.json`) are excluded from this repository via `.gitignore` and must never be committed to version control. Download them from Google Cloud Console and place them in the appropriate workflow directory locally.

---

## License

MIT — free to use, adapt, and build upon.
