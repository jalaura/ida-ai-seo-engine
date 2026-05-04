# Product Requirements Document (PRD) - IDA AI-Powered SEO Engine

## Executive Summary

This document outlines the Product Requirements for the International Drivers Association (IDA) AI-Powered SEO Engine, a strategic initiative designed to revolutionize content management and SEO optimization. By leveraging n8n for automation, Supabase as a vector database, and Statamic as the source of truth, IDA aims to transition from manual content processes to an automated, AI-driven system. This shift will enable scalable SEO auditing, intelligent internal linking, and dynamic content freshness across IDA's extensive, country-specific page grid without increasing headcount. The project is critical for enhancing IDA's digital footprint, improving operational efficiency, and positioning the organization for future growth and acquisition readiness by demonstrating advanced technological capabilities and a robust, scalable infrastructure.

## Goals

The primary goals of the IDA AI-Powered SEO Engine are to:

*   **Centralize Site Intelligence**: Consolidate page content and SEO metadata into a searchable Supabase Vector Store, creating a single source of truth for all SEO-related data.
*   **Automate Audits**: Implement automated systems to instantly identify structural SEO weaknesses, such as H1 violations and missing entities, across over 200 jurisdictions.
*   **Semantic Optimization**: Utilize AI (specifically Claude) to dynamically rewrite metadata based on real-time competitive Search Engine Results Page (SERP) data, ensuring optimal search visibility.
*   **Contextual Linking**: Develop an AI-driven system to automatically identify and propose relevant internal links using semantic "Vector" matching, enhancing site navigation and SEO value.

## Phases

The project is structured into distinct phases, each building upon the last to deliver a comprehensive AI-powered SEO solution.

### PRD Phase 0: The Intelligence Indexer (Ingestion)

**Goal**: To populate Supabase with all country page content from Statamic, establishing the foundational data layer for all subsequent automations.

**n8n Workflow Requirements**:

*   **Fetcher**: Implement a GET request to `/api/private/collections/countries/entries` using Bearer Token authentication to retrieve page data.
*   **The Locale Filter (Crucial)**: Despite API limitations, an n8n Filter Node MUST be used to retain only entries with `locale === 'en'` (or the designated target language).
*   **The Defensive Extractor**: Utilize n8n expressions (e.g., `{{ $json.slug || $json.data.slug }}`) to robustly extract fields that may reside at either the top-level or within the `data` object.
*   **Content Chunking**: Employ the Recursive Character Text Splitter node to segment ProseMirror JSON content into manageable chunks suitable for AI processing.
*   **Vector Storage**: Store the processed text and its corresponding mathematical "Embedding" within the Supabase Vector Store.

### PRD Phase 1: Structural & H1 Audit

**Goal**: To identify and flag structural SEO weaknesses, such as missing or improperly formatted H1 headings, across the entire page grid.

**Requirements**:

*   **Trigger**: The audit workflow will be scheduled to run weekly within n8n.
*   **Logic**: Scan the `page_content` column in Supabase. Count heading nodes where `attrs.level: 1`. If the H1 count is not equal to 1, the corresponding row in Supabase will be flagged as "Action Required," and a notification will be sent to a designated review queue.

### PRD Phase 2: Meta & Content Optimizer

**Goal**: To automate the rewriting of page Titles and Descriptions to outperform competitors in SERP rankings.

**Requirements**:

*   **Data Sourcing**: Fetch the top 3 SERP competitors' data via DataForSEO.
*   **Analysis**: Send the current `meta_title` and competitor data to Claude.
*   **Drafting**: Claude will generate optimized new Meta Titles and Descriptions.
*   **The Slug Guard (Critical Maintenance)**: To prevent accidental URL deletion, the n8n workflow MUST fetch the current slug and "echo" it back in the update body during any PATCH request to Statamic.

## Success Metrics

Success for the IDA AI-Powered SEO Engine will be measured through a combination of operational efficiency gains and improved SEO performance. Key metrics include:

