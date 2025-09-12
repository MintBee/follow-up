Follow Up Specification

0. Document Control
Version: 0.2.0  
Status: Draft  
Owner: TBD  
Last Updated: 2025-09-11  
Change Log:  
- 0.2.0 Restructured spec, enumerated requirements, added metrics & governance sections, split data model to separate appendix file.  
- 0.1.0 Initial draft.  

1. Summary
Follow Up is a service that tracks events within user-defined categories for each user and summarizes how each category's situation has changed over a user-defined period (e.g., today vs yesterday, or any custom date range). It also helps users catch up after an absence by explaining what changed and why. 

2. Goals
- Let users monitor domains (categories) they care about.
- Show how the state of a category changed between two points in time.
- Explain changes using the events that occurred in the interval.
- Provide a fast “catch up since I was away” view.
- Allow guests to explore predefined categories without sign-up.
- Enable registered users to create custom categories with tailored instructions (future enrichment use).

3. Out of Scope (OS)
OS-001 Real-time streaming updates.  
OS-002 Collaboration / shared categories between users.  
OS-003 Advanced analytics or forecasting beyond summarization.  
OS-004 Native mobile applications (web-first).  
OS-005 Third-party connector auto-ingestion (future enhancement).  

4. Personas
- Guest: Browses predefined categories (read-only).
- Registered User: Manages custom categories and views all features.
- Returning User After Absence: Wants a concise change digest.

5. Core Concepts
Category: A domain being tracked (e.g., Finance).  
Event: A discrete piece of information affecting a category.  
Situation: Summarized representation of category state at a point in time.  
Period Comparison: Start situation + end situation + diff + explaining events.  
Absence Catch-Up: Comparison from last seen timestamp to now.

5.1 System Context (Placeholder)
High-level actors: User (guest/authenticated), Web App (frontend), API Service, AI Service, Job Runner (situations), Data Store, Metrics/Logging. Diagram to be added.

5.2 Domain Entities (Conceptual Only)
- Category: Named scope users monitor.
- Event: Time-stamped fact influencing a Category.
- Situation: Summarized state derived from events up to a point.
- Comparison: Computed difference between two states plus contributing events.
- User: Actor creating and viewing categories/events.

6. Predefined Categories (guest-visible)
- Finance
- Technology
- Software Ecosystem

7. Functional Requirements (Enumerated)
FR-001 Guest can retrieve list of predefined categories.  
FR-002 Only active predefined categories appear in guest list.  
FR-003 Guest can view today vs past day (up to last week) summary for a predefined category.  
FR-004 Guest attempting to create a category is prompted to sign up; no category is created.  
FR-005 User can register via Google social login (Only login method for MVP)
FR-006 System records user.lastSeenAt at successful authenticated session start.  
FR-007 Authenticated user can create a category (name required; description & instructions optional). Creating a category without a name is not allowed. Category instructions are stored and limited to 2000 characters (hard validation).  
FR-028 
FR-009 Authenticated user can update own category name, description, instructions.  
FR-010 User can archive (soft delete) own category.  
FR-011 Archived categories are excluded from default listing endpoints.  
FR-012 Authenticated user can list categories: own + active predefined.  
FR-013 User can view category detail including latest situation summary (materialized or on-demand).  
FR-014 Authenticated user can create an event for a category they own.  
FR-015 System defaults event.occurredAt to current UTC timestamp if omitted.  
FR-018 Daily situation and news collecting job executes once per 24h UTC for each active category.  
FR-019 If no events since prior situation boundary, events for the day is shown as "nothing happened for this day" and situation is shown same as the day before for the day.
FR-020 User can request on-demand period comparison specifying categoryId, startDate, endDate (inclusive dates).  
FR-021 Comparison resolves start situation (closest on/before startDate) or the day before if absent
FR-022 Comparison resolves end situation (closest on/before endDate) or generates provisional current state.  
FR-023 Comparison returns: startSummary, endSummary, diffSummary, orderedEvents, keyMetricsChanges (empty structures allowed).  
FR-024 Today-vs-yesterday endpoint is a convenience wrapper over generic comparison.  
FR-025 Absence catch-up uses user.lastSeenAt (date floor) to now for each category.  
FR-026 Catch-up excludes categories with zero events & no state change since lastSeenAt.  
FR-027 Events in responses are ordered by importance (high > normal > low) then chronological within each tier.  

AI Functional Requirements  
AI-FR-001 AI generates daily situation summaries when events exist for the interval.  
AI-FR-002 AI generates diff summaries for period comparisons.  
AI-FR-003 AI generates catch-up digest summaries.  
AI-FR-004 Each AI call enforces hard timeout <= 4s.  
AI-FR-005 On AI timeout, error, or rate limit, system returns deterministic fallback diff/summary within overall SLA.  
AI-FR-006 Fallback pathway is logged distinctly (successFlag=false).  
AI-FR-007 AI request includes minimal redacted content per privacy rules (Section 19).  
AI-FR-008 p95 end-to-end comparison latency with AI enabled < 2.5s.  

8. Non-Functional Requirements (NFR)
NFR-002 Overall comparison with AI (including fallback) <2.5s p95.  
NFR-003 System prepared to shard events by categoryId when total events >1M.  
NFR-004 Simple RBAC: users may mutate only their own categories/events.  
NFR-005 Nightly situation job retries up to 3 times on failure with exponential backoff.  
NFR-006 Structured logs for all key operations (ingestion, comparison, situation, AI calls).  
NFR-007 Events retained indefinitely in MVP; pruning policy deferred.  
NFR-008 Track and surface token usage aggregates daily.  
NFR-009 Graceful degradation ensures no hard dependency on AI for core diff output.  

