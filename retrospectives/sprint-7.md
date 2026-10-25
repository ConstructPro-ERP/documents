# Sprint 7 Retrospective

Date: 2026-09-15  
Attendees: Iddamalgoda I.H.R.P. (SE/2021/038), Dhanushan K.I. (SE/2021/027), Ranasundara R.I.K. (SE/2021/001), Abeywardena M.B. (SE/2021/037)

## Went Well

1. At our 15 September planning meeting, we shifted focus to connecting the frontend and backend. We integrated Next.js user interfaces with the NestJS services and API Gateway and refined both sides when implementation differences became apparent.
2. Project-domain implementation continued after the Sprint 6 delays. [Backend issue #74](https://github.com/ConstructPro-ERP/backend/issues/74) / [PR #79](https://github.com/ConstructPro-ERP/backend/pull/79) delivered the project foundation, and [issue #80](https://github.com/ConstructPro-ERP/backend/issues/80) / [PR #89](https://github.com/ConstructPro-ERP/backend/pull/89) delivered project operations and manager assignment.
3. We connected further user workflows across quotations, leads, projects and dashboards. Related pull requests included [Frontend PR #44](https://github.com/ConstructPro-ERP/frontend/pull/44) for quotation approval/conversion, [PR #49](https://github.com/ConstructPro-ERP/frontend/pull/49) for quotation lifecycle/API integration, [PR #48](https://github.com/ConstructPro-ERP/frontend/pull/48) for lead management and [PR #46](https://github.com/ConstructPro-ERP/frontend/pull/46) for the dashboard.
4. We reviewed the integrated work around 7 October and agreed that the final sprint should focus on testing, quality, performance and remaining defects.
5. After Sprint 7, we demonstrated the progress to the client, Ishara Homes (Pvt.) Ltd. The client expressed satisfaction with the system demonstrated.

## To Improve

1. Some frontend and backend API contracts needed changes during integration. We should coordinate field names, validation rules and response formats earlier.
2. Integration work needs timely end-to-end verification so that connected user flows do not rely only on separately passing module tests.
3. Several integration PRs merged shortly after the approximate 7 October review, including [Frontend PR #44](https://github.com/ConstructPro-ERP/frontend/pull/44) on 8 October and [PR #49](https://github.com/ConstructPro-ERP/frontend/pull/49) on 9 October. Work carried across the review date should remain visible until its PR and verification are complete.
4. The client demonstration was useful feedback, but we still needed structured testing and release checks before submission.

## Action Items

| Action                                                                                   | Owner            | Due                     |
| ---------------------------------------------------------------------------------------- | ---------------- | ----------------------- |
| Resolve remaining frontend–backend contract mismatches in assigned modules.              | All four members | Sprint 8                |
| Prepare unit, integration and end-to-end tests for the main business workflows.          | All four members | Sprint 8, Week 1 onward |
| Check all main UI paths, loading/error states and role-based permissions.                | All four members | Sprint 8                |
| Prioritize reliability, performance checks and system-wide testing for the final sprint. | All four members | Sprint 8                |

## Metrics

- Issues planned: 25 issues in the Sprint 7 milestone in the final record.
- Issues completed: 25 of 25 milestone issues eventually closed; some related PRs merged after the early-October sprint review.
- Velocity: Not recorded in story points on the project board.
- CI pipeline pass rate: A sprint-specific percentage was not recorded on the project board.
- Test coverage: A sprint-close percentage was not recorded on the project board.
