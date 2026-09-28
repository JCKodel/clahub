# Queue

## Milestone 1: current signatures, owner emails, personal data

Closes when an owner can require contributors to re-sign after a new CLA
version, with an optional grace period, and PR checks honour it; when the
notification email sent on signing uses the owner's own subject and body,
falling back to the default; and when any user can download their personal
data as one JSON file and delete their account. Source: upstream issues
DamageLabs/clahub #270, #274 and #268, in that order.

```
[ ] resign-on-version-bump       owners can require re-signing when a new CLA version is published, with an optional grace period (#270)
[ ] custom-email-templates       owners customize the subject and body of the new-signature email with variables and a preview (#274)
[ ] gdpr-export-and-deletion     users download their data as JSON and delete their account (#268)
```
