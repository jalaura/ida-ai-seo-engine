# Pull Request

## Summary

Provide a clear description of what this PR changes and why.

## Type of Change

- [ ] New n8n workflow
- [ ] Workflow modification
- [ ] Documentation update
- [ ] Bug fix
- [ ] Infrastructure / configuration change

## Related Issue

Closes #

## Changes Made

Describe the specific changes in this PR, including which workflows, nodes, or documents were modified.

## Safety Checklist

For any workflow that writes back to Statamic, confirm the following:

- [ ] **Slug Guard**: The workflow fetches the current slug and echoes it back in all PATCH request bodies
- [ ] **Drift Check**: Pre/Post Verification Pattern is implemented (capture state → PATCH → compare response)
- [ ] **Select Fields**: Only raw string keys (e.g., `"south-america"`) are sent in PATCH requests, not full objects
- [ ] **Bard Fields**: ProseMirror JSON is manipulated directly; no HTML conversion is performed
- [ ] **Locale Filter**: Ingestion workflows filter for `locale === 'en'` (or designated target locale)

## Testing

Describe how this was tested. Include:

- Number of test pages processed
- Workflow execution logs reviewed
- Data integrity verified in Supabase

## Documentation

- [ ] PRD updated (if applicable)
- [ ] Architecture docs updated (if applicable)
- [ ] API Reference updated (if applicable)
- [ ] CHANGELOG updated
- [ ] PROJECT_STATUS.md updated

## Screenshots / Logs

Attach relevant n8n execution screenshots or Supabase query results if applicable.
