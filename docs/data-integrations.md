# Data Integrations -- External SEO Data Sources

## 1. Why External Data Matters

The Knowledge Base (Supabase pgvector) stores what's ON the site -- content, metadata, embeddings. But to make smart SEO decisions, the agent also needs to know what's happening AROUND the site: how pages rank, what traffic they get, who links to them, what competitors are doing, and what people are searching for.

External data sources fill this gap. They feed the agent with real-world performance data so it can make decisions grounded in actual search behavior, not just content analysis.

---

## 2. Integration Architecture

```
AGENT CONTROLLER
  Skills query data through the Data Layer -- they never
  call external APIs directly. This keeps skills portable
  and makes it easy to swap data providers.
       |
       v
DATA LAYER (Unified query interface)
  dataLayer.getKeywordMetrics("international drivers license")
  dataLayer.getBacklinks("internationaldriversassociation.com")
  dataLayer.getPageTraffic("/andorra-driving-guide")
       |
       v
  GSC  |  GA4  |  DataForSEO  |  Ahrefs  |  SEMrush
```

### Design Principles

1. **Data Layer abstraction** -- Skills never call external APIs directly. They call the Data Layer, which routes to the right provider.
2. 2. **Cache everything** -- External API calls cost money and have rate limits. All responses are cached with configurable TTL.
   3. 3. **Graceful degradation** -- If an API is down, the agent still functions with available data.
      4. 4. **Cost awareness** -- Each API call has a cost. The Data Layer tracks usage and enforces budgets.
        
         5. ---
        
         6. ## 3. Data Sources
        
         7. ### 3.1 Google Search Console (GSC)
        
         8. **What it provides:** First-party search performance data directly from Google.
        
         9. | Data Point | Use Case |
         10. |-----------|----------|
         11. | Impressions per page/query | Which pages are visible in search |
         12. | Clicks per page/query | Which pages get traffic from search |
         13. | Average position per query | Current ranking for target keywords |
         14. | CTR (click-through rate) | How compelling the meta tags are |
         15. | Index coverage | Which pages are indexed |
         16. | Core Web Vitals | Page experience signals |
        
         17. **Agent usage:** Low CTR + high impressions = meta tags need rewriting. Pages losing position = content needs refreshing. Low impressions + good content = not enough internal links.
        
         18. **Integration:** OAuth 2.0 service account, 25,000 queries/day, 2-3 day data delay. Env vars: `GSC_SERVICE_ACCOUNT_JSON`, `GSC_PROPERTY_URL`
        
         19. ### 3.2 Google Analytics 4 (GA4)
        
         20. **What it provides:** User behavior data -- what happens after someone lands on the site.
        
         21. | Data Point | Use Case |
         22. |-----------|----------|
         23. | Sessions per page | Total traffic (all sources) |
         24. | Engagement rate | Are users reading the content? |
         25. | Average engagement time | Time spent on each page |
         26. | Bounce rate | Users who leave immediately |
         27. | Conversion events | License applications, form submissions |
         28. | Traffic sources | Organic vs. paid vs. referral vs. direct |
        
         29. **Agent usage:** High traffic + low engagement = intent mismatch. High engagement + low traffic = needs better SEO. Identify topics that drive conversions.
        
         30. **Integration:** OAuth 2.0 service account, 10,000 requests/day. Env vars: `GA4_PROPERTY_ID`, `GA4_SERVICE_ACCOUNT_JSON`
        
         31. ### 3.3 DataForSEO
        
         32. **What it provides:** Real-time SERP data, keyword metrics, backlinks, on-page analysis, AI optimization. Most versatile single API for SEO data.
        
         33. | Endpoint Group | Data Points | Use Case |
         34. |---------------|-------------|----------|
         35. | SERP API | Live SERP results, featured snippets, PAA | Competitor analysis |
         36. | Keyword Data | Search volume, CPC, competition, trends | Keyword research |
         37. | DataForSEO Labs | Keyword difficulty, domain rank | Competitive intelligence |
         38. | Backlinks API | Backlink profiles, referring domains | Backlink auditing |
         39. | On-Page API | Content parsing, Lighthouse scores | Technical SEO audits |
         40. | AI Optimization | LLM mentions, ChatGPT scraper | GEO/AEO optimization |
        
         41. **Note:** Already connected to this workspace via MCP connector.
        
         42. **Integration:** HTTP Basic Auth, credit-based. Env vars: `DATAFORSEO_LOGIN`, `DATAFORSEO_PASSWORD`
        
         43. ### 3.4 Ahrefs
        
         44. **What it provides:** Industry standard for backlink analysis plus keyword and content tools.
        
         45. | Data Point | Use Case |
         46. |-----------|----------|
         47. | Domain Rating (DR) | Overall domain authority |
         48. | Backlink profile | All links pointing to IDA |
         49. | Referring domains | Unique domains linking to IDA |
         50. | Anchor text distribution | How other sites describe IDA |
         51. | Keyword rankings | What IDA ranks for + position history |
         52. | Content explorer | Content gaps and opportunities |
         53. | Broken backlinks | Links pointing to 404 pages |
        
         54. **Integration:** API token (Bearer), plan-based rate limits. Env var: `AHREFS_API_TOKEN`
        
         55. ### 3.5 SEMrush (Optional)
        
         56. **Recommendation:** Start with DataForSEO + Ahrefs. SEMrush is optional for second opinion on keyword data or competitive analysis.
        
         57. ---
        
         58. ## 4. Data Layer Implementation
        
         59. ### 4.1 Unified Interface
        
         60. ```php
             interface DataLayerInterface
             {
                 public function getKeywordMetrics(string $keyword, string $location = 'US'): KeywordMetrics;
                 public function getKeywordSuggestions(string $seed, int $limit = 50): array;
                 public function getKeywordDifficulty(array $keywords): array;
                 public function getRankings(string $domain, array $keywords = []): array;
                 public function getPositionHistory(string $url, string $keyword, int $months = 6): array;
                 public function getSearchPerformance(string $url, DateRange $range): SearchPerformance;
                 public function getIndexCoverage(): IndexCoverage;
                 public function getPageTraffic(string $url, DateRange $range): TrafficData;
                 public function getConversions(string $url, DateRange $range): ConversionData;
                 public function getBacklinkProfile(string $domain): BacklinkProfile;
                 public function getReferringDomains(string $domain, int $limit = 100): array;
                 public function getAnchorTextDistribution(string $domain): array;
                 public function getSerpResults(string $keyword, string $location = 'US'): SerpResults;
                 public function getLlmMentions(string $brand, array $keywords): LlmMentionData;
                 public function getAiOverviewPresence(string $keyword): AiOverviewData;
                 public function getCompetitorContent(string $keyword, int $topN = 10): array;
                 public function getContentGaps(string $domain, array $competitors): array;
             }
             ```

             ### 4.2 Provider Routing

             ```php
             class DataLayer implements DataLayerInterface
             {
                 private array $routing = [
                     'keyword_metrics'     => ['dataforseo', 'semrush', 'ahrefs'],
                     'rankings'            => ['gsc', 'dataforseo', 'ahrefs', 'semrush'],
                     'backlinks'           => ['ahrefs', 'dataforseo', 'semrush'],
                     'traffic'             => ['ga4'],
                     'serp'                => ['dataforseo'],
                     'ai_search'           => ['dataforseo'],
                     'content_gaps'        => ['ahrefs', 'semrush'],
                     'search_performance'  => ['gsc'],
                 ];
             }
             ```

             ### 4.3 Caching Strategy

             | Data Type | Cache TTL | Reason |
             |----------|-----------|--------|
             | Keyword metrics | 7 days | Doesn't change frequently |
             | Rankings/positions | 24 hours | Changes daily |
             | GSC search performance | 12 hours | Data has 2-3 day lag |
             | GA4 traffic data | 6 hours | Near real-time |
             | Backlink profile | 7 days | Changes slowly |
             | SERP results | 24 hours | Changes frequently |
             | LLM mentions | 7 days | Slow-moving |

             ### 4.4 Cost Management

             ```php
             class ApiCostTracker
             {
                 public function trackUsage(string $provider, string $endpoint, int $credits): void;
                 public function canMakeCall(string $provider, int $estimatedCredits): bool;
                 public function setBudget(string $provider, int $dailyLimit, int $monthlyLimit): void;
                 public function checkBudgetAlerts(): void; // Alert in Slack when approaching limits
             }
             ```

             ---

             ## 5. How Skills Use External Data

             ### Meta Tag Engine + GSC + DataForSEO

             Pull GSC data: pages with high impressions but low CTR need meta tag work. Get keyword metrics from DataForSEO (volume, difficulty, intent). Get SERP results to see competitor meta tags. Generate optimized meta tags informed by real data. Set reminder to check GSC in 2 weeks for CTR improvement.

             ### Internal Linking Engine + GSC + Ahrefs

             GSC: pages ranking 11-20 = "striking distance" targets for internal link boost. Ahrefs: pages with most backlinks = highest authority, should be link hubs. Cross-reference with Knowledge Base for semantic connections. Prioritize links FROM high-authority TO striking-distance pages.

             ### Content Creation Engine + DataForSEO + Ahrefs

             Content gap analysis: keywords IDA doesn't rank for but competitors do. SERP analysis: what's ranking top 10 for target keywords. AI optimization: how topic appears in ChatGPT/Perplexity. After publishing: track rankings via GSC, traffic via GA4.

             ---

             ## 6. Environment Variables

             ```env
             # Google Search Console
             GSC_SERVICE_ACCOUNT_JSON=/path/to/service-account.json
             GSC_PROPERTY_URL=https://internationaldriversassociation.com

             # Google Analytics 4
             GA4_PROPERTY_ID=123456789
             GA4_SERVICE_ACCOUNT_JSON=/path/to/service-account.json

             # DataForSEO (already connected via MCP)
             DATAFORSEO_LOGIN=your-login
             DATAFORSEO_PASSWORD=your-password

             # Ahrefs
             AHREFS_API_TOKEN=your-api-token

             # SEMrush (optional)
             SEMRUSH_API_KEY=your-api-key

             # Cost budgets (credits per day)
             DATAFORSEO_DAILY_BUDGET=1000
             AHREFS_DAILY_BUDGET=500
             ```

             ---

             ## 7. Implementation Priority

             | Integration | Priority | Reason |
             |------------|----------|--------|
             | DataForSEO | Phase 1 | Already connected via MCP. Most versatile single API. |
             | Google Search Console | Phase 1 | Free, first-party, most authoritative ranking data. |
             | Google Analytics 4 | Phase 2 | Needed for content optimization and reporting. |
             | Ahrefs | Phase 3 | Essential for backlink expert skill. |
             | SEMrush | Optional | Mostly redundant with DataForSEO + Ahrefs. |
