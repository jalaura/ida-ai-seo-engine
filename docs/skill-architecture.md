# Skill Architecture -- Extensible Expert Modules

## 1. Why a Skill System

The IDA AI SEO Engine isn't a fixed-feature tool. It's a **platform** that hosts expert modules ("Skills") which can be added, updated, or replaced over time without rebuilding the core agent.

Today the agent ships with 4 built-in engines (meta tags, internal linking, content optimization, content creation). Tomorrow you might need a Backlink Audit Expert, an A/B Testing Manager, a Schema Markup Specialist, or a Competitor Intelligence Analyst. The Skill Architecture makes this possible by defining a standard interface that any new expert module plugs into.

Think of it like hiring specialists for your SEO team -- each one has deep domain expertise, their own playbooks and frameworks, but they all report to the same manager (the Agent Controller) and communicate through the same channel (Slack).

---

## 2. How Skills Work

### 2.1 Skill Definition

A Skill is a self-contained module that includes:

```
skills/
  backlink-expert/
    skill.json              # Metadata, triggers, capabilities
    system-prompt.md        # The expert's "brain" -- instructions, methodology, rules
    frameworks/             # Domain-specific frameworks and playbooks
      toxic-link-criteria.md
      outreach-templates.md
    actions/                # Callable actions this skill can perform
      audit.js            # Run a backlink audit
      disavow.js          # Generate disavow file
      opportunities.js    # Find link building opportunities
```

### 2.2 skill.json -- Skill Manifest

Every skill declares what it can do, what triggers it, and what permissions it needs:

```json
{
  "id": "backlink-expert",
  "name": "Backlink Audit Expert",
  "version": "1.0.0",
  "description": "Analyzes backlink profiles, identifies toxic links, finds link building opportunities, and manages disavow files.",

  "triggers": {
    "commands": ["/seo backlinks", "/seo disavow", "/seo link-audit"],
    "keywords": ["backlink", "referring domain", "toxic link", "disavow", "link building", "domain rating", "anchor text distribution"],
    "scheduled": ["weekly-backlink-check"]
  },

  "capabilities": {
    "read": ["knowledge-base", "statamic-api"],
    "write": ["knowledge-base"],
    "external_apis": ["ahrefs", "semrush"],
    "slack": ["send-message", "request-approval", "upload-file"]
  },

  "autonomy": {
    "auto_execute": ["audit", "report"],
    "requires_approval": ["disavow-submit", "outreach-send"]
  },

  "frameworks": ["toxic-link-criteria", "outreach-templates"],

  "actions": [
    {
      "id": "audit",
      "name": "Run Backlink Audit",
      "description": "Analyze the full backlink profile and generate a health report",
      "input": { "domain": "string", "depth": "quick|full" },
      "output": "BacklinkAuditReport"
    },
    {
      "id": "disavow",
      "name": "Generate Disavow File",
      "description": "Create a disavow file from toxic link analysis",
      "input": { "audit_id": "string", "threshold": "number" },
      "output": "DisavowFile"
    }
  ]
}
```

### 2.3 System Prompt -- The Expert's Brain

Each skill has a `system-prompt.md` that defines the expert persona, methodology, and decision-making rules. This is injected into the LLM context when the skill is activated.

Example structure:
```markdown
# Backlink Audit Expert

You are a senior backlink audit strategist. You analyze backlink profiles
using data from Ahrefs/SEMrush and apply the following methodology...

## Decision Framework
1. Pull the full referring domain list
2. Score each domain on: DR, spam score, relevance, anchor text...
3. Classify: healthy, suspicious, toxic
4. For toxic links: recommend disavow vs. outreach for removal
...

## Rules
- Never recommend disavowing links from DR 50+ domains without human review
- Always check if a "toxic" link is actually a legitimate media mention
- Flag anchor text ratios above 60% exact-match as suspicious
...
```

### 2.4 Frameworks -- Domain Playbooks

Frameworks are reference documents that encode expert knowledge. They're loaded into context alongside the system prompt when the skill needs deep domain guidance.

