---
name: Bug Report
about: Report a workflow failure, data integrity issue, or unexpected behavior
title: "[BUG] "
labels: bug
assignees: ''
---

## Bug Description

Provide a clear and concise description of the issue.

## Affected Component

Select the component(s) involved:

- [ ] Phase 0: Intelligence Indexer (Ingestion)
- [ ] Phase 1: H1 Audit Workflow
- [ ] Phase 2: Meta & Content Optimizer
- [ ] Slug Guard
- [ ] Drift Check / Pre-Post Verification
- [ ] Statamic API Integration
- [ ] Supabase Vector Store
- [ ] Claude AI Integration
- [ ] DataForSEO Integration

## Steps to Reproduce

1. Go to '...'
2. Run workflow '...'
3. Observe error '...'

## Expected Behavior

Describe what should have happened.

## Actual Behavior

Describe what actually happened. Include any error messages or n8n execution logs.

## Data Integrity Impact

**CRITICAL — Answer this before proceeding:**

- [ ] Was a Statamic page URL modified or deleted unexpectedly?
- [ ] Was a slug missing from a PATCH request (Slug Guard failure)?
- [ ] Did a field drift unexpectedly after a PATCH (Drift Check failure)?
- [ ] Was incorrect locale data ingested into Supabase?

## Environment

- n8n Version:
- Supabase Region:
- Statamic Version:
- Date/Time of Occurrence:

## Additional Context

Add any other context, screenshots, or n8n execution data here.