| Metric Category | Specific Metric | Target | Measurement Method | Acquisition Readiness Impact |
| :-------------- | :-------------- | :----- | :----------------- | :--------------------------- |
| **Operational Efficiency** | Reduction in manual SEO audit time | 80% | Time saved per audit cycle | Demonstrates cost savings and scalability. |
| | Number of automated internal links proposed/implemented | 1000+ per month | n8n workflow logs | Highlights automation capabilities and reduced manual effort. |
| | Content freshness score improvement | 20% | Custom metric based on content update frequency | Shows dynamic content management without increased headcount. |
| **SEO Performance** | Organic traffic growth | 15% increase | Google Analytics | Direct impact on business value and market reach. |
| | SERP ranking improvement for target keywords | Top 3 positions | SEO tracking tools | Indicates competitive advantage and effective AI optimization. |
| | Reduction in H1 violations | 95% | Supabase audit flags | Improves site health and user experience. |
| | Click-Through Rate (CTR) improvement for optimized pages | 10% | Google Search Console | Validates effectiveness of AI-generated meta descriptions. |

## Future Scalability

The architecture of the IDA AI-Powered SEO Engine is designed with scalability and future expansion in mind, making it highly attractive for acquisition.

*   **Modular Design**: The phased approach and clear separation of concerns (n8n for orchestration, Supabase for data, Claude for AI) allow for independent development, testing, and scaling of each component.
*   **Headless CMS Integration**: Statamic's API-first approach ensures that the system can easily integrate with other content sources or be adapted to different front-end requirements.
*   **AI Agnostic**: While Claude is currently used, the system is designed to be AI-agnostic, allowing for seamless integration of other LLMs or AI services as technology evolves or specific needs arise.
*   **Jurisdiction Expansion**: The system is built to handle IDA's massive, country-specific page grid, and can be readily expanded to support additional locales or new markets with minimal architectural changes.
*   **Data-Driven Enhancements**: The centralized Supabase Vector Store provides a rich dataset for continuous improvement and the development of new AI-powered features, such as personalized content recommendations or advanced competitor analysis.
*   **Acquisition Readiness**: The clear documentation, robust architecture, and demonstrated ability to automate complex SEO tasks position IDA as a technologically advanced and efficient operation, significantly enhancing its value proposition for potential acquirers. The system's ability to scale without proportional increases in headcount is a key differentiator.

## Technical Implementation Guidelines (April 2026 Standards)

These guidelines ensure robust and maintainable implementation.

### Handling Data Types

*   **Select Fields (e.g., Continent)**: When writing back to Statamic via n8n (PATCH requests), only the raw string key (e.g., "south-america") should be sent, not the full object.
*   **Bard Fields**: Content stored as ProseMirror JSON should be manipulated directly within n8n using JSON nodes, preserving Statamic's formatting rather than converting to raw HTML.

### Update Safety (The "Drift" Check)

Every n8n workflow that performs write operations back to Statamic MUST incorporate the Pre/Post Verification Pattern:

1.  **Capture State**: Record the state of critical fields (e.g., `title`, `slug`) before executing the PATCH request.
2.  **Perform PATCH**: Execute the update operation.
3.  **Compare Response**: Verify that the `slug` and other guarded fields in the response have not unexpectedly changed, ensuring data integrity.

## Success & Newbie Roadmap

This roadmap outlines the initial implementation plan and serves as a guide for new team members.

| Week | Focus | Key Activities | Expected Outcome |
| :--- | :---- | :------------- | :--------------- |
| **Week 1** | Setup & Ingestion | Establish n8n and Supabase connections. Run Phase 0 Ingestion for 10 test pages. | Functional data ingestion pipeline for test data. |
| **Week 2** | H1 Audit Development | Build the "read-only" H1 Audit workflow in n8n. | Automated identification of H1 violations. |
| **Week 3** | Meta Optimizer Implementation | Implement the Meta Optimizer, including the mandatory Slug Guard. | Automated, safe meta title and description optimization. |
