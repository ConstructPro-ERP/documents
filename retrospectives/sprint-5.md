# Sprint 5 Retrospective

Date: 2026-06-14  
Attendees: Iddamalgoda I.H.R.P. (SE/2021/038), Dhanushan K.I. (SE/2021/027), Ranasundara R.I.K. (SE/2021/001), Abeywardena M.B. (SE/2021/037)

## Went Well

1. We completed the planned foundation work and established a common structure for the application. The frontend was initialized with Next.js and TypeScript, while the backend was organized as a NestJS monorepo with separate service applications under `apps/` and an API Gateway. ([Frontend issue #1](https://github.com/ConstructPro-ERP/frontend/issues/1), [PR #5](https://github.com/ConstructPro-ERP/frontend/pull/5); [Backend issue #1](https://github.com/ConstructPro-ERP/backend/issues/1), [PR #2](https://github.com/ConstructPro-ERP/backend/pull/2).)
2. We created the public-facing landing pages, initial login interfaces, shared layouts and a dashboard skeleton with placeholder content. This gave us a consistent structure for introducing functional dashboard modules in later sprints. ([Frontend issue #6](https://github.com/ConstructPro-ERP/frontend/issues/6), [PR #9](https://github.com/ConstructPro-ERP/frontend/pull/9); subsequent authentication and dashboard refinement continued in [PR #11](https://github.com/ConstructPro-ERP/frontend/pull/11) and [PR #20](https://github.com/ConstructPro-ERP/frontend/pull/20).)
3. We established the initial Prisma configuration and database-model direction for the business modules. ([Backend issue #8](https://github.com/ConstructPro-ERP/backend/issues/8), [PR #18](https://github.com/ConstructPro-ERP/backend/pull/18).)
4. The sprint-end meeting confirmed the progress and allowed us to divide the next stage into backend responsibilities for all four members. We also prepared the four-person frontend task breakdown and agreed to begin frontend module work approximately two weeks after Sprint 6 backend work started.

## To Improve

1. We need to confirm shared API and database interfaces before the four backend work packages develop too far independently.
2. Placeholder dashboard sections should be replaced incrementally with connected modules while retaining the common layout.
3. We should plan integration checkpoints alongside module implementation rather than leave all integration work until later.

## Action Items

| Action                                                                                 | Owner            | Due              |
| -------------------------------------------------------------------------------------- | ---------------- | ---------------- |
| Finalize and assign the Sprint 6 backend work packages to four members.                | All four members | Sprint 6, Week 1 |
| Finalize the corresponding frontend implementation breakdown.                          | All four members | Sprint 6, Week 1 |
| Start frontend module implementation after the first two weeks of backend development. | All four members | Sprint 6, Week 3 |

## Metrics

- Issues planned: 12 issues in the Sprint 5 milestone.
- Issues completed: 12 of 12 milestone issues closed in the final milestone record; the foundation tasks were completed as planned.
- Velocity: Not recorded in story points on the project board.
- CI pipeline pass rate: A sprint-specific percentage was not recorded on the project board.
- Test coverage: A sprint-close percentage was not recorded on the project board.
