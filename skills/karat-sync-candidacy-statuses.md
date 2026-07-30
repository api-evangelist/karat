---
name: Sync and disposition candidacy statuses
description: Page through Karat candidacies to sync interview status back to your ATS, then bulk-update dispositions.
api: graphql/karat-operations.graphql
operations: [candidacies, bulkUpdateCandidacyStatus]
---

# Sync and disposition candidacy statuses

Keep your ATS in sync with Karat interview outcomes, then push dispositions back to Karat.

## Endpoint & auth
- POST `https://{subdomain}.karat.io/api/v1/graphql`.
- Header: `Authorization: Bearer <tenant-token>`.

## Steps
1. **Page candidacies.** Run the `candidacies` query with a `filter` (e.g.
   `lastStatusUpdateAfter`, `role`, `status`). It is a Relay connection: read
   `totalCount` and `pageInfo { endCursor hasNextPage }`, and page with `first` + `after`
   until `hasNextPage` is false.
2. **Read outcomes.** For each candidacy node use `status`, and the nested
   `interviews.nodes[].recommendation` / `result` and `codeChallenges.nodes[]` to map the
   Karat outcome to your ATS. `atsUrl` / `atsCandidateId` / `atsApplicationId` link back
   to your system.
3. **Disposition in bulk.** Call `bulkUpdateCandidacyStatus` with `updates` grouping
   candidacy `ids` by target status (e.g. `OFFER_ACCEPTED`, `DECLINED_AT_ONSITE_INTERVIEW`).
   Read `successfulUpdates`, `failedUpdates`, and `errors { message }`.

## Rules
- Always page to completion using the cursor; do not assume a single page.
- `bulkUpdateCandidacyStatus` reports partial success — check `failedUpdates`/`errors`
  and retry only the failed ids.
- See `data-model/karat-data-model.yml` for the Candidacy → Role/Candidate/Interview graph.
