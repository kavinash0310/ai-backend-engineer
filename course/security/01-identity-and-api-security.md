# Identity, authorization, and API security

## Authentication and authorization are different

Authentication asks, “Who is making this request?” Authorization asks, “May this identity perform this action on this resource?” A valid login does not mean a user can read every document or invoke every tool.

A session cookie or bearer token can carry or reference identity claims. A JWT is a signed data format, not encryption by default and not proof that its claims are correct unless signature, issuer, audience, expiry, and key rotation are verified. Use established identity libraries and a trusted identity provider rather than inventing password or token protocols.

OAuth is an authorization framework; OpenID Connect adds an identity layer. Do not place a provider API key in browser code. The backend should keep provider credentials server-side and apply its own user limits.

## Object-level authorization

Every database read or write that uses a caller-supplied resource ID must check ownership or permission. This prevents broken object-level authorization, sometimes called IDOR/BOLA. Checking only that a user is logged in is insufficient.

Instead of loading document 123 and assuming it is theirs, query by both document ID and authenticated owner or evaluate a policy that includes tenant membership and role. Apply checks to list, download, delete, feedback, and background-job status endpoints. A job ID is also an object ID and needs authorization.

Validate allowed properties. Accepting arbitrary JSON and mass-assigning it to a database model can let a client change fields such as owner, role, or billing status. Define request schemas and explicit updateable fields.

## Least privilege and secrets

Grant each service account only the permissions required. A worker that reads documents need not administer users. Rotate keys, restrict scopes, and keep secrets in a managed secret store or protected deployment environment. Avoid logging authorization headers, raw prompts containing personal data, or provider responses with sensitive content.

Set request and upload limits, rate limits, and timeouts. Unrestricted resource consumption is a security risk and a cost risk, especially when each request can trigger expensive model tokens or retrieval operations.

## Common API threat categories

OWASP API risks include broken object/function-level authorization, broken authentication, unrestricted resource consumption, sensitive business flow abuse, server-side request forgery, security misconfiguration, improper inventory, and unsafe consumption of APIs. Treat this as a review guide, not a checklist that guarantees security.

For SSRF, never fetch arbitrary user-provided URLs from a privileged backend without strict controls. Attackers may target internal network services or cloud metadata endpoints. Allowlist hosts where possible, block private/link-local destinations, validate redirects and DNS results, and enforce network egress policy.

## Questions and practice

**If a JWT is signed, can I trust every claim?** Only after validating signature and relevant issuer, audience, time, and key assumptions. A signed token is not automatically encrypted.

**Why check authorization at the object query?** It reduces the chance that a forgotten later check exposes another user's data.

Practice: create two test users and two private documents. Attempt read, list, update, delete, and job-status operations with the wrong identity. Every operation should deny access without revealing whether sensitive records exist beyond the product's intended behavior.

Further reading: [OWASP API Security Top 10](https://api-security.owasp.org/editions/2023/en/0x11-t10/) and [OAuth 2.0 security best current practice, RFC 9700](https://datatracker.ietf.org/doc/html/rfc9700).

## Verify tokens and permissions at every boundary

A bearer token is a credential: whoever holds it can act until expiry or revocation. Do not log it. When accepting a JWT, validate its signature, permitted algorithm, issuer, audience, expiry, and key source. Decoding its JSON is not validation. OAuth access tokens authorize access; OpenID Connect ID tokens describe authentication to a client. Do not swap their roles.

Authorization should evaluate caller, action, resource, tenant, and context. A role such as member is coarse; a document-specific permission is finer. Every endpoint that loads a caller-provided ID needs ownership/permission enforcement: list, detail, update, delete, search, download, and job status. The UI hiding a button is not an API control.

Use explicit input schemas so a caller cannot mass-assign fields such as owner_id, role, or billing status. Apply least privilege to workers and service accounts. Rate-limit and cap request size, especially when an operation can trigger model costs.

Questions: Is a user ID supplied in JSON proof of identity? No; derive identity from a validated credential. Is CORS authorization? No; it is a browser access policy and does not stop direct API calls.

Practice: create two users and test cross-user read, list, update, deletion, and job-status routes. Also test changing owner_id in an update request.
