---
title: Security, identity and compliance principles
id: security-identity-principles
doc_type: concept
summary: Track G. Separate authentication from authorisation; learn OAuth 2.0, OIDC, JWT and PKCE with a validation checklist; take identity only from the verified token and check every object access; keep secrets out of code and logs; use keyless, short-lived service identity; scan delivery and uploads; cap abuse per cost vector; speak control and privacy vocabulary.
status: draft
updated: 2026-11-04
audience: engineers joining the team
scope: [knowledge-transfer/]
tags: [security, identity, authentication, authorization, oauth2, oidc, jwt, pkce, tenancy, idor, bola, secrets, iam, workload-identity, supply-chain, scanning, uploads, rate-limiting, compliance, privacy, gdpr]
aliases: [AuthN, AuthZ, RBAC, ABAC, access token, ID token, refresh token, JWKS, token validation, insecure direct object reference, broken object level authorization, least privilege, secrets manager, Workload Identity Federation, SAST, SCA, SLSA, malware scan, token bucket, 429, ISO 27001, SOC 2, erasure]
related: [engineering-principles-map.md, architecture-principles.md, backend-api-principles.md, ai-agent-engineering.md]
group: Run
order: 70
html_twin: security-identity-principles.html
---

# Security, identity and compliance principles

## TL;DR
- Authentication says who you are; authorisation says what you may do. Keep them as separate, ordered checks.
- OAuth 2.0 delegates authorisation; OIDC adds identity with an ID token. Browsers and mobile apps use authorization code with PKCE; the implicit and password grants are out.
- Validate every token: signature against published keys with an allowed algorithm, issuer, audience, expiry, and the token's type.
- Identity comes from the verified token, never from the path, the body or a model's tool arguments; check authorisation on every object access.
- Cap each cost vector separately; limiters may fail open, identity checks never.
- Notes are private study notes and carry no ref; Practice sources are public links.

