 are stored efficiently using the Supabase Vector Store node.

### Handling Data Types

*   **Select Fields**: When updating Statamic via n8n (PATCH requests), `Select Fields` (e.g., Continent) must be sent as their raw string key (e.g., "south-america"), not as objects, to ensure correct data interpretation by Statamic.
*   **Bard Fields**: Content stored as ProseMirror JSON should be manipulated directly within n8n using JSON nodes. Converting to raw HTML is discouraged, as it can lead to loss of Statamic’s native formatting and potential data inconsistencies.

### Update Safety: The "Drift" Check

Every n8n workflow that writes back to Statamic must incorporate the **Pre/Post Verification Pattern** to prevent unintended data changes and ensure system integrity. This pattern is detailed in the following section.

## The Pre/Post Verification Pattern

The Pre/Post Verification Pattern is a critical safety mechanism designed to prevent data drift and ensure that updates to Statamic are executed as intended. This pattern must be applied to every n8n workflow that performs write operations (e.g., PATCH requests) back to Statamic.

**Steps:**

1.  **Capture Pre-PATCH State**: Before executing a PATCH request to Statamic, capture the current state of critical fields (e.g., `title`, `slug`, `meta_title`, `meta_description`) for the target entry. This snapshot represents the expected state before the update.
2.  **Perform the PATCH**: Execute the n8n workflow's PATCH operation to update the Statamic entry.
3.  **Compare Post-PATCH Response**: Immediately after the PATCH operation, compare the response from Statamic with the captured pre-PATCH state. Specifically, verify that guarded fields, such as the `slug`, and other critical data points, have not changed unexpectedly. This step is crucial for identifying and mitigating issues like the "Slug Guard" constraint, where a missing slug in a PATCH request could delete the page URL.

This pattern acts as a defensive measure, providing an immediate feedback loop on the success and integrity of data updates, thereby safeguarding against accidental data loss or corruption.

## Newbie Roadmap (Weeks 1-3)

This roadmap provides a structured onboarding plan for new team members, guiding them through the initial setup and development phases of the IDA AI-Powered SEO Engine project. Each week focuses on building foundational knowledge and practical experience with key components of the system.

| Week | Focus Area | Key Activities & Learning Objectives |
| :--- | :--------- | :----------------------------------- |
| **Week 1** | **Setup & Ingestion Fundamentals** | Establish n8n and Supabase connections. Successfully run the Phase 0 Ingestion workflow for 10 test pages to understand data flow from Statamic to Supabase. |
| **Week 2** | **Read-Only Workflow Development** | Build the H1 Audit workflow (Phase 1). This is a "read-only" workflow, providing a safe environment to learn n8n logic and interaction with Supabase without modifying production data. |
| **Week 3** | **Advanced Workflow & Safety Implementation** | Implement the Meta & Content Optimizer workflow (Phase 2), including the mandatory "Slug Guard" mechanism. Focus on ensuring no URLs are broken during AI-driven updates by correctly applying the Pre/Post Verification Pattern. |

## Conclusion

Adhering to the guidelines outlined in this document is essential for the collaborative development and long-term maintainability of the IDA AI-Powered SEO Engine. By following best practices for workflow creation, diligently applying the Pre/Post Verification Pattern, and leveraging the structured newbie roadmap, the team can ensure a robust, scalable, and error-resistant intelligence layer that drives IDA's SEO strategy forward.