# Sprint 6 Retrospective

Date: 2026-07-02  
Attendees: Iddamalgoda I.H.R.P. (SE/2021/038), Dhanushan K.I. (SE/2021/027), Ranasundara R.I.K. (SE/2021/001), Abeywardena M.B. (SE/2021/037)

## Went Well

1. We divided backend development among the four members and made progress on authentication and user management, leads and clients, quotations, invoices and payments, analytics and AI functionality. Examples include [Backend PR #23](https://github.com/ConstructPro-ERP/backend/pull/23) for quotation generation, [PR #34](https://github.com/ConstructPro-ERP/backend/pull/34) for invoice operations, [PR #35](https://github.com/ConstructPro-ERP/backend/pull/35) for payments and [PR #47](https://github.com/ConstructPro-ERP/backend/pull/47) for AI/RAG integration.
2. We discussed the incomplete work rather than assuming the sprint had finished successfully. The team identified university examinations and parallel research work as the main reasons for the delays and agreed to extend the planned implementation period by two weeks.
3. We began deploying the frontend and some implemented backend microservices to Vercel during the extended development period. This also helped us identify deployment and environment-configuration issues earlier.
4. After the end-semester examination break, we resumed development and worked through remaining backend tasks. Prisma reconciliation was merged through [Backend issue #75](https://github.com/ConstructPro-ERP/backend/issues/75) / [PR #76](https://github.com/ConstructPro-ERP/backend/pull/76) on 11 September, followed by further project-domain work.

## To Improve

1. We underestimated the effect of examination and research commitments on our availability. At the original sprint deadline, several planned backend tasks—including project operations—were incomplete.
2. The GitHub Project burn-up chart shows a prolonged flat period from approximately early July into early September. This coincided with the examination period and the interruption to regular development, which lasted around six weeks. Future sprint plans should account for academic commitments and expected pauses rather than depend on uninterrupted availability.
3. We need to identify and reschedule incomplete work as soon as it becomes clear that the agreed sprint scope cannot be delivered on time.
4. Cross-service data models, API contracts and deployment configuration need earlier checks to reduce integration delays once frontend work begins.

## Action Items

| Action                                                                                                           | Owner                                     | Due                                |
| ---------------------------------------------------------------------------------------------------------------- | ----------------------------------------- | ---------------------------------- |
| Extend the Sprint 6 implementation period to address unfinished backend work.                                    | All four members                          | Revised Sprint 6 schedule          |
| Resume and complete outstanding work following the end-semester examination period.                              | All four members, within assigned modules | September recovery period          |
| Continue configuring and deploying the frontend, API Gateway and available backend services on Vercel.           | All four members, within assigned modules | Before Sprint 7 integration review |
| Carry remaining project-domain and operations work into the recovery/integration plan until merged and verified. | All four members, within assigned modules | Sprint 7 follow-through            |
| Review outstanding tasks and plan frontend–backend integration in the 15 September meeting.                      | All four members                          | Sprint 7 planning                  |

## Metrics

- Issues planned: 32 issues are assigned to the Sprint 6 milestone in the final record; the original pre-extension baseline was not separately preserved in the export.
- Issues completed: The sprint was incomplete at its original deadline. All 32 milestone issues were eventually closed after the extended/recovery period; this is **not** the original sprint-end completion count.
- Velocity: Not recorded in story points on the project board.
- CI pipeline pass rate: A sprint-specific percentage was not recorded on the project board.
- Test coverage: A sprint-close percentage was not recorded on the project board.

Project-domain work continued across the revised sprint boundary: [Backend issue #74](https://github.com/ConstructPro-ERP/backend/issues/74) / [PR #79](https://github.com/ConstructPro-ERP/backend/pull/79) merged on 18 September and [Backend issue #80](https://github.com/ConstructPro-ERP/backend/issues/80) / [PR #89](https://github.com/ConstructPro-ERP/backend/pull/89) merged on 29 September.
