# System Architecture

## 1. Overview

The IDA AI SEO Engine is an autonomous agent that continuously improves the SEO performance of internationaldriversassociation.com. It operates across four capability domains — meta tags, internal linking, content optimization, and content creation — and uses a tiered autonomy model where low-risk actions execute automatically while high-risk changes require human approval via Slack.

The system is built on three pillars:

1. **Knowledge Base (RAG)** — All site content is embedded in Supabase pgvector, giving the agent full semantic understanding of every page, its intent, its keywords, and how it relates to other pages.
2. 2. **Self-Reflecting Agent Loop** — The agent doesn't just execute tasks; it plans, executes, evaluates its own output, and iterates before finalizing. Every action goes through a reflect-then-act cycle.
   3. 3. **Human-in-the-Loop via Slack** — Slack is the sole communication channel. The team sends instructions, receives reports, and approves high-risk changes — all without leaving Slack.
     
      4. ---
     
      5. ## 2. System Components
     
      6. ### 2.1 Agent Controller (Orchestrator)
     
      7. The brain of the system. Receives tasks (from Slack commands, scheduled jobs, or internal triggers), routes them to the appropriate engine, and manages the execution lifecycle.
     
      8. **Responsibilities:**
      9. - Parse and validate incoming task requests
         - - Route tasks to the correct engine (meta tags, linking, content, creation)
           - - Manage the self-reflection loop (plan → execute → evaluate → refine)
             - - Enforce autonomy tiers (auto-execute vs. queue for approval)
               - - Track task state and history
                 - - Handle error recovery and retries
                  
                   - **Key design decisions:**
                   - - Single-threaded task execution per page to prevent conflicting edits
                     - - Task queue with priority levels (urgent, normal, background)
                       - - Idempotent operations — re-running a task produces the same result
                         - - Full audit trail of every action taken
                          
                           - ### 2.2 Knowledge Base (Supabase pgvector)
                          
                           - The agent's memory. Every page on the site is chunked, embedded, and stored with rich metadata. This powers all agent reasoning — from understanding page intent to finding linking opportunities.
                          
                           - **Current state (Phase 0 complete):**
                           - - **Supabase project**: `nbeancschpzqblzvmmjs`
                             - - **Table**: `documents` with columns `id`, `content`, `metadata` (jsonb), `embedding` (vector)
                               - - **Collections indexed**: `countries` (197 pages), `driving-guides` (partial), `blog` (partial)
                                 - - **Embedding model**: OpenAI `text-embedding-ada-002`
                                  
                                   - **Metadata schema per document:**
                                   - ```json
                                     {
                                       "url": "/international-drivers-license-france/",
                                       "slug": "france",
                                       "title": "Get an International Driver's License for France",
                                       "h1": "International Driver's License France",
                                       "meta_title": "...",
                                       "meta_description": "...",
                                       "collection": "countries",
                                       "continent": "Europe",
                                       "region": "Western Europe",
                                       "category": "international-drivers-license",
                                       "locale": "en",
                                       "source": "statamic"
                                     }
                                     ```

                                     **RAG query patterns the agent uses:**
                                     - "Find all pages about driving in Europe" → metadata filter + semantic search
                                     - - "Which pages mention car rental in Morocco?" → full-text + embedding similarity
                                       - - "What pages could link to this new blog post about road trips?" → semantic nearest neighbors
                                         - - "Are there any pages with duplicate meta descriptions?" → metadata scan
                                          
                                           - ### 2.3 Statamic CMS (Source of Truth)
                                          
                                           - The live website runs on Statamic (Laravel-based flat-file CMS). All content changes ultimately write back to Statamic via its Private REST API.
                                          
                                           - **API details:**
                                           - - Base URL: `https://internationaldriversassociation.com/api/private`
                                             - - Auth: Bearer token (HTTP Header Auth)
                                               - - Collections: `countries`, `driving-guides`, `blog`
                                                 - - Multilingual: ~10 locales, English is `default` site
                                                   - - Pagination: Standard Laravel pagination (`meta.current_page`, `meta.last_page`, `links.next`)
                                                    
                                                     - **URL structures:**
                                                     - | Collection | URL Pattern | Example |
                                                     - |-----------|-------------|--------|
                                                     - | Countries | `/international-drivers-license-{country}/` | `/international-drivers-license-france/` |
                                                     - | Driving Guides | `/{country}-driving-guide` | `/andorra-driving-guide` |
                                                     - | Blog | `/blog/{post-slug}` | `/blog/tuscany-italy-road-trip` |
                                                    
                                                     - **Write operations the agent needs:**
                                                     - - Update entry fields (meta_title, meta_description, content)
                                                       - - The Statamic API supports PUT/PATCH for entry updates
                                                         - - All writes must preserve existing locale data (only modify `default` locale)
                                                          
                                                           - ### 2.4 Data Layer (External SEO Data)
                                                          
                                                           - The agent's eyes and ears in the real world. Skills never call external APIs directly — they query the Data Layer, which routes to the best available provider, caches responses, and tracks costs.
                                                          
                                                           - **Connected data sources:**
                                                           - | Source | Data | Priority |
                                                           - |--------|------|----------|
                                                           - | Google Search Console | Rankings, impressions, CTR, index coverage | Phase 1 (free, first-party) |
                                                           - | Google Analytics 4 | Traffic, engagement, conversions, demographics | Phase 2 |
                                                           - | DataForSEO | Keyword metrics, SERP data, backlinks, AI search presence | Phase 1 (already connected) |
                                                           - | Ahrefs | Backlink profiles, DR, content gaps, competitor analysis | Phase 3 |
                                                           - | SEMrush | Keyword gaps, site audit, traffic analytics | Optional |
                                                          
                                                           - **See [Data Integrations](data-integrations.md) for full specification, unified interface, caching strategy, and cost management.**
                                                          
                                                           - ### 2.5 Slack Integration Layer
                                                          
                                                           - The sole interface between humans and the agent. Handles three flows: inbound commands, outbound notifications, and approval workflows.
                                                          
                                                           - **See [Slack Integration](slack-integration.md) for full specification.**
                                                          
                                                           - ### 2.6 Task Queue & Scheduler
                                                          
                                                           - Manages async task execution, scheduling, and retry logic.
                                                          
                                                           - **Queue types:**
                                                           - - **Immediate**: Triggered by Slack commands or API calls
                                                             - - **Scheduled**: Recurring audits (weekly meta tag scan, monthly content review)
                                                               - - **Background**: Low-priority batch operations (bulk re-embedding, link recrawl)
                                                                
                                                                 - **Implementation options:**
                                                                 - - Laravel Queue (Redis/database driver) for job management
                                                                   - - Laravel Scheduler for recurring tasks
                                                                     - - Database-backed task history for audit trail
                                                                      
                                                                       - ---

                                                                       ## 3. Data Flow

                                                                       ### 3.1 Inbound Task Flow (Slack → Agent → CMS)

                                                                       ```
                                                                       User sends Slack command
                                                                              │
                                                                              ▼
                                                                       Slack webhook receives message
                                                                              │
                                                                              ▼
                                                                       Agent Controller parses intent
                                                                              │
                                                                              ▼
                                                                       Controller queries Knowledge Base (RAG)
                                                                       for context about affected pages
                                                                              │
                                                                              ▼
                                                                       Route to appropriate Engine
                                                                              │
                                                                              ▼
                                                                       Engine generates proposed changes
                                                                              │
                                                                              ▼
                                                                       Self-Reflection: Agent evaluates its own output
                                                                         ├─ Does this make sense given page intent?
                                                                         ├─ Will this create keyword cannibalization?
                                                                         ├─ Does the tone match our brand voice?
                                                                         └─ Are there unintended side effects?
                                                                              │
                                                                              ▼
                                                                       Autonomy Check
                                                                         ├─ LOW RISK → Auto-execute, notify Slack
                                                                         └─ HIGH RISK → Queue for approval, notify Slack
                                                                              │
                                                                              ▼
                                                                       On approval (or auto): Write to Statamic CMS
                                                                              │
                                                                              ▼
                                                                       Update Knowledge Base embeddings
                                                                              │
                                                                              ▼
                                                                       Confirm in Slack with summary
                                                                       ```

                                                                       ### 3.2 Knowledge Base Sync Flow

                                                                       ```
                                                                       Statamic CMS (source of truth)
                                                                              │
                                                                              ▼ (Scheduled sync or webhook trigger)
                                                                       Fetch entries via Private API
                                                                              │
                                                                              ▼
                                                                       Extract text + metadata
                                                                              │
                                                                              ▼
                                                                       Chunk content (Recursive Text Splitter)
                                                                              │
                                                                              ▼
                                                                       Generate embeddings (OpenAI)
                                                                              │
                                                                              ▼
                                                                       Upsert to Supabase pgvector
                                                                              │
                                                                              ▼
                                                                       Mark stale embeddings for cleanup
                                                                       ```

                                                                       ---

                                                                       ## 4. Tech Stack Recommendations

                                                                       The dev should evaluate these options. The recommendation leans toward a Laravel-native approach since the CMS is already Statamic (Laravel-based).

                                                                       ### Option A: Laravel-Native (Recommended)

                                                                       Everything in one Laravel application. Simplest deployment, tightest CMS integration.

                                                                       | Layer | Technology | Notes |
                                                                       |-------|-----------|-------|
                                                                       | Application | Laravel 11+ | Same framework as Statamic |
                                                                       | AI / LLM | OpenAI API (GPT-4o / GPT-4.1) | Via official PHP SDK or HTTP client |
                                                                       | Embeddings | OpenAI text-embedding-3-small | Upgrade from ada-002 for better performance |
                                                                       | Vector DB | Supabase pgvector | Already set up, keep using it |
                                                                       | Queue | Laravel Queue (Redis) | For async task processing |
                                                                       | Scheduler | Laravel Scheduler | For recurring audits |
                                                                       | Slack | Slack Bolt for PHP or HTTP webhooks | For bot interaction |
                                                                       | Cache | Redis | For rate limiting, session state |
                                                                       | CMS Integration | Direct Statamic API or Eloquent | Since it's the same Laravel app, could be a Statamic addon |

                                                                       **Pros:** Single codebase, native Statamic integration, simpler deployment, PHP team already knows Laravel.
                                                                       **Cons:** PHP isn't the strongest for AI/ML libraries; relies on API calls for all AI operations.

                                                                       ### Option B: Laravel + Python Microservice

                                                                       Laravel handles CMS and Slack. A Python service handles AI-heavy operations.

                                                                       | Layer | Technology | Notes |
                                                                       |-------|-----------|-------|
                                                                       | Orchestration | Laravel 11+ | Task routing, Slack, CMS writes |
                                                                       | AI Service | Python (FastAPI) | LLM calls, RAG, embeddings, content generation |
                                                                       | Communication | REST API or message queue | Between Laravel and Python |
                                                                       | Everything else | Same as Option A | — |

                                                                       **Pros:** Access to Python AI ecosystem (LangChain, LlamaIndex, etc.), better for complex RAG pipelines.
                                                                       **Cons:** Two services to deploy and maintain, added network latency, more complex architecture.

                                                                       ### Option C: Statamic Addon

                                                                       Build the agent as a native Statamic addon (Laravel package). Deepest CMS integration — direct access to entries, fieldtypes, events.

                                                                       **Pros:** Can hook into Statamic lifecycle events (entry.saving, entry.saved), native CP integration, access to all content without API calls.
                                                                       **Cons:** Tightly coupled to Statamic version, harder to test independently, may complicate Statamic upgrades.

                                                                       ### Recommendation

                                                                       **Start with Option A** (Laravel-native). It's the simplest path, and since all AI operations go through API calls anyway (OpenAI, Supabase), PHP handles them fine. If the RAG pipeline gets complex enough to warrant Python, refactor the AI layer into a microservice later (Option B) — the task routing and Slack layers stay in Laravel either way.

                                                                       If the dev is comfortable with Statamic addon development, **Option C** is the highest-leverage choice for CMS integration but requires deeper Statamic expertise.

                                                                       ---

                                                                       ## 5. Infrastructure

                                                                       ### Current Infrastructure
                                                                       - **CMS**: Statamic on existing hosting
                                                                       - - **Vector DB**: Supabase (project `nbeancschpzqblzvmmjs`)
                                                                         - - **n8n**: Self-hosted at `n8n.keylearning.site` (used for Phase 0, will be replaced by native Laravel)
                                                                           - - **AI**: OpenAI API (existing account)
                                                                            
                                                                             - ### Required Infrastructure (New)
                                                                             - - **Redis**: For Laravel Queue + caching (can use managed Redis or self-host)
                                                                               - - **Application Server**: For the Laravel agent app (could be same server as Statamic or separate)
                                                                                 - - **Slack App**: New Slack app with bot token, slash commands, and event subscriptions
                                                                                  
                                                                                   - ### Environment Variables
                                                                                   - ```env
                                                                                     # OpenAI
                                                                                     OPENAI_API_KEY=sk-...
                                                                                     OPENAI_EMBEDDING_MODEL=text-embedding-3-small
                                                                                     OPENAI_CHAT_MODEL=gpt-4o

                                                                                     # Supabase
                                                                                     SUPABASE_URL=https://nbeancschpzqblzvmmjs.supabase.co
                                                                                     SUPABASE_KEY=...
                                                                                     SUPABASE_TABLE=documents

                                                                                     # Statamic CMS
                                                                                     STATAMIC_API_URL=https://internationaldriversassociation.com/api/private
                                                                                     STATAMIC_API_TOKEN=...

                                                                                     # Slack
                                                                                     SLACK_BOT_TOKEN=xoxb-...
                                                                                     SLACK_SIGNING_SECRET=...
                                                                                     SLACK_CHANNEL_ID=...

                                                                                     # Queue
                                                                                     QUEUE_CONNECTION=redis
                                                                                     REDIS_HOST=127.0.0.1
                                                                                     ```

                                                                                     ---

                                                                                     ## 6. Security Considerations

                                                                                     - **API tokens** must be stored in environment variables, never in code
                                                                                     - - **Slack request verification** — validate all incoming webhooks using Slack signing secret
                                                                                       - - **Rate limiting** — respect OpenAI and Statamic API rate limits; implement exponential backoff
                                                                                         - - **Audit trail** — log every CMS write with timestamp, user/trigger, before/after values
                                                                                           - - **Rollback capability** — store previous values before any CMS write for instant rollback
                                                                                             - - **Locale safety** — only modify `default` (English) locale; never touch translated content
