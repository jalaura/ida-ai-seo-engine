# Implementation Roadmap

## Phase Overview

| Phase | Name | Duration | Goal |
|-------|------|----------|------|
| 0 | Intelligence Indexer | DONE | Ingest all site content into Supabase pgvector |
| 1 | Foundation + Skill Architecture | 3-4 weeks | Laravel skeleton, Slack bot, RAG, skill registry, framework loader |
| 2 | Content Frameworks | 1-2 weeks | Load Koray framework, IDA brand voice, entity SEO rules |
| 3 | Meta Tag Engine (Skill) | 2 weeks | First pluggable skill: bulk meta tag analysis and optimization |
| 4 | Internal Linking Engine (Skill) | 2-3 weeks | Link mapping, opportunity detection, insertion |
| 5 | Content Optimization Engine (Skill) | 2-3 weeks | Content auditing, rewriting, improvement |
| 6 | Content Creation Engine (Skill) | 2-3 weeks | New content generation from briefs |
| 7 | Self-Reflection and Autonomy | 1-2 weeks | Reflection loop, risk classification, auto-execute |
| 8 | Production Hardening | 1-2 weeks | Monitoring, error handling, rollback, documentation |

**Total estimated timeline: 14-20 weeks**

---

## Phase 0: Intelligence Indexer (Complete)

**Status:** Done. All site content embedded in Supabase pgvector.

