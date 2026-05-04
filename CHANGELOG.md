# Changelog

All notable changes to the IDA AI-Powered SEO Engine will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

### Planned — Phase 0: Intelligence Indexer (Ingestion)
- [ ] n8n workflow: Fetcher node calling `GET /api/private/collections/countries/entries`
- [ ] n8n workflow: Locale Filter node (retain `locale === 'en'` only)
- [ ] n8n workflow: Defensive Extractor using `{{ $json.slug || $json.data.slug }}`
- [ ] n8n workflow: Recursive Character Text Splitter for ProseMirror JSON chunking
- [ ] Supabase Vector Store node integration for embedding storage
- [ ] Bearer Token authentication setup for Statamic Private API

### Planned — Phase 1: Structural & H1 Audit
- [ ] n8n workflow: Weekly scheduled trigger
- [ ] n8n workflow: Supabase `page_content` column scanner
- [ ] n8n workflow: H1 heading node counter (`attrs.level: 1`)
- [ ] Supabase: "Action Required" flag logic for pages with H1 count ≠ 1
- [ ] Notification system: Review queue integration

### Planned — Phase 2: Meta & Content Optimizer
- [ ] DataForSEO integration: Fetch top 3 SERP competitors per page
- [ ] Claude AI integration: Meta title and description analysis
- [ ] Claude AI integration: Optimized meta content generation
- [ ] n8n workflow: Slug Guard implementation (pre-PATCH slug fetch + echo)
- [ ] n8n workflow: Pre/Post Verification Pattern (Drift Check)

### Planned — Infrastructure & Safety
- [ ] Supabase schema: `page_content`, `meta_title`, `slug`, `audit_status` columns
- [ ] n8n credential management for Statamic Bearer Token, Supabase, DataForSEO, Claude
- [ ] Select Field writeback normalization (raw string key only)
- [ ] Bard Field ProseMirror JSON manipulation (no HTML conversion)

---

## [1.0.0-blueprint] — 2026-04-01

### Added
- Initial project blueprint document (Version 1.0, Integration Reference April 2026)
- Defined core stack: n8n, Supabase, Statamic, Claude
- Documented three-phase architecture (Phase 0, Phase 1, Phase 2)
- Established technical implementation guidelines (April 2026 Standards)
- Defined Slug Guard and Drift Check safety patterns
- Created newbie onboarding roadmap (Weeks 1–3)

---

> **Note for Acquisition Due Diligence**: This project is currently in the pre-build/planning stage as of May 2026. All items listed under `[Unreleased]` represent the planned feature set. No production workflows have been deployed yet. The blueprint and documentation represent the full intended scope and technical design.
