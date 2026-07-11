🟦 1. User Identities & Hybrid Identity (20–25%)
Identity Models
PHS — simplest, cloud‑based auth, supports CA & Identity Protection.

PTA — cloud agent validates passwords against AD.

Federation — AD FS, token issuance on‑prem, complex.

Entra Connect vs Cloud Sync
Feature	Entra Connect	Cloud Sync
Sync engine	On‑prem	Cloud‑based
Writeback	Yes	Limited
Scale	Single forest	Multi‑forest
Best for	Complex orgs	Lightweight, modern


External Identities
B2B — guest users, governed by CA, PIM, access packages.

Cross‑tenant access — inbound/outbound rules, trust settings.

B2B Direct Connect — Teams shared channels.

Custom Security Attributes
Custom key‑value pairs for apps.

Assigned to users; governed by attribute sets.

Used in dynamic groups.

🟦 2. Authentication & Access Management (25–30%)
Conditional Access — Core Concepts
Assignments: Users/groups, cloud apps.

Conditions: Sign‑in risk, device, platform, location.

Access Controls: Grant (MFA, compliant device), Session (sign‑in frequency, token protection).

Enable policy: Always test with Report‑only first.

High‑Yield CA Policies
Require MFA for all users.

Block legacy authentication.

Require compliant device for admins.

Require passwordless for privileged roles.

Terms of Use for external users.

Token Protection
Prevents token replay.

Requires supported apps/platforms.

Continuous Access Evaluation (CAE)
Real‑time session revocation.

Works with Exchange, SharePoint, Teams.

Authentication Methods
Passwordless: FIDO2, WHfB, Authenticator.

Temporary Access Pass (TAP): bootstrap method.

MFA: phone, app, FIDO2.

Identity Protection
Risk types: User risk, sign‑in risk.

Policies:

User risk → force password reset.

Sign‑in risk → require MFA.

Risk detections: Impossible travel, unfamiliar sign‑in, malware‑linked IP.

🟦 3. Workload Identities & App Access (15–20%)
App Registrations
Delegated permissions — user present.

Application permissions — daemon/service, admin consent required.

Client secrets — short‑lived; certificates preferred.

Redirect URIs — required for OAuth flows.

Enterprise Apps
SSO types:

SAML

OIDC

Password‑based

Linked

Provisioning: SCIM → automated user lifecycle.

Application Proxy
Publish internal apps externally.

Connector installed on‑prem.

Pre‑authentication via Entra ID.

Consent Governance
User consent — limited, controlled by policy.

Admin consent workflow — approval process for app permissions.

Managed Identities
System‑assigned — tied to resource lifecycle.

User‑assigned — reusable across resources.

Used for Azure RBAC + token acquisition.

🟦 4. Identity Governance (25%)
Entitlement Management
Catalogs — group apps/resources.

Access packages — bundles of access.

Policies — request, approval, lifecycle.

Connected organizations — external partners.

Access Reviews
Review access for:

Groups

Apps

Privileged roles

Recurrence supported.

Auto‑apply results (remove access).

Privileged Identity Management (PIM)
Eligible — must activate.

Active — currently assigned.

Activation settings: MFA, justification, approval, ticket.

Alerts: Too many owners, permanent assignments.

Lifecycle Workflows
Joiner — onboarding tasks.

Mover — role/department changes.

Leaver — disable account, remove access.

Uses event‑based triggers.

Terms of Use
PDF uploaded.

Enforced via Conditional Access.

Track acceptance records.

⭐ SC‑300 “Must‑Know” Tables
Authentication Method Comparison
Method	Best For	Notes
FIDO2	High security	Phishing‑resistant
WHfB	Windows devices	Uses biometrics
Authenticator	Mobile‑based	Push notifications


CA Conditions
Condition	Example
User risk	High‑risk user → block
Sign‑in risk	Medium risk → MFA
Device	Compliant device required
Location	Block non‑trusted countries


PIM Role Types
Role Type	Scope
Entra roles	Tenant‑wide
Azure roles	Resource‑specific


⭐ SC‑300 “Trick Questions” You Must Expect
PHS does not require AD FS.

Cloud Sync does not support writeback.

CA report‑only does not enforce controls.

Identity Protection requires P1/P2 licensing.

Managed identities cannot be used outside Azure.

Access packages can be assigned to external users.

PIM eligible ≠ active — exam loves this distinction.