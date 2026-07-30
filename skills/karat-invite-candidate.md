---
name: Invite a candidate into a Karat assessment
description: Look up the target hiring role, then invite a candidate into a Karat technical assessment via the GraphQL API.
api: graphql/karat-operations.graphql
operations: [roles, createInvitation]
---

# Invite a candidate into a Karat assessment

Use Karat's GraphQL API to invite a candidate to a technical assessment for a specific role.

## Endpoint & auth
- POST `https://{subdomain}.karat.io/api/v1/graphql` (test env: `{subdomain}.cotrain.io`).
- Header: `Authorization: Bearer <tenant-token>`.
- All requests are scoped to your tenant subdomain (see `authentication/karat-authentication.yml`).

## Steps
1. **Find the role.** Run the `roles` query (filter/search by name) and capture the `id`
   of the role you are hiring for. Roles belong to a `group`.
2. **Create the invitation.** Call the `createInvitation` mutation with the required
   inputs `name`, `email`, and `roleId`. Optional inputs: `phone`, `atsUrl`, `resume`,
   `githubUrl`, `linkedinUrl`.
3. **Read the result.** On success the payload returns `assessment { id }`. Domain
   problems are returned in `errors { message }` in the same payload — always check it.

## Rules
- `name`, `email`, and `roleId` are mandatory; the mutation fails if any is missing.
- There is no idempotency key — do not blindly retry a `createInvitation`; re-query first
  to avoid duplicate invitations (see `conventions/karat-conventions.yml`).
- Roles queries are Relay connections: page with `first` + `after` (`pageInfo.endCursor`).