This is where methodologies like Koray Gubur's Topical Authority framework live. See Section 4 below for the full Content Framework system.

---

## 3. Skill Lifecycle

### 3.1 Registration

When the agent starts (or when a new skill is added), the Agent Controller:

1. Scans the `skills/` directory for `skill.json` files
2. 2. Registers each skill's triggers (commands, keywords, schedules)
   3. 3. Validates required permissions and external API credentials
      4. 4. Logs the skill as "available" in the skill registry
        
         5. ### 3.2 Activation
        
         6. When a task arrives (via Slack or scheduler), the Agent Controller:
        
         7. 1. Matches the task against registered triggers (command match or keyword detection)
            2. 2. If multiple skills match, selects the most specific one (or asks the user)
               3. 3. Loads the skill's system prompt + relevant frameworks into the LLM context
                  4. 4. Routes the task to the skill's appropriate action
                     5. 5. The skill executes within the standard self-reflection loop
                       
                        6. ### 3.3 Execution Flow
                       
                        7. ```
                           Task arrives (Slack command / scheduled / internal trigger)
                                  |
                                  v
                           Agent Controller matches task to skill
                                  |
                                  v
                           Load skill context:
                             - system-prompt.md (expert persona and methodology)
                             - Relevant frameworks (domain playbooks)
                             - Action handler (the specific operation)
                                  |
                                  v
                           Standard self-reflection loop:
                             UNDERSTAND - RESEARCH - PLAN - EXECUTE - REFLECT - REFINE
                                  |
                                  v
                           Autonomy check (per skill.json config):
                             - auto_execute actions: run and notify
                             - requires_approval actions: queue for Slack approval
                                  |
                                  v
                           Output + Slack notification
                           ```

                           ### 3.4 Adding a New Skill

                           To add a new expert module:

                           1. Create a new directory under `skills/` with the skill files
                           2. 2. Define `skill.json` with triggers, capabilities, and actions
                              3. 3. Write `system-prompt.md` with the expert methodology
                                 4. 4. Add any frameworks to `frameworks/`
                                    5. 5. Implement action handlers
                                       6. 6. Restart the agent (or hot-reload if supported)
                                         
                                          7. No changes to the Agent Controller, Slack integration, or any other skill required.
                                         
                                          8. ---
                                         
                                          9. ## 4. Content Framework System
                                         
                                          10. ### 4.1 What Are Content Frameworks?
                                         
                                          11. Content Frameworks are structured methodology documents that the agent loads into its context when making content decisions. They ensure the agent doesn't just write "good content" -- it follows a specific, proven SEO methodology.
                                         
                                          12. Frameworks are stored as markdown files and loaded dynamically. You can swap, update, or combine frameworks without changing any code.
                                         
                                          13. ### 4.2 Framework Storage
                                         
                                          14. ```
                                              frameworks/
                                                koray-topical-authority/
                                                  framework.json           # Metadata and when to load
                                                  core-philosophy.md       # EAV thinking, network effects, source context
                                                  topical-map-design.md    # 5-pillar architecture, core vs. outer sections
                                                  content-writing-rules.md # 41 authorship rules, question H2, 40-word answers
                                                  internal-linking.md      # Progressive intent linking, anchor text rules
                                                  audit-checklist.md       # 6-month reconfiguration process

                                                entity-seo/
                                                  framework.json
                                                  knowledge-graph.md       # Entity Home, Kalicube method
                                                  schema-markup.md         # @id, sameAs, about, mentions
                                                  ai-search-readiness.md   # GEO/AEO optimization

                                                ida-brand-voice/
                                                  framework.json
                                                  tone-guidelines.md       # How IDA sounds (helpful, authoritative, traveler-friendly)
                                                  terminology.md           # "International Driver's License" not "International Driving Permit"
                                                  templates.md             # Meta tag templates, heading patterns, CTA style

                                                local-seo/
                                                  framework.json
                                                  gbp-optimization.md
                                                  citation-strategy.md
                                                  review-management.md
                                              ```

                                              ### 4.3 framework.json

                                              ```json
                                              {
                                                "id": "koray-topical-authority",
                                                "name": "Koray Gubur's Topical Authority Framework",
                                                "version": "2026.1",
                                                "description": "Complete semantic SEO and topical authority methodology. EAV framework, 5-pillar topical maps, 41 authorship rules, progressive intent linking.",

                                                "applies_to": ["content-creation", "content-optimization", "internal-linking"],

                                                "load_when": {
                                                  "always": ["core-philosophy.md"],
                                                  "on_action": {
                                                    "create_content": ["topical-map-design.md", "content-writing-rules.md"],
                                                    "optimize_content": ["content-writing-rules.md", "audit-checklist.md"],
                                                    "internal_linking": ["internal-linking.md", "topical-map-design.md"]
                                                  }
                                                },

                                                "priority": 1
                                              }
                                              ```

                                              ### 4.4 How Frameworks Get Used

                                              When the agent needs to create or optimize content, the Agent Controller:

                                              1. Checks which frameworks are registered for the current action
                                              2. 2. Loads the relevant framework files into the LLM context (based on `load_when` rules)
                                                 3. 3. The framework instructions become part of the agent's reasoning context
                                                    4. 4. The self-reflection step evaluates output against framework criteria
                                                      
                                                       5. Example -- creating a new blog post:
                                                      
                                                       6. ```
                                                          LLM Context =
                                                            Agent system prompt (base personality, safety rules)
                                                            + Skill system prompt (Content Creator expert)
                                                            + Framework: Koray core-philosophy.md (EAV thinking, source context)
                                                            + Framework: Koray content-writing-rules.md (question H2, 40-word answers)
                                                            + Framework: Koray topical-map-design.md (where does this page fit?)
                                                            + Framework: IDA brand-voice/tone-guidelines.md (how IDA sounds)
                                                            + Framework: IDA brand-voice/terminology.md (correct terms)
                                                            + RAG context (related existing pages from Knowledge Base)
                                                            + Task details (keyword, brief, target audience)
                                                          ```

                                                          ### 4.5 Koray Framework Integration -- Specific Rules

                                                          When the Koray Topical Authority framework is active, the agent enforces these rules in its self-reflection:

                                                          **Content Structure:**
                                                          - Every page covers exactly one macro context (no topic mixing)
                                                          - - H2 headings phrased as user questions
                                                            - - ~40-word direct, factual answer immediately below each H2
                                                              - - No vague or opinion-based language -- definitive, factual statements
                                                                - - Content completeness measured by EAV coverage (entities, attributes, values)
                                                                 
                                                                  - **Site Architecture:**
                                                                  - - All content must serve the semantic content network (no "content for content's sake")
                                                                    - - Core section = commercial/money pages, outer section = informational/trust-building
                                                                      - - Hub-and-spoke: homepage to pillar pages to cluster pages
                                                                        - - Cross-silo links only where entities share attributes
                                                                         
                                                                          - **Internal Linking:**
                                                                          - - Progressive intent linking: informational to commercial to transactional
                                                                            - - Anchor text must match target page's title and macro context
                                                                              - - Contextual bridges required when expanding into adjacent topics (minimum 3 connections)
                                                                               
                                                                                - **Quality Standards:**
                                                                                - - No page published without a topical map purpose
                                                                                  - - Single thin page can dilute entire semantic network authority
                                                                                    - - Content reconfiguration audit every 6 months
                                                                                     
                                                                                      - ### 4.6 IDA Brand Voice Framework
                                                                                     
                                                                                      - This framework ensures all agent output sounds like IDA -- not generic AI content:
                                                                                     
                                                                                      - **Tone:** Helpful, authoritative, traveler-friendly. Written for international travelers who may not be native English speakers. Clear and direct, avoiding jargon.
                                                                                     
                                                                                      - **Terminology:**
                                                                                      - - Always: "International Driver's License" (never "International Driving Permit" or "IDP")
                                                                                        - - Always: "International Drivers Association" or "IDA" (never abbreviate differently)
                                                                                          - - Country names follow IDA's established patterns (check existing pages)
                                                                                           
                                                                                            - **Templates:**
                                                                                            - - Meta titles follow collection-specific patterns (defined in architecture doc)
                                                                                              - - CTAs: "Get Your International Driver's License" (not "Apply Now" or "Buy Today")
                                                                                                - - Headings: factual and specific (not clickbait)
                                                                                                 
                                                                                                  - ---

                                                                                                  ## 5. Planned Skills (Future Roadmap)

                                                                                                  These are skills that can be added to the platform as needs grow:

                                                                                                  | Skill | Description | Priority |
                                                                                                  |-------|-------------|----------|
                                                                                                  | Backlink Expert | Audit backlink profiles, identify toxic links, generate disavow files, find link building opportunities | High |
                                                                                                  | A/B Test Manager | Design and manage SEO A/B tests (title tags, meta descriptions, content variations), track results, recommend winners | Medium |
                                                                                                  | Schema Specialist | Audit and generate structured data (JSON-LD) for all page types, validate against Google's requirements | High |
                                                                                                  | Competitor Intelligence | Monitor competitor rankings, content changes, and backlink strategies; identify opportunities | Medium |
                                                                                                  | Technical SEO Auditor | Crawl site for technical issues (Core Web Vitals, mobile-first, crawlability, indexation), generate fix recommendations | High |
                                                                                                  | Content Calendar | Plan and schedule content publication based on keyword gaps, seasonal trends, and topical authority needs | Medium |
                                                                                                  | GBP Manager | Manage Google Business Profile: posts, Q&A, review responses, category optimization | Low |
                                                                                                  | Reporting Engine | Generate automated weekly/monthly SEO reports with visualizations and trend analysis | Medium |
                                                                                                  | Migration Manager | Plan and execute CMS migrations with redirect mapping, pre/post checks, and monitoring | Low |

                                                                                                  ### Adding a New Skill -- Developer Checklist

                                                                                                  1. Create `skills/{skill-id}/skill.json` with triggers and capabilities
                                                                                                  2. 2. Write `skills/{skill-id}/system-prompt.md` with expert methodology
                                                                                                     3. 3. Add any frameworks to `frameworks/` (if the skill uses domain playbooks)
                                                                                                        4. 4. Implement action handlers in `skills/{skill-id}/actions/`
                                                                                                           5. 5. Configure external API credentials (if needed) in environment
                                                                                                              6. 6. Add Slack command registrations (if new slash commands needed)
                                                                                                                 7. 7. Test: trigger the skill via Slack, verify reflection loop, check output quality
                                                                                                                    8. 8. Deploy: restart agent or hot-reload skill registry
                                                                                                                      
                                                                                                                       9. ---
                                                                                                                      
                                                                                                                       10. ## 6. Skill Communication
                                                                                                                      
                                                                                                                       11. Skills can call each other when a task requires cross-domain expertise:
                                                                                                                      
                                                                                                                       12. ```
                                                                                                                           Content Creator skill is writing a new blog post
                                                                                                                             calls Internal Linking skill: "Find pages to link to/from this new post"
                                                                                                                             calls Schema Specialist skill: "Generate Article schema for this post"
                                                                                                                             calls Meta Tag Engine: "Generate meta title and description"
                                                                                                                             Each sub-call runs its own reflection loop
                                                                                                                             Results are combined into the final output
                                                                                                                           ```
                                                                                                                           
                                                                                                                           ### Communication Rules
                                                                                                                           - Skills communicate through the Agent Controller (never directly)
                                                                                                                           - - Each sub-call is logged as a separate task for audit purposes
                                                                                                                             - - If a sub-call requires approval, the parent task waits
                                                                                                                               - - Circular dependencies are detected and blocked
                                                                                                                                 - - Skills can declare dependencies in `skill.json` (e.g., Content Creator depends on Internal Linking)
