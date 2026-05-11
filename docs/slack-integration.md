# Slack Integration

Slack is the sole communication channel between the team and the AI agent. Every interaction -- commands, reports, approvals, and notifications -- flows through Slack.

## 1. Slack App Setup

### Required Scopes (Bot Token)
```
chat:write          -- Send messages
chat:write.public   -- Post to channels the bot isn't in
commands            -- Register slash commands
app_mentions:read   -- Respond when @mentioned
channels:history    -- Read channel messages (for threaded conversations)
files:write         -- Upload reports/files
reactions:read      -- Detect approval/rejection via emoji reactions
```

### Event Subscriptions
```
app_mention         -- Trigger when someone @mentions the bot
message.channels    -- Listen for messages in subscribed channels
```

### Dedicated Channel
Create a `#seo-agent` channel (or similar) for all agent communications. The agent should only operate in designated channels, never in DMs or random channels.

---

## 2. Inbound: Commands to the Agent

### Slash Commands

| Command | Description | Example |
|---------|-------------|----------|
| `/seo audit [scope]` | Run an SEO audit on specified pages | `/seo audit countries` |
| `/seo meta [scope]` | Analyze and optimize meta tags | `/seo meta blog` |
| `/seo links [scope]` | Analyze internal linking opportunities | `/seo links driving-guides` |
| `/seo optimize [url]` | Optimize a specific page's content | `/seo optimize /blog/tuscany-italy-road-trip` |
| `/seo create [brief]` | Create new content from a brief | `/seo create blog post about driving in winter` |
| `/seo status` | Show current task queue and status | `/seo status` |
| `/seo report [type]` | Generate and post a report | `/seo report weekly` |
| `/seo rollback [task-id]` | Rollback a previous change | `/seo rollback task-1234` |
| `/seo help` | Show available commands | `/seo help` |

### Natural Language (via @mention)

The agent should also respond to natural language instructions when @mentioned:

```
@seo-agent optimize the meta descriptions for all country pages

@seo-agent what pages don't have any internal links pointing to them?

@seo-agent write a blog post about road trip essentials for Southeast Asia

@seo-agent how many pages have meta titles longer than 60 characters?

@seo-agent show me the internal link map for the driving guides collection
```

### Command Processing Flow

```
Slack message received
       |
       v
Verify signature (Slack signing secret)
       |
       v
Parse intent (slash command or NLP for @mentions)
       |
       v
Validate: Is this a recognized command?
  - YES: Acknowledge in Slack ("Working on it...")
         Create task in queue
         Execute via Agent Controller
  - NO:  Respond: "I didn't understand that. Try /seo help"
```

---

## 3. Outbound: Agent to Slack

### Notification Types

#### Task Started
```
[RUNNING] Meta Tag Optimization Started
Scope: Blog collection (52 pages)
Estimated time: ~5 minutes
Triggered by: @john via /seo meta blog
```

#### Auto-Executed Change (Tier 1)
```
[DONE] Meta Tags Updated -- Auto-Applied
Pages affected: 12
Changes:
- /blog/tuscany-italy-road-trip -- Title shortened from 73 to 58 chars
- /blog/best-car-rental-in-ireland -- Description rewritten (was duplicate)
- ... (10 more)

Full report: [View Details]
Rollback: /seo rollback task-1234
```

#### Approval Request (Tier 2)
```
[APPROVAL NEEDED] Content Optimization
Page: /andorra-driving-guide
Requested by: @john via /seo optimize

What's changing:
- H2 added: "Required Documents for Driving in Andorra"
- Section expanded: "Traffic Rules" (added 280 words on speed limits and roundabouts)
- 3 internal links added to related country pages

Before/After preview:
[View Diff] (link to detailed diff)

React to approve:
Check = Approve  |  X = Reject  |  Pencil = Modify
```

#### Weekly Report
```
[WEEKLY REPORT] SEO Agent Report
Period: May 5 - May 11, 2026

Actions taken:
- 34 meta titles optimized (auto)
- 18 meta descriptions rewritten (auto)
- 5 content optimizations (3 approved, 1 rejected, 1 pending)
- 1 new blog post drafted (pending approval)
- 47 internal link opportunities identified

Key metrics:
- Pages with meta issues: 156 to 122 (down 22%)
- Orphan pages: 23 to 18 (down 22%)
- Average content word count: 1,340 to 1,420 (up 6%)

Pending approvals: 2 items waiting in queue
```

#### Error Notification
```
[WARNING] Task Failed: Content Optimization
Page: /blog/electric-car-rental-guide-usa
Error: Statamic API returned 429 (rate limited)
Retries: 3/3 exhausted

Action needed: Will retry automatically in 1 hour.
Override: /seo retry task-1235
```

---

## 4. Approval Workflow

### Flow

```
Agent posts approval request to #seo-agent
       |
       v
Human reviews the proposed change
       |
       |- Check reaction: Agent executes the change
       |      Agent confirms: "Change applied to /andorra-driving-guide"
       |
       |- X reaction: Agent cancels
       |      Agent asks: "Noted. Can you tell me why so I improve next time?"
       |
       |- Pencil reaction: Agent asks for modifications
       |      "What would you like me to change about this?"
       |      Human replies in thread
       |      Agent revises and re-requests approval
       |
       |- No response (48h): Agent sends reminder
              Still no response (72h): Auto-cancel, log as timed out
```

### Thread-Based Conversations

All discussion about a specific approval happens in the thread of the original approval message. This keeps the channel clean and groups context together.

```
[APPROVAL REQUEST] (main message)
  Thread:
     - Agent: "Here's the detailed diff..."
     - Human: "The third paragraph sounds too salesy"
     - Agent: "Revised. Here's the updated version."
     - Human: (approves revised version)
```

---

## 5. Conversational Context

The agent should maintain conversation context within a Slack thread. If a human asks a follow-up question in the same thread, the agent should understand it refers to the same topic.

```
Human: @seo-agent optimize the meta tags for country pages
Agent: "Working on it... Found 34 pages with issues. Here are the proposed changes: [...]"
Human: "Actually, skip the Pacific Island countries for now"
Agent: "Got it. Excluding 12 Pacific Island country pages. Updated proposal: [...]"
Human: "Looks good, go ahead"
Agent: "Applying changes to 22 country pages..."
```

---

## 6. Permissions and Safety

- Only designated team members can issue commands (configurable whitelist)
- - Only designated approvers can approve Tier 2 changes
  - - The agent never acts on messages from outside the designated channel
    - - All Slack interactions are logged with timestamp, user, and content
      - - The agent never shares API keys, passwords, or sensitive data in Slack
        - - Rate limit Slack messages: max 1 message per second, batch multiple changes into single messages