**What was built:**
- Statamic Private API integration (fetches all entries with pagination)
- - Text extraction + metadata enrichment pipeline
  - - OpenAI embedding generation (text-embedding-ada-002)
    - - Supabase pgvector storage with jsonb metadata
      - - Deduplication and English-only filtering
       
        - **What's in Supabase:**
        - - ~197 country pages (complete)
          - - ~18 driving guide pages (partial, ingestion pipeline being fixed)
            - - ~52 blog posts (partial, ingestion pipeline being fixed)
             
              - **Current tooling:** n8n workflow (to be replaced by native Laravel in Phase 1)
             
              - ---

              ## Phase 1: Foundation + Skill Architecture (3-4 weeks)

              **Goal:** Build the Laravel application skeleton with Slack integration, basic RAG querying, AND the pluggable skill registry that all future engines will use.

              ### Tasks

              **1.1 Laravel Application Setup**
              - [ ] Initialize Laravel 11+ project
              - [ ] - [ ] Configure environment variables (OpenAI, Supabase, Slack, Statamic)
              - [ ] - [ ] Set up Redis for queue and cache
              - [ ] - [ ] Set up database for task tracking and audit logs
              - [ ] - [ ] Create base models: Task, AuditLog, ApprovalRequest
             
              - [ ] **1.2 Supabase RAG Integration**
              - [ ] - [ ] Build Supabase client service (query, upsert, delete)
              - [ ] - [ ] Implement vector similarity search (cosine distance)
              - [ ] - [ ] Implement metadata-filtered queries
              - [ ] - [ ] Build embedding generation service (OpenAI API)
              - [ ] - [ ] Create Knowledge Base sync command (replaces n8n workflow)
              - [ ]   - Fetch from Statamic API with pagination
              - [ ]     - Chunk content (recursive text splitter)
              - [ ]   - Generate embeddings
              - [ ]     - Upsert to Supabase
              - [ ] - [ ] Test: query "pages about driving in France" returns relevant results
             
              - [ ] **1.3 Statamic CMS Client**
              - [ ] - [ ] Build Statamic API client (GET entries, PUT/PATCH entries)
              - [ ] - [ ] Handle pagination, rate limiting, auth
              - [ ] - [ ] Implement read operations: get entry, list entries by collection
              - [ ] - [ ] Implement write operations: update entry fields
              - [ ] - [ ] Locale safety: only modify `default` site
              - [ ] - [ ] Test: read a country page, update its meta title, verify in CMS
             
              - [ ] **1.4 Slack Bot**
              - [ ] - [ ] Create Slack app with required scopes
              - [ ] - [ ] Implement webhook endpoint for slash commands
              - [ ] - [ ] Implement event subscription for @mentions
              - [ ] - [ ] Build message formatting service (blocks, attachments)
              - [ ] - [ ] Implement approval workflow (emoji reactions to action)
              - [ ] - [ ] Implement threaded conversations
              - [ ] - [ ] Register `/seo` slash command with subcommands
              - [ ] - [ ] Test: `/seo status` returns "No active tasks"
             
              - [ ] **1.5 Task Queue System**
              - [ ] - [ ] Set up Laravel Queue with Redis driver
              - [ ] - [ ] Create base Job class with logging, error handling, retry logic
              - [ ] - [ ] Build task status tracking (pending to running to completed/failed)
              - [ ] - [ ] Implement Slack notification on task start/complete/fail
              - [ ] - [ ] Set up Laravel Scheduler for recurring tasks
              - [ ] - [ ] Test: queue a dummy task, see it execute, get Slack notification
             
              - [ ] **1.6 Skill Registry**
              - [ ] - [ ] Build SkillRegistry service that scans `skills/` directory for `skill.json` manifests
              - [ ] - [ ] Implement skill trigger matching (command match, keyword detection, schedule match)
              - [ ] - [ ] Build skill context loader: loads system prompt + relevant frameworks into LLM context
              - [ ] - [ ] Define standard Skill interface (actions, triggers, capabilities, autonomy config)
              - [ ] - [ ] Implement skill-to-skill communication through Agent Controller
              - [ ] - [ ] Hot-reload support: detect new/updated skills without full restart
              - [ ] - [ ] Test: create a dummy "hello-world" skill, trigger via Slack, see it execute
             
              - [ ] **1.7 Framework Loader**
              - [ ] - [ ] Build FrameworkLoader service that reads `frameworks/` directory
              - [ ] - [ ] Parse `framework.json` to understand load rules (`always`, `on_action`)
              - [ ] - [ ] Implement context assembly: merge agent prompt + skill prompt + framework docs + RAG context
              - [ ] - [ ] Framework priority system (when multiple frameworks apply, load in priority order)
              - [ ] - [ ] Token budget management: ensure combined context doesn't exceed LLM limits
              - [ ] - [ ] Test: trigger a content action, verify Koray framework rules appear in agent reasoning
             
              - [ ] ### Phase 1 Deliverable
              - [ ] A working Laravel app that can:
              - [ ] - Receive a Slack command
              - [ ] - Query the Knowledge Base
              - [ ] - Read/write Statamic entries
              - [ ] - Report results back to Slack
              - [ ] - Load and execute pluggable skills
              - [ ] - Inject content frameworks into agent reasoning
             
              - [ ] ---
             
              - [ ] ## Phase 2: Content Frameworks (1-2 weeks)
             
              - [ ] **Goal:** Encode SEO methodologies and brand guidelines as loadable framework documents that shape all agent output.
             
              - [ ] ### Tasks
             
              - [ ] **2.1 Koray Topical Authority Framework**
              - [ ] - [ ] Encode core philosophy: EAV thinking, source context, network effects
              - [ ] - [ ] Encode topical map design: 5-pillar architecture, core vs. outer sections
              - [ ] - [ ] Encode 41 content writing rules: question H2, 40-word answers, factual language
              - [ ] - [ ] Encode internal linking rules: progressive intent, anchor text diversity, contextual bridges
              - [ ] - [ ] Encode audit checklist: 6-month reconfiguration process
              - [ ] - [ ] Create `framework.json` with load rules (which files for which actions)
             
              - [ ] **2.2 Entity SEO Framework**
              - [ ] - [ ] Knowledge Graph optimization rules
              - [ ] - [ ] Schema markup patterns (@id, sameAs, about, mentions)
              - [ ] - [ ] AI search readiness (GEO/AEO for ChatGPT, Perplexity, AI Overviews)
              - [ ] - [ ] Entity Home methodology (Kalicube method)
             
              - [ ] **2.3 IDA Brand Voice Framework**
              - [ ] - [ ] Tone guidelines: helpful, authoritative, traveler-friendly
              - [ ] - [ ] Terminology rules: "International Driver's License" (never "IDP"), correct brand naming
              - [ ] - [ ] Meta tag templates per collection
              - [ ] - [ ] CTA patterns, heading style, content structure conventions
             
              - [ ] **2.4 Framework Testing**
              - [ ] - [ ] Create test harness: give the agent a content task with and without frameworks loaded
              - [ ] - [ ] Verify Koray rules are enforced (question H2s, EAV coverage, no vague language)
              - [ ] - [ ] Verify brand voice is consistent (correct terminology, right tone)
              - [ ] - [ ] Verify self-reflection catches framework violations
             
              - [ ] ### Phase 2 Deliverable
              - [ ] All frameworks encoded and loadable. When the agent creates content, it provably follows Koray's methodology and IDA's brand voice. Self-reflection catches violations.
             
              - [ ] ---
             
              - [ ] ## Phase 3: Meta Tag Engine, First Skill (2 weeks)
             
              - [ ] **Goal:** The first pluggable skill: automated meta tag analysis and optimization. This validates the entire skill architecture.
             
              - [ ] ### Tasks
             
              - [ ] **3.1 Meta Tag Auditor**
              - [ ] - [ ] Scan all pages for meta tag issues:
              - [ ]   - Missing meta title or description
              - [ ]     - Title too long (>60 chars) or too short (<30 chars)
              - [ ]   - Description too long (>160 chars) or too short (<70 chars)
              - [ ]     - Duplicate titles or descriptions across pages
              - [ ]   - Missing target keyword in title
              - [ ]     - Title doesn't match page content/intent
              - [ ] - [ ] Generate audit report with severity levels
              - [ ] - [ ] Post audit summary to Slack
             
              - [ ] **3.2 Meta Tag Generator**
              - [ ] - [ ] Build LLM prompt for meta tag generation:
              - [ ]   - Input: page content (from RAG), current meta tags, target keyword, collection type
              - [ ]     - Output: optimized title + description
              - [ ]   - Constraints: character limits, keyword inclusion, brand voice
              - [ ]   - [ ] Implement IDA template matching per collection
              - [ ]   - [ ] Batch processing: handle 50 pages per run
              - [ ]   - [ ] Before/after diff generation for Slack preview
             
              - [ ]   **3.3 Cannibalization Detection**
              - [ ]   - [ ] For each proposed meta tag change, query Knowledge Base for pages with similar keywords
              - [ ]   - [ ] Flag if two pages would compete for the same primary keyword
              - [ ]   - [ ] Suggest differentiation strategies
             
              - [ ]   **3.4 Integration**
              - [ ]   - [ ] Wire up `/seo meta [scope]` command
              - [ ]   - [ ] Auto-execute meta tag updates (Tier 1) with Slack notification
              - [ ]   - [ ] Audit log: store before/after values for every change
              - [ ]   - [ ] Rollback command: `/seo rollback [task-id]`
             
              - [ ]   ### Phase 3 Deliverable
              - [ ]   `/seo meta countries` triggers agent to scan all country pages, identify issues, generate optimized meta tags (following IDA brand templates from the framework), auto-apply safe changes, and report to Slack. This validates the full skill architecture end-to-end.
             
              - [ ]   ---
             
              - [ ]   ## Phase 4: Internal Linking Engine, Skill (2-3 weeks)
             
              - [ ]   **Goal:** Map existing links, find opportunities, and implement new internal links.
             
              - [ ]   ### Tasks
             
              - [ ]   **4.1 Link Crawler**
              - [ ]   - [ ] Crawl all pages and extract existing internal links (source, anchor text, target)
              - [ ]   - [ ] Store link map in database
              - [ ]   - [ ] Detect orphan pages (0 inbound internal links)
              - [ ]   - [ ] Detect dead internal links (404s)
              - [ ]   - [ ] Generate link graph visualization (optional)
             
              - [ ]   **4.2 Opportunity Finder**
              - [ ]   - [ ] For each page, use RAG to find semantically related pages
              - [ ]   - [ ] Apply linking rules (see Agent Behavior doc):
              - [ ]     - Blog to parent country page
              - [ ]   - Country to/from driving guide
              - [ ]     - Related blog posts cross-link
              - [ ]   - No orphan pages
              - [ ]   - [ ] Score each opportunity: relevance, authority value, user journey logic
              - [ ]   - [ ] Generate suggested anchor text (varied, natural, intent-aligned)
             
              - [ ]   **4.3 Link Insertion**
              - [ ]   - [ ] Parse page content to find natural insertion points
              - [ ]   - [ ] Generate the linked text in context (not just "click here")
              - [ ]   - [ ] Preview the change with surrounding paragraph context
              - [ ]   - [ ] Queue for human approval (Tier 2)
              - [ ]   - [ ] On approval, update page content via Statamic API
              - [ ]   - [ ] Update Knowledge Base embeddings for modified pages
             
              - [ ]   **4.4 Anchor Text Management**
              - [ ]   - [ ] Track anchor text usage across the entire site
              - [ ]   - [ ] Prevent anchor text cannibalization (same anchor to different targets)
              - [ ]   - [ ] Ensure anchor text diversity for each target page
              - [ ]   - [ ] Flag over-optimized anchor text patterns
             
              - [ ]   ### Phase 4 Deliverable
              - [ ]   `/seo links blog` triggers agent to map all blog internal links, use Koray's progressive intent linking rules, identify 47 opportunities, present top 10 in Slack with context, human approves 8, agent inserts links.
             
              - [ ]   ---
             
              - [ ]   ## Phase 5: Content Optimization Engine, Skill (2-3 weeks)
             
              - [ ]   **Goal:** Analyze existing content and generate targeted improvements.
             
              - [ ]   ### Tasks
             
              - [ ]   **5.1 Content Auditor**
              - [ ]   - [ ] Audit all pages for content quality:
              - [ ]     - Word count vs. target by collection type
              - [ ]   - Heading structure (proper H1, H2, H3 hierarchy)
              - [ ]     - Keyword density and placement
              - [ ]   - Readability score (Flesch-Kincaid)
              - [ ]     - Content freshness (last modified date)
              - [ ]   - Thin content detection (<500 words)
              - [ ]   - [ ] Generate content health report
              - [ ]   - [ ] Prioritize pages by improvement potential
             
              - [ ]   **5.2 Content Optimizer**
              - [ ]   - [ ] Build LLM prompt for content optimization:
              - [ ]     - Input: current content, page metadata, target keywords, competitor insights
              - [ ]   - Output: specific section improvements (not full rewrites)
              - [ ]     - Constraints: maintain existing facts, match brand voice, preserve internal links
              - [ ] - [ ] Generate before/after diff for each change
              - [ ] - [ ] Support targeted optimization: "improve the FAQ section" or "expand the requirements section"
             
              - [ ] **5.3 Integration**
              - [ ] - [ ] Wire up `/seo optimize [url]` command
              - [ ] - [ ] Queue all content changes for approval (Tier 2)
              - [ ] - [ ] Slack preview with inline diff
              - [ ] - [ ] Track optimization history per page (don't re-optimize recently optimized content)
             
              - [ ] ### Phase 5 Deliverable
              - [ ] `/seo optimize /andorra-driving-guide` triggers agent to analyze the page against Koray's EAV framework, identify 3 improvement areas, generate optimized sections using question H2 format, post diff to Slack, human approves, agent updates CMS.
             
              - [ ] ---
             
              - [ ] ## Phase 6: Content Creation Engine, Skill (2-3 weeks)
             
              - [ ] **Goal:** Generate new pages from keyword research and content gap analysis.
             
              - [ ] ### Tasks
             
              - [ ] **6.1 Content Gap Finder**
              - [ ] - [ ] Compare target keyword list against existing content coverage
              - [ ] - [ ] Identify keywords with no dedicated page
              - [ ] - [ ] Use RAG to verify the gap isn't covered as a section in another page
              - [ ] - [ ] Prioritize gaps by search volume, difficulty, and business relevance
             
              - [ ] **6.2 Brief Generator**
              - [ ] - [ ] Generate content briefs from gap analysis:
              - [ ]   - Target keyword and secondary keywords
              - [ ]     - Search intent (informational, transactional, navigational)
              - [ ]   - Suggested outline (H1, H2s, H3s)
              - [ ]     - Target word count
              - [ ]   - Internal links to include
              - [ ]     - Competing pages to reference
              - [ ] - [ ] Post brief to Slack for approval before writing
             
              - [ ] **6.3 Content Writer**
              - [ ] - [ ] Build LLM prompt for full content generation:
              - [ ]   - Input: approved brief, brand voice guidelines, existing related content (from RAG)
              - [ ]     - Output: complete page with headings, meta tags, and internal link suggestions
              - [ ]   - Constraints: factual accuracy, no duplication of existing content, proper structure
              - [ ]   - [ ] Generate in sections (outline then intro then body sections then conclusion then meta tags)
              - [ ]   - [ ] Self-reflection loop with extra emphasis on factual accuracy and uniqueness
             
              - [ ]   **6.4 Integration**
              - [ ]   - [ ] Wire up `/seo create [brief]` command
              - [ ]   - [ ] Two-stage approval: brief approval then content approval
              - [ ]   - [ ] On final approval, create new entry in Statamic
              - [ ]   - [ ] Auto-generate Knowledge Base embedding for new page
              - [ ]   - [ ] Trigger internal linking engine to find linking opportunities for the new page
             
              - [ ]   ### Phase 6 Deliverable
              - [ ]   `/seo create blog post about winter driving tips in Scandinavia` triggers agent to check topical map placement (Koray framework), research keywords, generate brief with EAV coverage plan, get approval, write 1500-word post following 41 authorship rules with IDA brand voice, get final approval, publish to Statamic.
             
              - [ ]   ---
             
              - [ ]   ## Phase 7: Self-Reflection and Autonomy (1-2 weeks)
             
              - [ ]   **Goal:** Implement the full reflection loop and tiered autonomy system.
             
              - [ ]   ### Tasks
             
              - [ ]   - [ ] Build the reflection prompt (evaluate output against criteria)
              - [ ]   - [ ] Implement reflection loop with max 3 iterations
              - [ ]   - [ ] Build risk classifier (low/high based on action type, volume, confidence)
              - [ ]   - [ ] Auto-execute flow for Tier 1 actions
              - [ ]   - [ ] Approval queue flow for Tier 2 actions
              - [ ]   - [ ] Escalation flow when reflection fails after max iterations
              - [ ]   - [ ] Logging: store reflection reasoning for every task
              - [ ]   - [ ] A/B comparison: track whether reflected outputs perform better over time
             
              - [ ]   ---
             
              - [ ]   ## Phase 8: Production Hardening (1-2 weeks)
             
              - [ ]   **Goal:** Make the system reliable, observable, and maintainable.
             
              - [ ]   ### Tasks
             
              - [ ]   - [ ] Error handling: graceful failures, retries with backoff, Slack error notifications
              - [ ]   - [ ] Monitoring: task success rate, API latency, queue depth
              - [ ]   - [ ] Rollback system: one-command reversal of any change
              - [ ]   - [ ] Rate limiting: respect all API limits (OpenAI, Statamic, Slack)
              - [ ]   - [ ] Audit log viewer: query past actions by date, page, type
              - [ ]   - [ ] Documentation: API docs, deployment guide, troubleshooting runbook
              - [ ]   - [ ] Automated tests: unit tests for each engine, integration tests for Slack flow
              - [ ]   - [ ] Cost tracking: OpenAI token usage per task type
              - [ ]   - [ ] Knowledge Base maintenance: scheduled re-sync, stale embedding cleanup
              - [ ]   - [ ] Security review: token rotation, webhook verification, input sanitization
