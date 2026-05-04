# API & Integration Reference: IDA AI-Powered SEO Engine

This document provides a comprehensive reference for the API and integration points within the IDA AI-Powered SEO Engine project. It details the interactions between Statamic, n8n, Supabase, and Claude, with a specific focus on data handling and workflow design.

## 1. Project Overview and System Architecture

The IDA AI-Powered SEO Engine aims to transform manual content management into an automated, AI-driven process for scaling SEO auditing, internal linking, and content freshness across IDA's country-specific page grid. The system operates as a continuous loop, where n8n orchestrates data flow between Statamic (content source), Supabase (intelligence layer), and Claude (AI for content optimization).

**Core Stack:**

*   **n8n:** Automation and workflow orchestration (The "Nervous System")
*   **Supabase:** Vector database for storing content and metadata (The "Brain")
*   **Statamic:** Content Management System (CMS) and source of truth (The "Body")
*   **Claude (AI):** Semantic analysis and content drafting (The "Editor")

## 2. Statamic Private API Integration

The Statamic Private API serves as the primary interface for retrieving and updating content within the CMS. All interactions require secure authentication.

### 2.1 Authentication

*   **Method:** Bearer Token authentication.

### 2.2 Data Retrieval (Ingestion)

*   **Endpoint:** `GET /api/private/collections/countries/entries`
*   **Locale Filtering:** The API may return entries for all locales regardless of the `?site=` parameter. n8n workflows **MUST** implement a filter node to retain only entries matching the target locale (e.g., `locale === 'en'`).
*   **Defensive Extractor:** Fields can be located either at the top-level of the JSON response or nested within a `data` object. n8n expressions **MUST** account for both possibilities, such as `{{ $json.slug || $json.data.slug }}`.

### 2.3 Data Updates (Optimization)

*   **Method:** `PATCH` requests are used to update existing entries.
*   **Slug Guard (Critical Maintenance):** When performing `PATCH` requests, it is **CRITICAL** to include the `slug` field in the request body. Failure to do so will result in Statamic deleting the page URL. n8n workflows **MUST** fetch the current slug and "echo" it back in the update body to prevent unintended URL deletion.

## 3. n8n Workflows

n8n acts as the central orchestrator, managing data ingestion, processing, and optimization workflows.

### 3.1 Phase 0: The Intelligence Indexer (Ingestion)

*   **Goal:** Populate Supabase with all country page data from Statamic.
*   **Key Steps:**
    1.  **Fetcher:** Call `GET /api/private/collections/countries/entries` with Bearer Token authentication.
    2.  **Locale Filter Node:** Filter entries to keep only the target locale (e.g., `locale === 'en'`).
    3.  **Defensive Extractor:** Use expressions like `{{ $json.slug || $json.data.slug }}` to reliably extract fields.
    4.  **Content Chunking:** Utilize the Recursive Character Text Splitter node to break down ProseMirror JSON content into smaller, manageable chunks suitable for AI processing.
    5.  **Vector Storage:** Store the processed text and its mathematical "Embedding" into the Supabase Vector Store using the dedicated node.

### 3.2 Phase 1: Structural & H1 Audit

*   **Goal:** Identify and flag structural weaknesses (e.g., missing or incorrect H1s).
*   **Trigger:** Scheduled weekly run in n8n.
*   **Logic:**
    1.  Scan the `page_content` column in Supabase.
    2.  Count heading nodes where `attrs.level: 1`.
    3.  If the H1 count is not equal to 1 (`H1 count ≠ 1`), flag the corresponding row in Supabase as "Action Required".
    4.  Send a notification to a designated review queue.

### 3.3 Phase 2: Meta & Content Optimizer

*   **Goal:** Automate the rewriting of Meta Titles and Descriptions based on competitor data.
*   **Key Steps:**
    1.  **Data Sourcing:** Fetch the top 3 SERP competitors via DataForSEO.
    2.  **Analysis:** Send the current `meta_title` and competitor data to Claude for analysis.
    3.  **Drafting:** Claude generates a new Meta Title and Description.
    4.  **Slug Guard:** Before updating Statamic, fetch the current slug and include it in the `PATCH` request body to prevent URL deletion.

## 4. Supabase Vector Store Integration

Supabase serves as the project's "Brain," centralizing site intelligence by storing vectorized content and metadata.

*   **Purpose:** Store page content and SEO metadata as searchable vectors.
*   **Ingestion:** Populated via n8n workflows (Phase 0) after content chunking and embedding generation.
*   **Auditing:** Used as the data source for structural audits (Phase 1).

## 5. Claude Integration

Claude functions as the "Editor," providing AI capabilities for semantic analysis and content drafting.

*   **Role:** Analyzes current metadata and competitor data to draft optimized Meta Titles and Descriptions.
*   **Workflow:** Integrated into n8n workflows (Phase 2) for automated content optimization.

## 6. Handling Specific Data Types

Precise handling of Statamic's custom field types is essential for data integrity.

### 6.1 Select Fields

*   **API Representation:** The Statamic API returns Select Fields (e.g., "Continent") as objects.
*   **n8n Writeback:** When writing back to Statamic via n8n (`PATCH` requests), **ONLY** the raw string key (e.g., `"south-america"`) should be sent, not the full object.

### 6.2 Bard Fields

*   **Storage Format:** Bard Fields store content as ProseMirror JSON.
*   **Manipulation:** n8n workflows **MUST** manipulate the JSON nodes directly rather than converting the content to raw HTML. This ensures that Statamic’s formatting and rich text capabilities remain intact.

## 7. Update Safety: The "Drift" Check

To prevent unintended data changes or corruption, every n8n workflow that writes back to Statamic **MUST** implement a Pre/Post Verification Pattern:

1.  **Capture Pre-State:** Record the state of critical fields (e.g., `title`, `slug`) before executing the `PATCH` request.
2.  **Perform PATCH:** Execute the update operation to Statamic.
3.  **Compare Post-State:** Compare the response from the `PATCH` request with the captured pre-state to ensure that the `slug` and other guarded fields did not change unexpectedly.

## 8. Current State and Roadmap

This project is currently in the planning and initial setup phase, with the following roadmap for implementation:

*   **Week 1:** Setup n8n and Supabase connections. Execute Phase 0 Ingestion for 10 test pages.
*   **Week 2:** Develop and implement the H1 Audit workflow (Phase 1), which is a read-only process and serves as a safe introduction to n8n logic.
*   **Week 3:** Implement the Meta & Content Optimizer (Phase 2), ensuring the mandatory Slug Guard is in place to prevent URL breakage during AI-driven updates.

This document will be updated as the project progresses through its development phases.