# Domain

| Term | In code | Meaning |
|---|---|---|
| Agreement | `Agreement` | the CLA of one repository or one organization |
| Scope | `Agreement.scope` (`repo`, `org`) | whether the agreement covers one repository or every repository of an organization |
| Version | `AgreementVersion` | one published text of an agreement; signing always uses the latest |
| Field | `AgreementField` | a custom input a signer fills (text, email, url, checkbox, date) |
| Field entry | `FieldEntry` | the value a signer gave for a field |
| Owner | `User.role = "owner"`, `Agreement.ownerId` | the user who manages an agreement; signed in through the owner OAuth app |
| Contributor | `User.role = "contributor"` | a user who signs; signed in through the contributor OAuth app |
| Org admin | `AccessLevel "org_admin"` | GitHub admin of the agreement's organization, with read access |
| Signature | `Signature` | a user's acceptance of an agreement at a version |
| Individual signature | `signatureType = "individual"` | covers the signer only |
| Corporate signature | `signatureType = "corporate"` | signed by a company representative; covers authors whose email domain equals `companyDomain` |
| Source | `Signature.source` | how the signature arrived: online, manual entry, CSV import |
| Revocation | `Signature.revokedAt` | a signature withdrawn by the owner; can be restored |
| Exclusion | `Exclusion` (`bot_auto`, `user`, `team`) | an author exempt from the CLA |
| Commit author | `AuthorInfo` | githubId, login and email taken from a PR's commits |
| Check | `CheckResult`, check run | classification of a PR's authors and the GitHub check posted from it |
| Re-check | `recheckOpenPRs` | re-running the check on every open PR of an agreement |
| Installation | `Agreement.installationId` | the GitHub App installation that lets CLAHub act on the repository |
| API key | `ApiKey`, prefix `clahub_` | a bearer token for the REST API, stored hashed |
| Audit log | `AuditLog`, `logAudit` | the record of every mutation, with before and after |

## Entities and invariants

* One signature per user per agreement (`@@unique([userId, agreementId])`);
  re-signing after revocation is allowed.
* An agreement can be signed only when it has at least one version and is
  not deleted.
* A repo agreement is identified by `githubRepoId`; `ownerName` and
  `repoName` are display fields kept current by webhook events.
* A PR author is classified in this order: excluded, signed (non-revoked
  signature of the matched user), corporate-covered (email domain), unsigned.
  Users are matched by githubId, then email, then login.
* A PR check succeeds exactly when no author is unsigned.
* Every mutation writes an audit log entry in the same transaction.
* API keys are never stored in clear.
