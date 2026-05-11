# Agent Behavior & Autonomy

## 1. The Self-Reflection Loop

The IDA AI SEO Engine isn't a simple "input to output" system. Every task runs through a structured reasoning loop that mirrors how a senior SEO specialist would think.

### The Loop

```
1. UNDERSTAND - What exactly is being asked? What pages are affected?
2. RESEARCH   - Query the Knowledge Base for context
3. PLAN       - Generate a proposed action plan
4. EXECUTE    - Generate the actual outputs
5. REFLECT    - Critically evaluate the output against SEO best practices
6. REFINE     - If reflection found issues, loop back to step 4
7. FINALIZE   - Output passes reflection, proceed to autonomy check
```

### Why Self-Reflection Matters

Without reflection, an AI agent will confidently produce outputs that look correct superficially but contain subtle SEO mistakes:

- **Cannibalization**: Optimizing two pages for the same keyword without realizing they compete
- - **Intent mismatch**: Writing a transactional meta description for an informational page
  - - **Over-optimization**: Stuffing keywords unnaturally because the raw data says to
    - - **Context blindness**: Suggesting internal links to irrelevant pages because embeddings are close
     
      - The reflection step catches these by forcing the agent to evaluate its work against the broader site context stored in the Knowledge Base.
     
      - ### Implementation Pattern
     
      - ```
        function executeWithReflection(task, maxIterations = 3) {
            context = knowledgeBase.query(task.affectedPages)
            plan = llm.plan(task, context)

            for (i = 0; i < maxIterations; i++) {
                output = llm.execute(plan, context)
                reflection = llm.reflect({
                    task, output, context,
                    criteria: ["search_intent_alignment", "no_keyword_cannibalization",
                               "brand_voice_consistency", "no_side_effects", "factual_accuracy"]
                })
                if (reflection.approved) return output
                plan = llm.refine(plan, reflection.issues)
            }
            return escalateToHuman(task, output, reflection)
        }
        ```

        ---

        ## 2. Autonomy Tiers

        Not all changes carry the same risk. The agent uses a two-tier model.

        ### Tier 1: Auto-Execute (Low Risk)

        These changes are applied immediately. The agent notifies Slack after execution.

        | Action | Why Low Risk |
        |--------|-------------|
        | Meta title updates (within template) | Formulaic, follows existing patterns |
        | Meta description rewrites | No structural impact, easily reversible |
        | Internal link suggestions (report only) | No CMS change, just recommendations |
        | SEO audit reports | Read-only analysis |
        | Knowledge Base re-sync | No CMS impact |
        | Keyword research and analysis | Read-only |

        **Guardrails even for auto-execute:**
        - Changes must pass the self-reflection loop
        - - Every change is logged with before/after values
          - - Batch limits: max 50 pages per auto-execution run
            - - Rollback available within 24 hours
              - - Slack notification sent with summary of all changes
               
                - ### Tier 2: Approval Required (High Risk)
               
                - These changes are drafted, then queued for human approval via Slack.
               
                - | Action | Why High Risk |
                - |--------|-------------|
                - | Content body rewrites or optimization | Significant content change |
                - | New content creation (blog posts, guides) | New pages on the site |
                - | Bulk changes affecting 50+ pages | Wide blast radius |
                - | Internal link implementation (actual insertion) | Modifies page content |
                - | Schema markup changes | Technical SEO impact |
                - | URL or slug modifications | Redirect implications |
                - | Any change flagged by self-reflection | Agent is uncertain |
               
                - **Approval workflow:**
                - 1. Agent drafts the change
                  2. 2. Posts to Slack with: what is changing, why, affected pages, before/after preview
                     3. 3. Human responds: approve, reject, or modify
                        4. 4. On approve, agent executes and confirms
                           5. 5. On reject, agent logs the rejection reason for learning
                              6. 6. No response within 48 hours, auto-remind in Slack
                                
                                 7. ### Risk Classification Logic
                                
                                 8. ```
                                    function classifyRisk(action) {
                                        if (action.type === "content_body_change") return "high"
                                        if (action.type === "new_content") return "high"
                                        if (action.type === "url_change") return "high"
                                        if (action.type === "schema_change") return "high"
                                        if (action.affectedPages > 50) return "high"
                                        if (action.reflectionConfidence < 0.8) return "high"
                                        return "low"
                                    }
                                    ```

                                    ---

                                    ## 3. Agent Capabilities

                                    ### 3.1 Bulk Meta Tag Optimization

                                    Analyzes and rewrites meta titles and descriptions across all pages following IDA templates and SEO best practices.

                                    **Process:**
                                    1. Scan all pages for meta tag issues (too long, too short, missing, duplicate)
                                    2. 2. Generate optimized alternatives using page content + search intent
                                       3. 3. Self-reflect: check for cannibalization across pages
                                          4. 4. Auto-execute in batches of 50 (Tier 1)
                                             5. 5. Report results to Slack with before/after comparisons
                                               
                                                6. **IDA Meta Tag Templates:**
                                                7. - Countries: "Get an International Driver's License for {Country} | IDA"
                                                   - - Driving Guides: "{Country} Driving Guide: Rules, Tips and Requirements | IDA"
                                                     - - Blog: "{Post Title} | International Drivers Association"
                                                      
                                                       - ### 3.2 Internal Linking
                                                      
                                                       - Analyzes the site internal link structure and identifies opportunities to strengthen topical authority.
                                                      
                                                       - **Link strategy rules:**
                                                       - - Every blog post should link to its parent country page
                                                         - - Every country page should link to its driving guide (and vice versa)
                                                           - - Topically related blog posts should cross-link
                                                             - - Anchor text must be varied, never use the same anchor for the same target
                                                               - - Avoid orphan pages (every page gets at least 3 internal links)
                                                                 - - Follow hub-and-spoke model: collection pages are hubs, blog posts are spokes
                                                                  
                                                                   - ### 3.3 Content Optimization
                                                                  
                                                                   - Analyzes existing content for SEO weaknesses and rewrites sections to improve rankings.
                                                                  
                                                                   - **Optimization criteria:**
                                                                   - - Target word count: Countries (2000+), Driving Guides (1500+), Blog (1200+)
                                                                     - - Heading hierarchy: H1, H2, H3 (no skipping levels)
                                                                       - - Keyword placement: in H1, first paragraph, at least one H2, meta description
                                                                         - - Readability: Flesch reading ease score 60+
                                                                           - - Freshness: flag content not updated in 6+ months
                                                                            
                                                                             - ### 3.4 Content Creation
                                                                            
                                                                             - Generates entirely new pages based on keyword research and content gap analysis.
                                                                            
                                                                             - **Two-stage approval:** 1. Brief approval, then 2. Final content approval
                                                                            
                                                                             - **Content types:** Blog posts (1200-2000 words), driving guides (1500+), FAQ sections, supporting content for topical clusters.
                                                                            
                                                                             - ---

                                                                             ## 4. Error Handling and Edge Cases

                                                                             ### When the Agent Gets Stuck
                                                                             After 3 reflection iterations without approval: stops, posts best attempt + notes to Slack, asks human for guidance.

                                                                             ### When the CMS Write Fails
                                                                             Retry 3x with exponential backoff. On persistent failure, notify Slack. Never leave partial writes.

                                                                             ### When Conflicting Instructions Arrive
                                                                             Queue sequentially. Manual commands take priority over scheduled tasks. Show conflicts to human.

                                                                             ### Rate Limits
                                                                             - OpenAI: Token tracking and request queuing
                                                                             - - Statamic API: Max 5 concurrent requests, 2-second delay between writes
                                                                               - - Slack API: Respect rate limit headers, queue messages if throttled
