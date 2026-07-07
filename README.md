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
- 📧 Full inbound and outbound email pipeline
- 📌 Pinterest pin creation and publishing
- 🕷️ Houzz market data scraping

**Tech stack:** n8n (self-hosted), Google Drive API, Google Sheets API, Shopify GraphQL Admin API, OpenAI, Pinterest API, Python

---

## Table of Contents

- [Workflows](#workflows)
  - [1. Google Drive → Shopify Image Sync](#1-google-drive--shopify-image-sync)
  - [2. AI Product Title & Description Generator](#2-ai-product-title--description-generator)
  - [3. Shopify Product Sync Pipeline](#3-shopify-product-sync-pipeline)
  - [4. Wall Coverings Classifier](#4-wall-coverings-classifier)
  - [5. Email Automation Pipeline](#5-email-automation-pipeline)
  - [6. Pinterest Publishing Pipeline](#6-pinterest-publishing-pipeline)
- [Houzz Scraper](#7-houzz-scraper)
- [Repository Structure](#repository-structure)
- [Setup & Installation](#setup--installation)
- [Credentials & Security](#credentials--security)

---

## Workflows

All workflows are built in **n8n** and exported as importable `.json` files. Each can be imported independently into any n8n instance.

---

### 1. Google Drive → Shopify Image Sync

**File:** `workflows/image-sync/drive_shopify_image_sync.json`

Automatically syncs product images from Google Drive to Shopify CDN on a daily schedule. No manual uploads needed — just drop images in Drive with the correct naming convention and the workflow handles everything.

**Trigger:** Daily at 9:00 AM (configurable cron schedule)

**How it works:**

| Phase | What happens |
|-------|-------------|
| **Phase 1 — Collect** | Scans all 19 Google Sheets product tabs, collects rows where `IMAGE URL1` is empty |
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

**Supported categories (19 sheets):**
Animal, Bench, Planter, Fountain, Lamppost, Bathtub, Sink, Vanity, Fireplace, Table, Statue, MarbleSlab, 2026Instock, DoorSurround, Wall-Coverings, MarbleRangeHood, Moulding, StoneMosaicTile, MarbleMedallion

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

**File:** `workflows/product-management/Shopify Merge product sync Pipeline.json`

Syncs product data from Google Sheets directly into Shopify. Handles product creation, field updates, collection assignments, and metafield mapping across all 19 category sheets.

---

### 4. Wall Coverings Classifier

**File:** `workflows/product-management/wall_coverings_classifier_workflow.json`

Classifies wall covering products by material, dimensions, and style attributes, then assigns them to the correct Shopify collections and categories automatically.

---

### 5. Email Automation Pipeline

**Directory:** `workflows/email-automation/`

Five interconnected n8n workflows covering the complete email lifecycle:

| Workflow | Purpose |
|---------|---------|
| `Email Receiving Automation` | Processes inbound emails, parses content, routes to correct handler |
| `Email Sending Automation` | Sends outbound emails based on workflow triggers |
| `Email To Interested` | Automated follow-up sequence for interested leads |
| `Email To Manager` | Escalation notifications with context to managers |
| `Global Error Handler` | Catches errors across all workflows and sends alert emails |

---

### 6. Pinterest Publishing Pipeline

**Directory:** `workflows/pinterest/`

Six workflows automating the full Pinterest content cycle from product classification to live pin:

| Workflow | Purpose |
|---------|---------|
| `product_classifier_workflow` | Classifies products and maps them to Pinterest boards |
| `pinterest_prepare_workflow` | Prepares pin content — image, title, description, destination URL |
| `pinterest_verify_workflow` | Validates content against Pinterest guidelines before publishing |
| `pinterest_publish_workflow` | Publishes approved pins to the correct boards |
| `shopify_blog_n8n_workflow` | Syncs Shopify blog posts as Pinterest idea pins |
| `tailwind_list_boards_helper` | Helper workflow for Tailwind board management |

---

### 7. Houzz Scraper

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
│   ├── image-sync/
│   │   └── drive_shopify_image_sync.json       # Google Drive → Shopify image sync
│   ├── product-management/
│   │   ├── AI Product Title & Description Generator.json
│   │   ├── Shopify Merge product sync Pipeline.json
│   │   └── wall_coverings_classifier_workflow.json
│   ├── email-automation/
│   │   ├── Email Receiving Automation.json
│   │   ├── Email Sending Automation.json
│   │   ├── Email To Interested.json
│   │   ├── Email To Manager.json
│   │   └── Global Error Handler.json
│   └── pinterest/
│       ├── pinterest_prepare_workflow.json
│       ├── pinterest_publish_workflow.json
│       ├── pinterest_verify_workflow.json
│       ├── product_classifier_workflow.json
│       ├── shopify_blog_n8n_workflow.json
│       └── tailwind_list_boards_helper.json
├── houzz-scraper/                               # Submodule → houzz_scrapper repo
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
| OpenAI API | AI Title & Description Generator |
| Pinterest OAuth2 | Pinterest Publishing Pipeline |
| Gmail / SMTP | Email Automation workflows |

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