9. Success Metrics
M-001 p95 comparison latency (AI path) <2.5s.  
M-002 Fallback diff usage rate <5% daily (7-day rolling average).  
M-003 situation job success rate ≥99% daily.  
M-004 Mean AI latency <1.2s.  
M-005 Average token cost per comparison ≤ (TBD threshold) tokens.  

10. Edge Cases
EC-001 Period with zero events returns identical start/end summaries and empty diff explanation.  
EC-002 Events spanning DST transitions use UTC normalization; no duplicate or missing hour.  
EC-003 Large burst (>5k events in day) triggers pagination or streaming diff strategy (implementation detail) yet still returns summary within SLA (may truncate events list with indicator).  
EC-004 AI partial output (truncated) triggers validation + fallback.  
EC-005 Duplicate events (same id) rejected; idempotency key = event id.  
EC-006 Category archived mid-comparison returns 410 or designated error code (decision TBD TD-7).  

11. Risks & Mitigations
R-001 AI cost spike → Mitigation: per-request token cap + daily aggregate alert.  
R-002 situation job failure → Mitigation: retry + alert channel + fallback on-demand situations.  
R-003 Prompt injection via event content → Mitigation: sanitize & neutralize system directives before AI call.  
R-004 Latency regressions → Mitigation: budgets enforced in CI perf tests.  
R-005 Data model drift → Mitigation: schema versioning & migration checklist.  
R-006 Rate limiting from AI provider → Mitigation: local queue + jittered retry once then fallback.  

12. Privacy & Redaction Rules
P-001 Strip user emails and internal URLs before sending content to AI.  
P-002 Remove tokens matching configured secret patterns (regex) prior to AI submission.  
P-003 Limit AI prompt to only events within requested window + minimal category context.  
P-004 Log redacted form, never raw sensitive content, for AI prompts.  

13. Observability Specs
LOG-001 Log each AI call: categoryId, requestType, latencyMs, tokenIn, tokenOut, successFlag, fallbackFlag.  
LOG-002 Log situation job start/end with duration & event count.  
MET-001 Counter events_ingested_total.  
MET-002 Counter ai_calls_total (labels: success|fallback|errorType).  
MET-003 Histogram ai_latency_ms.  
MET-004 Counter situation_job_failures_total.  
MET-005 Gauge token_usage_daily (per request type).  
ALERT-001 Trigger alert if fallback rate >10% for 30 min.  

14. Fallback Behavior Matrix
Case: Timeout → One retry? No (direct fallback) to preserve SLA.  
Case: Rate limit → Single retry with exponential backoff capped at 500ms then fallback.  
Case: Provider error 5xx → One retry; on failure fallback.  
Case: Prompt safety rejection → Sanitize & retry once; else fallback.  
Case: Validation error (internal) → Immediate fallback.  
All fallbacks produce deterministic diff + mark fallbackFlag=true.  

15. API Surface (Draft)
GET /api/categories  
POST /api/categories  
PATCH /api/categories/{id}  
POST /api/categories/{id}/events  
GET /api/categories/{id}/comparison?start=YYYY-MM-DD&end=YYYY-MM-DD  
GET /api/categories/{id}/today-vs-yesterday  
GET /api/catch-up (auth)  
POST /internal/situations/run (secured)  
POST /internal/ai/rewrite (secured, optional)  

16. Acceptance Criteria (Samples)
AC-001 Guest listing returns only active predefined categories (FR-001/FR-002).  
AC-002 Missing category name on create returns 400 (FR-008).  
AC-003 Comparison with no events returns empty diffSummary (FR-023, EC-001).  
AC-004 Catch-up excludes unchanged categories (FR-026).  
AC-005 AI timeout triggers fallback diff with fallbackFlag=true and latency <2.5s (AI-FR-004/AI-FR-005/M-001).  
AC-006 AI metadata present in AI-generated responses (FR-032).  
AC-007 Event order respects importance then time (FR-027).  
AC-008 Instructions >2000 chars rejected (FR-028).  

17. Technical Decisions Log
TD-1 situation storage strategy (store vs infer). Status: Pending.  
TD-2 Pagination approach for large event lists (cursor on occurredAt+id). Status: Pending.  
TD-3 Maximum custom categories per user (proposed 20). Status: Pending.  
TD-4 Event enrichment / dedup logic approach. Status: Pending.  
TD-5 Token cost guardrails (per-user & global caps). Status: Pending.  
TD-6 Redaction strategy details (patterns list). Status: Pending.  
TD-7 Response code for archived category access (410 vs 404). Status: Pending.  

18. Future Enhancements
- Advanced AI reasoning traces.  
- Webhooks / RSS feeds.  
- Tagging events.  
- Metrics extraction & structured metric events.  
- Mobile push notifications.  
- Automated event ingestion connectors.  

19. Glossary
situation: Summarized state at or before a point in time.  
Diff Summary: Human-readable explanation of change between two situations.  
Catch-Up: Aggregated comparison since user’s lastSeenAt.  
Fallback Diff: Non-AI deterministic diff when AI unavailable.  

20. Implementation Phases
Phase 1 (MVP): Core auth, predefined & custom categories, manual events, today vs yesterday, basic period comparison, AI integration with fallback.  
Phase 2: Absence catch-up, nightly situations automation, importance sorting refinements, cost/usage dashboards.  
Phase 3: Bulk import, advanced AI reasoning enhancements, metrics extraction, connectors, mobile push.  

21. Requirement Traceability (Placeholder)
Future table mapping FR/AI-FR/NFR IDs → Test IDs.  

22. References
Appendix Data Model Sketch: see appendix-data-model.md  
