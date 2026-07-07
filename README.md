# Marblebee Automation Suite

End-to-end automation ecosystem built for **Marblebee** — a marble and stone product business selling on Shopify, Etsy, and Houzz. All workflows are built in **n8n** (self-hosted) and connect Google Workspace, Shopify, Google Drive, and social platforms.

---

## What We Built

### 1. Drive → Shopify Image Sync
**`workflows/image-sync/drive_shopify_image_sync.json`**

Runs daily at 9 AM. Scans all 19 product sheets in Google Sheets, finds rows missing images, locates each product's folder in Google Drive, uploads images to Shopify CDN, and writes the CDN URLs back to the sheet.

**How it works:**
- Phase 1: Reads all 19 category sheets and collects products missing `IMAGE URL1`
- Phase 2: For each product, navigates Drive (Root → Category → ProductNo → LOGO) and classifies images by naming convention
- Uploads via Shopify Staged Upload API (multipart GCS POST)
- Writes CDN URLs to `IMAGE URL1`–`IMAGE URL10` and renders a thumbnail in the `Picture` column using `=IMAGE()` formula

**Image naming convention (files must be renamed manually before sync runs):**
- `ProductNo-P.jpg` → Primary image (IMAGE URL1 + Picture thumbnail)
- `ProductNo-S.jpg` → Secondary image (IMAGE URL2)
- All others → IMAGE URL3, URL4, … (in order)
- Products without a `-P` file are skipped until someone renames the files

---

### 2. AI Product Title & Description Generator
**`workflows/product-management/AI Product Title & Description Generator.json`**

Reads product rows from Google Sheets and uses AI to generate Shopify-ready titles, descriptions, SEO titles, SEO descriptions, tags, and product types. Writes results back to the sheet automatically.

---

### 3. Shopify Product Sync Pipeline
**`workflows/product-management/Shopify Merge product sync Pipeline.json`**

Merges and syncs product data from Google Sheets into Shopify — handles product creation, updates, and field mapping across all 19 category sheets.

---

### 4. Wall Coverings Classifier
**`workflows/product-management/wall_coverings_classifier_workflow.json`**

Classifies wall covering products based on attributes and organises them into the correct Shopify collections and categories.

---

### 5. Email Automation
**`workflows/email-automation/`**

Five workflows handling the full inbound and outbound email pipeline:
- **Email Receiving Automation** — processes inbound emails and routes them
- **Email Sending Automation** — sends outbound emails based on triggers
- **Email To Interested** — follow-up emails to interested leads
- **Email To Manager** — escalation notifications to managers
- **Global Error Handler** — catches workflow errors and sends alert emails

---

### 6. Pinterest Publishing Pipeline
**`workflows/pinterest/`**

Six workflows automating the full Pinterest content cycle:
- **product_classifier_workflow** — classifies products for Pinterest boards
- **pinterest_prepare_workflow** — prepares pin content (images, descriptions, links)
- **pinterest_verify_workflow** — verifies content before publishing
- **pinterest_publish_workflow** — publishes pins to the correct boards
- **shopify_blog_n8n_workflow** — syncs Shopify blog posts to Pinterest
- **tailwind_list_boards_helper** — helper for Tailwind board management

---

### 7. Houzz Scraper
**`houzz-scraper/`**

Python scraper that extracts product and contractor listings from Houzz for market research and lead generation. Built with async scraping, a database pipeline, and phone number formatting.

**Stack:** Python, async HTTP, SQLite/PostgreSQL, custom pipeline

---

## Repository Structure

```
marblebee/
├── workflows/
│   ├── image-sync/          # Drive → Shopify image upload automation
│   ├── product-management/  # AI content generation + Shopify sync + classifier
│   ├── email-automation/    # Full email pipeline (receive, send, escalate)
│   └── pinterest/           # Pinterest content publishing pipeline
├── houzz-scraper/           # Python scraper for Houzz listings
└── README.md
```

---

## Setup

### n8n Workflows
1. Import the desired `.json` file into your n8n instance (Settings → Import Workflow)
2. Connect credentials:
   - **Google Drive OAuth2** — for Drive file access
   - **Google Sheets OAuth2** — spreadsheet named `Auto sync shopify 2026`
   - **Shopify OAuth2** — store: `marblebee.myshopify.com`
3. For the Image Sync workflow, toggle the **Active** switch ON to enable the daily 9 AM schedule

### Houzz Scraper
```bash
cd houzz-scraper
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python main.py
```

---

## Credentials

OAuth client secret files (`client_secret_*.json`) are excluded from this repository via `.gitignore`. Store them securely and never commit them to version control.
