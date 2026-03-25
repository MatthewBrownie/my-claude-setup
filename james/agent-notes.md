# James Agent Notes
_Last updated: 2026-03-24_

## Standing concerns
<!-- Architectural decisions, persistent preferences, or constraints that should always be on James's radar.
     These are never pruned unless the user explicitly says to remove one. -->

## Active topics
<!-- One entry per topic currently in play.
     Format:
       [YYYY-MM-DD] Topic name: brief description of where things stand
       [DONE YYYY-MM-DD] Topic name: one-line outcome summary

     Pruning rules:
       - Mark complete with [DONE YYYY-MM-DD] and condense to one line immediately.
       - Topics not touched in 14+ days: condense to a single line.
       - Topics and completed items are NEVER removed — they stay as one-liners indefinitely. -->

[2026-03-23] Discovery-core test suite quality: Reviewed 34 spec files, ~37% statement / 27% branch coverage. Two styles identified: stub "should be defined" tests vs substantive __tests__ folder suites. Happy-path-only testing drives the branch coverage gap. Need to establish whether tests should encode intent or observed behaviour. Error handling strategy and validation pipeline placement are unknown — knowledge gap to resolve with James. Tidy-up item: test-helpers files are appearing in coverage reports — coveragePathIgnorePatterns in Jest config should exclude them.

[2026-03-24] Enrichment Pipeline architecture: Architecture settled for meeting. Decisions reached: single `enrich(chunks)` interface method (orchestrator gathers chunks, hands them to handlers); handler-owned persistence (each handler writes its own Neo4j nodes); two-tier auditing (orchestrator logs overall enrichment-run milestone, individual handlers log their own milestones); BullMQ worker pattern for queue consumption; registry is in-memory with handlers registering via onModuleInit. Open question: the "context" object on registry return — what it carries is undefined and may be challenged. Minor discrepancy: diagram shows `enrich(text: string)` vs notes saying `enrich(chunks)` — needs reconciling. The `@UseInterceptors(AuditingInterceptor)` on EmbeddingQueueService is likely a leftover (no HTTP context in queue worker).

[2026-03-24] Auditing / Milestone system: Two-tier audit model understood. REQUEST level via AuditingInterceptor on controller methods; MILESTONE level via manual auditingService.createMilestone calls. Both linked by cls_id. Status lifecycle: CREATED -> STARTED -> PROCESSING -> COMPLETED/FAILED. Audit failures never break the request. AsyncTransactionHelper provides fire-and-forget and awaited patterns.

[2026-03-24] BullMQ queue consumer pattern (enrichment service): EmbeddingQueueService extends BaseQueueService<JobData, JobResult>. Pattern well understood: define queueName/jobName/defaultRedisDb, implement processJob(data), implement getRedisDbEnvVar(). Queue service creates its own milestones.

## Session log
<!-- One entry per session, most recent first.
     Format: [YYYY-MM-DD] Brief summary of what was discussed and any decisions reached.

     Pruning rule: entries older than 30 days are removed. -->

[2026-03-24] Meeting prep session: assessed readiness for James meeting on enrichment pipeline. Architecture coherent and defensible. Key decisions settled: single enrich(chunks) method, handler-owned persistence, two-tier auditing, BullMQ worker pattern, in-memory registry with onModuleInit. Flagged minor items: diagram/notes signature discrepancy, undefined "context" on registry, Promise.allSettled vs Promise.all unresolved, leftover AuditingInterceptor on queue service.

[2026-03-24] Familiarisation session: read through auditing library (entity, service, interceptor, decorator, repository, async helper) and BullMQ queue consumer pattern (EmbeddingQueueService extending BaseQueueService). Identified two-tier audit model (REQUEST via interceptor, MILESTONE via manual service calls). Key question surfaced: where milestone auditing belongs in the enrichment pipeline — orchestrator vs handlers.

[2026-03-23] Enrichment pipeline architecture session: settled vocabulary (Orchestrator, Registry, Handler, Extractor, Queue Service, Interface), defined the full request flow from HTTP trigger through fan-out via BullMQ to Neo4j persistence. Key open question for James: handler-owned persistence vs orchestrator-mediated persistence. Diagram prepared for meeting.

[2026-03-23] Reviewed discovery-core test suite: coverage metrics, test styles, test-helpers as mock factories, intent vs behaviour in testing, identified error handling strategy as a knowledge gap for the James meeting. Noted coveragePathIgnorePatterns as a tidy-up item for excluding test-helpers from coverage.

## Learning journey
<!-- Concepts, patterns, and techniques encountered or being internalised.
     Format:
       [YYYY-MM-DD] Concept: status — brief note
     Status: introduced | practicing | understood
     'Understood' entries are kept permanently as one-liners.
     'Introduced' and 'practicing' entries are condensed after 14 days but never removed. -->

[2026-03-23] Registry pattern (service locator for enrichment handlers): practicing — user can explain onModuleInit registration and orchestrator iteration; applying it is the next step.
[2026-03-23] Interface contracts / shared enrichment interface: practicing — user has settled on enrich(chunks) as the single interface method; return type and "context" still being refined.
[2026-03-23] Fan-out via BullMQ queues: practicing — user has existing queue services working and understands the pattern; applying it to the orchestrator fan-out is the next step.
[2026-03-23] test-helpers as mock factories: understood — user grasped the pattern immediately when reviewing discovery-core specs.
[2026-03-23] intent vs behaviour in testing: introduced — discussed the codifying trap where tests written after implementation encode observed behaviour rather than intended behaviour. Open question for James.
[2026-03-23] branch coverage vs statement coverage: introduced — user now understands what the columns mean and why branches lag behind (happy-path-only tests miss null/error paths).
[2026-03-23] NestJS exception handling pipeline: introduced — knowledge gap identified. User doesn't know where validation sits in the NestJS request pipeline or what the current error handling strategy is.
[2026-03-23] Istanbul/Jest coverage instrumentation and coveragePathIgnorePatterns: introduced — understanding how Jest decides which files to instrument for coverage and how to exclude non-production code (test-helpers, fixtures) via config.
[2026-03-24] Two-tier auditing model (REQUEST + MILESTONE): understood — user built the system themselves and can articulate both tiers, the cls_id linkage, and the status lifecycle.
[2026-03-24] NestJS interceptors and custom param decorators: practicing — user wrote AuditingInterceptor and @AuditId(); demonstrated understanding through implementation but not yet discussed the mechanics in conversation.
