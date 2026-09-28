# Backend

The server is the same Next.js application. It holds every rule; the
client never decides access or signature status.

## Schema

`prisma/schema.prisma`, SQLite. String columns stand in for enums.

| Model | Holds |
|---|---|
| `User` | GitHub user; `role` `contributor` or `owner`; `oauthToken` stored for owners only |
| `Agreement` | `scope` `repo` or `org`; `githubRepoId` (unique) or `githubOrgId`; display `ownerName`/`repoName`; `installationId`; `notifyOnSign`; soft delete `deletedAt` |
| `AgreementVersion` | versioned Markdown `text`, `changelog`; unique per (agreement, version) |
| `AgreementField` | custom field: `label`, `dataType` (text, email, url, checkbox, date), `required`, `sortOrder`, `enabled` |
| `Signature` | one per (user, agreement); `versionId`; `signatureType` `individual` or `corporate`; `source` (`online`, manual, import); company name, domain, title; `revokedAt` |
| `FieldEntry` | a field value on a signature |
| `Exclusion` | `type` `bot_auto`, `user` (`githubLogin`) or `team` (`githubTeamId`) |
| `ApiKey` | hashed key (`keyHash`), visible `keyPrefix`, `revokedAt`, `lastUsedAt` |
| `AuditLog` | user, `action`, entity, JSON `before`/`after`, IP |

## Access rules

* Anyone may view a signing page; signing requires a GitHub session.
* Owner pages (`/agreements`, `/agreements/new`, `/agreements/edit/…`,
  `/settings/…`) require `role = owner` (middleware, then `requireOwner()`
  or `getAgreementAccessLevel()` on the server).
* `getAgreementAccessLevel` returns `owner` for the agreement's owner,
  `org_admin` for a GitHub admin of the agreement's organization (read
  access to agreement and signatories), `none` otherwise.
* REST: `Authorization: Bearer clahub_…` API key or a session; owner-only
  resources return 403 to anyone else. Rate limits 100/60/30 requests per
  minute (API key / session / anonymous) with `X-RateLimit-*` headers.
* Webhooks are accepted only with a valid `x-hub-signature-256`.

## What the client may call

* **Server actions** in `src/lib/actions/`: agreement create, update,
  delete, transfer; signing; signature add, import, revoke, restore;
  exclusions; API keys; audit log reads; manual re-check; CONTRIBUTING.md
  snippet. All return `ActionResult`.
* **REST API** `/api/v1/agreements/…`, specified in `public/openapi.yaml`:
  list and CRUD agreements (repo `[owner]/[repo]` and org `[owner]`),
  signatures (list, create), `check/[username]` (public), CSV and PDF
  export.
* **Public:** `GET /api/badge/[owner]/[repo]` and `/api/badge/[owner]`
  (SVG), `GET /api/health`.
* **Owner helpers:** `/api/github/org-teams`, `/api/github/search-users`.

## Webhooks handled

`installation.created|deleted`, `installation_repositories.added|removed`
(update `installationId`), `pull_request.opened|synchronize|reopened` (CLA
check), `push`, `repository.renamed|transferred` (update display names).
