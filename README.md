# IDA AI-Powered SEO Engine: Project Overview

## 1. Project Overview & Mission

The International Drivers Association (IDA) manages an extensive grid of country-specific web pages. This project introduces an **Intelligence Layer** atop the existing Statamic CMS, leveraging **n8n** and **Supabase**. The primary objective is to transition from manual content management to an automated system driven by AI agents, thereby scaling SEO auditing, internal linking, and content freshness without increasing operational headcount.

### Primary Goals

*   **Centralize Site Intelligence**: Migrate page content and SEO metadata into a searchable Supabase Vector Store.
*   **Automate Audits**: Instantly identify H1 violations and missing entities across over 200 jurisdictions.
*   **Semantic Optimization**: Utilize AI (Claude) to dynamically rewrite metadata based on real-time competitor SERP data.
*   **Contextual Linking**: Automatically identify and propose internal links through semantic "Vector" matching.

## 2. System Architecture

The system operates as a closed-loop mechanism: n8n extracts data from Statamic, processes it within Supabase using AI, and subsequently pushes optimized content back to Statamic following a review process.

| Component       | Role                                    | Description                                        |
| :-------------- | :-------------------------------------- | :------------------------------------------------- |
| **Statamic**    | The "Body" (Content Source)             | Existing CMS where content resides.                |
| **n8n**         | The "Nervous System" (Automation)       | Orchestrates data movement and transformation.     |
| **Supabase**    | The "Brain" (Data & Metadata Storage)   | Stores vectors and metadata for AI analysis.       |
| **Claude (AI)** | The "Editor" (Content Analysis & Draft) | Analyzes and drafts content optimizations.         |

## 3. Tech Stack

This project utilizes a modern, robust tech stack designed for scalability and automation:

*   **n8n**: Workflow automation and integration platform.
*   **Supabase**: Open-source Firebase alternative, used as a vector database for semantic search and metadata storage.
*   **Statamic**: Flat-file CMS, serving as the primary source of truth for content.
*   **Claude**: Advanced AI model for natural language processing, content analysis, and generation.

## 4. Setup Instructions

This section outlines the initial setup phases for the IDA AI-Powered SEO Engine. The project is currently in its planning stages, and these instructions reflect the intended implementation roadmap.

### Week 1: Setup n8n + Supabase Connection

**Goal**: Establish foundational connectivity and test data ingestion.

1.  **n8n Instance**: Ensure an n8n instance is deployed and accessible.
2.  **Supabase Project**: Set up a new Supabase project, including database and vector store configurations.
3.  **Initial Ingestion**: Configure an n8n workflow to fetch data from Statamic and store it in Supabase. Begin with 10 test pages to validate the process.

### Week 2: Build the H1 Audit Workflow

**Goal**: Implement a read-only workflow to identify structural SEO issues.

1.  **Develop H1 Audit Workflow**: Create an n8n workflow that scans the `page_content` column in Supabase.
2.  **Logic Implementation**: The workflow should count heading nodes with `attrs.level: 1`. If the H1 count is not equal to 1, flag the corresponding row in Supabase as "Action Required" and trigger a notification to a review queue.

### Week 3: Implement the Meta Optimizer with Slug Guard

**Goal**: Automate meta title and description optimization while ensuring URL integrity.

1.  **Meta Optimizer Workflow**: Develop an n8n workflow for automated rewriting of Titles and Descriptions.
2.  **Data Sourcing**: Integrate with DataForSEO to fetch top 3 SERP competitors.
3.  **AI Analysis & Drafting**: Send the current `meta_title` and competitor data to Claude for analysis and generation of new meta titles and descriptions.
4.  **Slug Guard Implementation**: Crucially, the n8n workflow MUST fetch the current slug from Statamic and "echo" it back in the update body during any PATCH request to prevent accidental deletion of page URLs.

## 5. Quick Start Guide (Roadmap)

This section provides a high-level roadmap for new users to get started with the IDA AI-Powered SEO Engine. The features described are part of the planned development.

### PRD Phase 0: The Intelligence Indexer (Ingestion)

**Goal**: Populate Supabase with all country pages from Statamic to enable future automations.

#### n8n Workflow Requirements:

*   **Fetcher**: Call `GET /api/private/collections/countries/entries` using Bearer Token authentication.
*   **Locale Filter**: Implement an n8n Filter Node to retain only entries where `locale === 'en'` (or the target language), as the API returns entries for all locales regardless of the `?site=` parameter.
*   **Defensive Extractor**: Use n8n expressions like `{{ $json.slug || $json.data.slug }}` to handle fields that may appear at either the top-level or within the `data` object.
*   **Content Chunking**: Utilize the Recursive Character Text Splitter node to break down ProseMirror JSON content into manageable chunks for AI processing.
*   **Vector Storage**: Employ the Supabase Vector Store node to store the processed text and its corresponding mathematical "Embedding."

### PRD Phase 1: Structural & H1 Audit

**Goal**: Identify structural weaknesses (e.g., missing H1s or poorly structured H2s) across the page grid.

#### Requirements:

*   **Trigger**: The workflow will be scheduled to run weekly in n8n.
*   **Logic**: Scan the `page_content` column in Supabase. Count heading nodes where `attrs.level: 1`. If the H1 count is not equal to 1, flag the row in Supabase as "Action Required" and send a notification to a review queue.

### PRD Phase 2: Meta & Content Optimizer

**Goal**: Achieve automated rewriting of Titles and Descriptions to outperform competitors.

#### Requirements:

*   **Data Sourcing**: Fetch the top 3 SERP competitors via DataForSEO.
*   **Analysis**: Send the current `meta_title` and competitor data to Claude for analysis.
*   **Drafting**: Claude will generate a new Meta Title and Description.
*   **The Slug Guard (Critical Maintenance)**: To prevent accidental deletion of page URLs, the n8n workflow MUST fetch the current slug and "echo" it back in the update body for any PATCH request to Statamic.

## 6. Technical Implementation Guidelines (April 2026 Standards)

### Handling Data Types

*   **Select Fields (e.g., Continent)**: The Statamic API returns these as objects. When writing back via n8n (PATCH), only the raw string key (e.g., "south-america") should be sent.
*   **Bard Fields**: Content is stored as ProseMirror JSON. n8n should manipulate the JSON nodes directly rather than converting to raw HTML to preserve Statamic’s formatting.

### Update Safety (The "Drift" Check)

Every n8n workflow that writes back to Statamic should adhere to the **Pre/Post Verification Pattern**:

1.  **Capture State**: Record the state of critical fields (e.g., title, slug) before performing the PATCH operation.
2.  **Perform PATCH**: Execute the update operation.
3.  **Compare Response**: Verify that the slug and other guarded fields have not changed unexpectedly in the response, ensuring data integrity.