# Architecture & System Design: IDA AI-Powered SEO Engine

## 1. Introduction

This document provides a detailed breakdown of the architecture and system design for the International Drivers Association (IDA) AI-Powered SEO Engine. The project aims to revolutionize IDA's content management by integrating an Intelligence Layer on top of the existing Statamic CMS, leveraging n8n for automation, Supabase for data storage and vectorization, and Claude AI for semantic optimization. This strategic shift from manual processes to automated AI agents is designed to scale SEO auditing, internal linking, and content freshness across IDA's extensive country-specific page grid without increasing headcount.

## 2. System Architecture Overview

The IDA AI-Powered SEO Engine operates as a continuous loop, orchestrating data flow and transformations across its core components. The fundamental cycle involves n8n pulling data from Statamic, processing it within Supabase with AI assistance, and subsequently pushing optimized content back to Statamic after a review process. Each component plays a distinct and critical role:

| Component          | Role                                       | Description                                                               |
| :----------------- | :----------------------------------------- | :------------------------------------------------------------------------ |
| **Statamic**       | The "Body" (Content Source of Truth)     | Serves as the primary content management system, housing all page content. |
| **n8n**            | The "Nervous System" (Automation Layer)  | Facilitates data movement, transformation, and workflow orchestration.    |
| **Supabase**       | The "Brain" (Vector Database & Analysis) | Stores vectorized content and metadata, enabling advanced analysis.       |
| **Claude (AI)**    | The "Editor" (Content Optimization)      | Analyzes and drafts content, performing semantic optimizations.           |

## 3. Data Flow and PRD Phases

The system's functionality is structured into distinct phases, each addressing a specific goal in the content optimization lifecycle.

### 3.1. PRD Phase 0: The Intelligence Indexer (Ingestion)

**Goal:** To populate Supabase with every country page from Statamic, thereby establishing the foundational data layer for all subsequent automations.

**n8n Workflow Requirements:**

*   **Fetcher:** Initiates data retrieval by calling `GET /api/private/collections/countries/entries`. Authentication is managed via a Bearer Token.
*   **The Locale Filter (Crucial):** Despite potential `?site=` parameters, the API may return entries for all locales. An n8n Filter Node is mandated to retain only entries where `locale === 'en'` (or the designated target language).
*   **The Defensive Extractor:** To account for variations in data structure where fields might reside at the top-level or within a `data` object, n8n expressions must be used to check both locations (e.g., `{{ $json.slug || $json.data.slug }}`).
*   **Content Chunking:** The Recursive Character Text Splitter node is employed to break down the ProseMirror JSON content into manageable segments suitable for AI processing.
*   **Vector Storage:** The Supabase Vector Store node is utilized to store the processed text along with its corresponding mathematical "Embedding."

### 3.2. PRD Phase 1: Structural & H1 Audit

**Goal:** To identify and flag structural weaknesses, such as missing or incorrectly formatted H1 headings, across the entire page grid.

**Requirements:**

*   **Trigger:** The audit workflow is scheduled to run weekly within n8n.
*   **Logic:** The system scans the `page_content` column in Supabase. It counts heading nodes where `attrs.level: 1`. If the H1 count is not equal to 1, the corresponding row in Supabase is flagged as "Action Required," and a notification is dispatched to a designated review queue.

### 3.3. PRD Phase 2: Meta & Content Optimizer

**Goal:** To automate the rewriting of page titles and descriptions to enhance competitive performance in search engine results.

**Requirements:**

*   **Data Sourcing:** The top 3 SERP competitors are fetched via DataForSEO.
*   **Analysis:** The current `meta_title` and competitor data are sent to Claude AI for comprehensive analysis.
*   **Drafting:** Claude AI generates new, optimized Meta Titles and Descriptions.

## 4. Technical Implementation Guidelines (April 2026 Standards)

These guidelines ensure robust and safe interactions with the Statamic CMS, particularly during write-back operations.

### 4.1. Handling Data Types

*   **Select Fields (e.g., Continent):** When the Statamic API returns these as objects, n8n workflows performing a `PATCH` request must send only the raw string key (e.g., "south-america") back to Statamic.
*   **Bard Fields:** Content stored as ProseMirror JSON must be manipulated directly within n8n using JSON nodes. Conversion to raw HTML is to be avoided to preserve Statamic’s native formatting and integrity.

### 4.2. Update Safety: The "Drift" Check

To prevent unintended changes and ensure data consistency, every n8n workflow that writes back to Statamic must adhere to the **Pre/Post Verification Pattern**:

1.  **Capture Pre-PATCH State:** Before executing a `PATCH` request, critical fields such as `title` and `slug` are captured.
2.  **Perform PATCH:** The update operation is then executed.
3.  **Compare Post-PATCH State:** The response from the `PATCH` operation is compared against the captured pre-PATCH state to verify that the `slug` and other guarded fields have not changed unexpectedly.

### 4.3. The Slug Guard (Critical Maintenance)

"The Slug Guard" is a critical mechanism implemented within n8n workflows to prevent accidental deletion of page URLs in Statamic. Statamic's behavior dictates that if the `slug` field is omitted from a `PATCH` request, the corresponding page URL will be deleted. To counteract this, n8n workflows must:

*   **Fetch Current Slug:** Before any `PATCH` operation, n8n must explicitly fetch the current `slug` of the target page.
*   **Echo Slug in Update Body:** The fetched `slug` must then be explicitly included ("echoed") back into the `PATCH` request body, even if the slug itself is not being modified.

## 5. Current State and Roadmap

The IDA AI-Powered SEO Engine is currently in the planning and initial implementation phases, following a structured roadmap to ensure a phased rollout and robust integration:

*   **Week 1:** Focus on establishing the n8n and Supabase connections. This includes running the Phase 0 Ingestion for a set of 10 test pages to validate the data pipeline.
*   **Week 2:** Development and implementation of the H1 Audit (Phase 1). This is designed as a "read-only" workflow, providing a safe environment to familiarize with n8n logic and validate structural checks.
*   **Week 3:** Implementation of the Meta Optimizer (Phase 2), incorporating the mandatory "Slug Guard" to prevent URL breakage during AI-driven content updates.

This phased approach ensures that critical safeguards are in place before deploying advanced AI optimization features, minimizing risks and maximizing system stability.