## G1 Identity and access basics
- Priority: P1 core. Sources: Practice, Notes.
- Principle: authentication (AuthN) proves who you are; authorisation (AuthZ) decides what you may do. Groups or claims become roles; least privilege scopes every role to named resources.
- Principle: RBAC (role-based) grants by role; ABAC (attribute-based) decides from attributes such as tenant, owner or data class. Multi-tenant systems usually need both.
- Notes: task roles scoped to named parameters and keys; wildcard resources on one side counted as a finding.
- In practice: broken authentication, broken object-level, property-level and function-level authorisation are four of the OWASP API Security Top 10 (https://api-security.owasp.org/editions/2023/en/0x11-t10).
- Needs team input: how roles are granted and reviewed [unverified].
- Exercise: list every role your service grants and the resources each can touch.
- Check: why does a wildcard resource in an IAM policy count as a finding?

## G2 OAuth 2.0, OIDC, JWT and PKCE
- Priority: P1 core. Sources: Practice, Lesson.
- Principle, the standards:
  - OAuth 2.0 delegates authorisation (https://datatracker.ietf.org/doc/html/rfc6749); OpenID Connect is an identity layer on top of it, and its ID token is a JWT (https://openid.net/specs/openid-connect-core-1_0.html).
  - Three tokens: the ID token (who signed in, for the client), the access token (what the bearer may call, for APIs), the refresh token (gets new tokens).
  - Flows: authorization code with PKCE for browser and mobile apps (https://datatracker.ietf.org/doc/html/rfc7636); client credentials for service to service.
  - Current best practice: public clients must use PKCE; the implicit grant should not be used; the resource-owner password grant must not be used; public clients' refresh tokens must be sender-constrained or rotated (https://www.rfc-editor.org/rfc/rfc9700).
- Principle, the validation checklist for every JWT (https://www.rfc-editor.org/rfc/rfc8725):
  - Signature against the issuer's published keys (JWKS), with an allow-list of algorithms; never `none`.
  - Issuer, and the subject is valid for your app.
  - Audience: the token was issued for you; reject it otherwise.
  - Expiry and not-before.
  - Type: an ID token is not an access token; use explicit typing and mutually exclusive rules.
- In practice: for an ID token, `aud` must contain your client id (https://openid.net/specs/openid-connect-core-1_0.html).
- Lesson: a verifier configured without the client or audience check accepted ID tokens minted for any app of the same identity pool. Pin the audience.
- Lesson: an API that accepts ID tokens as bearer tokens works, but mixes two token types; APIs should accept access tokens.
- Lesson: a development fallback that decodes unsigned tokens must be fenced twice: off unless an explicit flag is set, and never in production.
- Needs team input: the identity provider and SSO standards for customers [unverified].
- Exercise: draw the PKCE sequence; decode one token and tick each item of the checklist.
- Check: why must a public client use PKCE?

## G3 Tenant isolation and object-level authorisation
- Priority: P1 core. Sources: Practice, Notes, Lesson.
- Principle: identity comes from the verified token, never from the path, the body or the model.
- In practice: an IDOR (insecure direct object reference) needs an object, a reference to it, and a missing object-level check. The fix is an authorisation check on every access, for example by scoping the lookup to the caller's records; random ids help but are not the fix (https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html).
- Notes: the tenant in the path is a log field; a token claim authorises; a hook filters reads (`architecture-principles`, B5).
- Lesson: an AI tool that took a `user_id` parameter would let a prompt choose whose data to read: prompt injection straight into an IDOR. Read the user from the verified session, not from tool arguments.
- Lesson: scope storage keys by owner (`users/<sub>/…`) and reject any key outside the caller's prefix.
- Principle: test cross-tenant and cross-user access explicitly, as their own test set.
- Exercise: write a test that requests another user's object by id.
- Check: why is a `user_id` argument on an agent tool an IDOR?

## G4 Secrets, keys and configuration
- Priority: P2 depth. Sources: Practice, Notes, Lesson.
- In practice: centralise secrets in one manager; automate rotation; least privilege per secret; prefer short-lived, dynamic credentials; audit every access; keep secrets out of code, images, pipeline output and logs (https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html).
- Notes: customer-managed encryption keys rotated per environment; one encrypted parameter per tenant per environment.
- Lesson: a public-by-design value (a web API key that authorises nothing alone) is not a secret and may sit in browser config; a provider token is a secret and is injected at runtime.
- Exercise: search one service's logs for anything that looks like a token.
- Check: why is a web API key in public config acceptable when a provider token is not?

## G5 Service-to-service identity
- Priority: P2 depth. Sources: Practice, Notes, Lesson.
- Principle: services authenticate with short-lived, keyless identity; no long-lived keys on disk.
- Notes: role-based task identity; pipelines that assume a role into each target account.
- In practice: Workload Identity Federation lets a workload on another cloud or identity provider exchange its own credential for a short-lived Google Cloud token, with no service account key (https://docs.cloud.google.com/iam/docs/workload-identity-federation).
- Notes: across clouds, an AWS role trusts Google's token, so no key is stored on either side.
- Lesson: if a protocol hop does not carry the end user's identity, expose over it only tools that need none, such as read-only public data.
- Exercise: list every credential your service holds and its lifetime.
- Check: what does keyless federation remove from the attack surface?

## G6 Network and edge security
- Priority: P2 depth. Sources: Notes, Lesson.
- Principle: a modern TLS policy at the edge; deny non-TLS requests to log buckets; security groups by source group; block public access on upload buckets; serve private objects through short-lived signed URLs.
- Lesson: signing URLs from a cloud service account may need the account to be allowed to sign as itself; grant that one permission, not a broad role.
- Principle: a web application firewall and DDoS protection in front of public endpoints.
- Exercise: check one bucket for public access and a TLS-only policy.
- Check: why sign URLs instead of making a bucket public?

## G7 Secure delivery and the supply chain
- Priority: P2 depth. Sources: Notes, Practice.
- Principle: SAST on code, SCA on dependencies and a quality gate on every merge; an external pen test every year.
- Notes: a full SAST scan every time rather than an incremental one, "because an incremental scan inherits the blind spots of the baseline it diffs against".
- In practice: SLSA is a framework of levels and controls for supply-chain integrity, starting with build provenance (https://slsa.dev/).
- Principle: secret scanning on push; signed images verified at deploy.
- Exercise: read one scanner report and triage its top finding.
- Check: why is a full SAST scan safer than an incremental one?

## G8 Uploads and content safety
- Priority: P2 depth. Sources: Practice, Notes.
- In practice: allow-list file types and check the real type, not the `Content-Type`; rename files on the server; cap size, including decompressed size; require authorisation; store outside the web root or on a separate host; scan, and disarm documents (https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html).
- Notes: an upload landed where nothing else could read it, was scanned, then a clean object was copied onward and anything else deleted. No copy was kept, so a true positive left nothing to investigate and a false positive nothing to restore. Keep a real quarantine.
- Principle: on output, escape or strip active content before rendering; markdown images and links are a classic exfiltration path.
- Exercise: design a real quarantine: where the copy goes, who can read it, when it expires.
- Check: what do you lose when a scan deletes instead of quarantining?

## G9 Abuse limits
- Priority: P2 depth. Sources: Practice, Lesson.
- Principle: cap each cost vector separately, check before the expensive work runs, and answer `429 Too Many Requests`, optionally with `Retry-After` (https://www.rfc-editor.org/rfc/rfc6585).
- In practice: unrestricted resource consumption is an OWASP API Security Top 10 risk (https://api-security.owasp.org/editions/2023/en/0x11-t10).
- Lesson, three limiters that worked together for an AI feature:
  - A per-user token bucket on the chat endpoint, refilled lazily at check time so nothing runs between requests.
  - A daily token cap, rolling at UTC midnight.
  - A separate daily image cap, because images were the dominant cost.
- Lesson: run guardrails before the model, because a check after the response has already paid for it; limiters fail open and the daily caps backstop abuse.
- Exercise: compute the worst-case monthly cost of one user under your caps.
- Check: why two daily caps instead of one?

## G10 Compliance and audit vocabulary
- Priority: P3 reference. Source: Notes.
- Frameworks: ISO 27001 (a certifiable information-security management system); SOC 2 Type I (designed well) versus Type II (worked over time); GDPR; NIST CSF; the EU AI Act and ISO/IEC 42001 (`ai-retrieval-evals-cost`, H17).
- Control vocabulary: a control and its objective; design versus operating effectiveness; evidence; IT general controls; the three lines; population versus sample; segregation of duties; audit trail; attestation.
- Principle: an AI feature is in scope of the controls that already exist.
- Needs team input: who owns the ISO 27001 controls and how they are evidenced [unverified].

## G11 Privacy and deletion
- Priority: P2 depth. Source: Notes.
- Principle: redact personal data at ingest and again at prompt time; pseudonymise where you can; redact logs on write, sample, keep retention short; know whether you are the controller or the processor.
- Principle: erasure cascades through chunks, vectors, caches, chat histories, golden sets and fine-tuning data; keep lineage pointers from day one.
- Exercise: list every derived copy of one user's data in your system.
- Check: a user asks to be erased. What must you delete besides the source row?

## Invariants
- NEVER take identity from a path, a body field or a model's tool arguments (G3).
- NEVER let authentication, authorisation or tenancy fail open (`data-distributed-systems`, E7).
- NEVER accept a token without checking its audience (G2).

## Gotchas
- A verifier without an audience check accepts tokens issued to other clients (G2).
- An OAuth `state` check that runs only when both values exist can be skipped (`frontend-react-principles`, D7).
- A scan that deletes instead of quarantining destroys the evidence (G8).

## Glossary
- AuthN, AuthZ — authentication, authorisation.
- PKCE — proof key for code exchange.
- JWKS — the issuer's published signing keys.
- IDOR, BOLA — insecure direct object reference; broken object-level authorisation.
- WIF — Workload Identity Federation.
- SAST, SCA — static application security testing; software composition analysis.

## Open questions
- The team's identity provider, SSO standards and role-review process [unverified].

## Sources
- https://www.rfc-editor.org/rfc/rfc9700 — OAuth 2.0 security best current practice.
- https://www.rfc-editor.org/rfc/rfc8725 — JWT best current practices.
- https://openid.net/specs/openid-connect-core-1_0.html — OpenID Connect Core.
- https://api-security.owasp.org/editions/2023/en/0x11-t10 — OWASP API Security Top 10 (2023).
- https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html — IDOR prevention.
- https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html — secrets management.
- https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html — file uploads.
