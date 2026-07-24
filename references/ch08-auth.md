# Chapter 8 — Authentication and authorisation

Grounding for the auth part of **Area 5**. Source: *Building Generative AI Services with FastAPI*, Ch 8.

## Authentication vs authorisation

Two separate concepts often confused. **Authentication** verifies identity, that an entity is who it claims to be, via authenticators like passwords, tokens, or keys (OWASP). **Authorisation** verifies that a requested action or resource is permitted for that identity (NIST). Analogy: a passport at immigration is authentication; the visa that grants entry and permitted activities is authorisation.

## Authentication methods (choose by context)

- **Basic** (username/password). Simple, fast, easy to understand. Limitation: sends credentials in plain text. Use: prototypes, internal or non-production environments only.
- **Token / JWT** (access tokens). Scalable, decoupled (suits microservices), can be signed and encrypted, customisable, self-contained (fewer DB round-trips), passed in HTTP headers. Limitations: short-lived tokens need regenerating, client-side storage complexity, token size/bandwidth, stateless tokens make multi-step flows harder, client misconfiguration can compromise them. Use: single-page and mobile apps, custom auth flows, REST APIs.
- **OAuth** (identity provider via OAuth2). Delegates authentication to external providers, standard and battle-tested for enterprise, can access external resources on behalf of the user. Limitations: complex, providers implement the flow differently. Use: apps needing user data from GitHub/Google/Microsoft, enterprise SSO.
- **Key-based** (public/private key pair, SSH-like). Limitations: key management and keeping private keys secure is complex, compromised keys are a risk, scalability issues. Use: small apps, internal environments.

The book implements basic, JWT, and OAuth.

## Authorisation

Verifies permissions of an authenticated identity to access or mutate resources. Implement role-based access control (RBAC) through the FastAPI dependency graph where interactions need restricting or moderating by role.

## Grading calibration (Area 5, auth)

- Match the method to the deployment context. Basic auth is acceptable for a prototype or internal service but a fail for a production or internet-facing one (plain-text credentials). Token/JWT suits SPA, mobile, and REST; OAuth suits enterprise SSO or external identity data.
- Protected routes that mutate data or expose the model should be behind an auth dependency. Where roles matter, check RBAC is enforced via dependencies, not assumed.
