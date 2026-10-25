# Sprint 8 Retrospective

Date: 2026-10-01  
Attendees: Iddamalgoda I.H.R.P. (SE/2021/038), Dhanushan K.I. (SE/2021/027), Ranasundara R.I.K. (SE/2021/001), Abeywardena M.B. (SE/2021/037)

## Went Well

1. We reviewed business rules and user actions across the implemented modules, checked the application interfaces and expanded unit, integration and end-to-end testing.
2. We reviewed failing tests and E2E scenarios together and planned corrections instead of treating test failures as acceptable closeout results. We strengthened shared-database E2E execution in [Backend PR #103](https://github.com/ConstructPro-ERP/backend/pull/103), expanded quotation testing in [Backend issue #124](https://github.com/ConstructPro-ERP/backend/issues/124) / [PR #126](https://github.com/ConstructPro-ERP/backend/pull/126), and added client-service unit/integration coverage in [Backend issue #144](https://github.com/ConstructPro-ERP/backend/issues/144) / [PR #145](https://github.com/ConstructPro-ERP/backend/pull/145).
3. We expanded cross-module browser testing through [Tests issue #3](https://github.com/ConstructPro-ERP/tests/issues/3) / [PR #4](https://github.com/ConstructPro-ERP/tests/pull/4) for quotation workflows, [Tests PR #5](https://github.com/ConstructPro-ERP/tests/pull/5) for lead/client/user scenarios, and [Tests issue #6](https://github.com/ConstructPro-ERP/tests/issues/6) / [PR #7](https://github.com/ConstructPro-ERP/tests/pull/7) for client-portal scenarios.
4. We used testing to identify and refine integration details. Examples included milestone API correction ([Frontend issue #63](https://github.com/ConstructPro-ERP/frontend/issues/63) / [PR #64](https://github.com/ConstructPro-ERP/frontend/pull/64)), expense validation ([Frontend issue #65](https://github.com/ConstructPro-ERP/frontend/issues/65) / [PR #67](https://github.com/ConstructPro-ERP/frontend/pull/67)), document uploads ([Frontend issue #74](https://github.com/ConstructPro-ERP/frontend/issues/74) / [PR #77](https://github.com/ConstructPro-ERP/frontend/pull/77)), and task API alignment ([Frontend issue #81](https://github.com/ConstructPro-ERP/frontend/issues/81) / [PR #84](https://github.com/ConstructPro-ERP/frontend/pull/84)).
5. The team discussed the final assignment submission together, including the required coverage threshold, performance testing, UI responsiveness, reliability and supporting documentation.

## To Improve

1. Failing automated tests require investigation, fixes and reruns before release readiness can be claimed. Different failure types—application behavior, test setup and environment issues—must be addressed appropriately.
2. We agreed to check the university's **80% or higher coverage requirement** against actual generated coverage reports and add tests where needed. Implementing coverage reporting in [Backend issue #120](https://github.com/ConstructPro-ERP/backend/issues/120) / [PR #121](https://github.com/ConstructPro-ERP/backend/pull/121) does not itself confirm that every required coverage threshold is met.
3. We needed to recheck responsive UI behavior following performance improvements, including dashboard changes such as [Frontend issue #69](https://github.com/ConstructPro-ERP/frontend/issues/69) / [PR #70](https://github.com/ConstructPro-ERP/frontend/pull/70).
4. Performance testing should be interpreted within its actual scope. The 23 October report covers 25 API routes under up to 50 workers and a single-browser frontend diagnostic; it does not demonstrate the separate 200-concurrent-user requirement or full deployed application performance.
5. The final SRS, SDS, testing records, security evidence, performance report and retrospective documents must accurately describe the implemented system and be prepared for submission.

## Action Items

| Action                                                                                                     | Owner                                 | Due                            |
| ---------------------------------------------------------------------------------------------------------- | ------------------------------------- | ------------------------------ |
| Investigate failing unit, integration and E2E tests; apply fixes and rerun the affected suites.            | All four members, by module ownership | Before final verification      |
| Generate and review coverage reports, address gaps against the required 80%+ threshold.                    | All four members                      | Before final submission        |
| Recheck complete user workflows, role-based permissions and responsive UI behavior after performance work. | All four members                      | Before final verification      |
| Review performance findings and document remaining constraints accurately.                                 | All four members                      | Before final report completion |
| Finalize and review the required university submission documents.                                          | All four members                      | Before submission              |

## Metrics

- Issues planned: 13 issues in the Sprint 8 milestone in the final record.
- Issues completed: 13 of 13 milestone issues eventually closed; [Backend issue #117](https://github.com/ConstructPro-ERP/backend/issues/117) was closed on 23 October following earlier quality-hardening work.
- Velocity: Not recorded in story points on the project board.
- CI pipeline pass rate: A Sprint 8 overall percentage was not recorded on the project board.
- Test coverage: The 80%+ figure was the required **target**, not an overall measured result. Coverage reporting and additional test suites were added; the project-wide sprint-close percentage must be taken from the final coverage artifact.
