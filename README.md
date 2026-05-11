# IDA AI SEO Engine

An autonomous, self-reflecting AI agent that manages SEO operations for [International Drivers Association](https://internationaldriversassociation.com). The agent handles bulk meta tag optimization, internal linking, content optimization, and content creation — with human-in-the-loop oversight via Slack.

## Documentation

| Document | Description |
|----------|-------------|
| [Architecture](docs/architecture.md) | System design, components, and data flow |
| [Agent Behavior](docs/agent-behavior.md) | Autonomy tiers, self-reflection, and decision-making |
| [Skill Architecture](docs/skill-architecture.md) | Plugin system, content frameworks, and extensibility |
| [Data Integrations](docs/data-integrations.md) | GSC, GA4, DataForSEO, Ahrefs — external data sources |
| [Slack Integration](docs/slack-integration.md) | Commands, notifications, and approval workflows |
| [Implementation Roadmap](docs/roadmap.md) | Phased build plan with milestones |

## Quick Overview

```
┌─────────────────────────────────────────────────────┐
│                    SLACK (Human)                     │
│  Commands · Approvals · Reports · Notifications      │
└──────────────┬──────────────────────┬───────────────┘
               │                      │
               ▼                      ▼
┌──────────────────────┐  ┌───────────────────────────┐
│    Agent Controller   │  │     Approval Queue        │
│  (Task Router + Loop) │  │  (Pending human review)   │
└──────────┬───────────┘  └───────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────────┐
│              SKILL REGISTRY (Pluggable)              │
│                                                      │
│  ┌─────────┐ ┌──────────┐ ┌───────────┐ ┌────────┐ │
│  │Meta Tag │ │ Internal │ │ Content   │ │Content │ │
│  │ Engine  │ │ Linking  │ │ Optimizer │ │Creator │ │
│  └─────────┘ └──────────┘ └───────────┘ └────────┘ │
│                                                      │
│  ┌─────────┐ ┌──────────┐ ┌───────────┐            │
│  │Backlink │ │ A/B Test │ │  Schema   │  + more... │
│  │ Expert  │ │ Manager  │ │Specialist │            │
│  └─────────┘ └──────────┘ └───────────┘            │
└──────────────────┬──────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────┐
│            CONTENT FRAMEWORKS (Loadable)             │
│  Koray Topical Authority · Entity SEO · Brand Voice  │
└─────────────────────────────────────────────────────┘
                   │
           ┌───────┴───────┐
           ▼               ▼
┌──────────────────┐  ┌───────────────────────────┐
│   Statamic CMS   │  │   Supabase (pgvector)     │
│ (Source of Truth) │  │   Knowledge Base + RAG    │
└──────────────────┘  └───────────────────────────┘
```

## Project Status

**Phase**: Pre-development (Architecture & Planning)
**Target**: Native Laravel implementation
**Contact**: johnray@internationaldriversassociation.com
