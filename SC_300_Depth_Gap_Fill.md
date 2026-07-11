# SC-300 Depth Gap-Fill Guide
### Advanced Scenario-Level Coverage for Exam Success
> Companion to your existing Cheat Sheet, Study Plan, and Lab Checklist.
> This document covers **what your current materials miss** — the edge cases, behavioral nuances, and scenario reasoning the exam actually tests.

---

## How to Use This Document

Your existing materials answer: *"What is X?"*
This document answers: *"How does X behave, why would you choose it, and what breaks when it's misconfigured?"*

Read each section **after** completing the corresponding Microsoft Learn module. Use the scenario callouts (marked ⚠️) to test your reasoning before reading the answer.

---

---

# DOMAIN 1 — Implement & Manage User Identities (20–25%)
## Gap Areas Not Covered in Your Cheat Sheet

---

### 1.1 Entra Connect — Behavioral Edge Cases

#### Attribute Precedence & Conflict Resolution
When the same user exists in multiple on-premises forests, Entra Connect must determine which forest "wins" for each attribute. This is governed by **precedence rules** in the metaverse.

- The default precedence order is set during installation; Forest 1 wins over Forest 2 for conflicting attributes unless you customize rules.
- You can create **custom synchronization rules** in the Sync Rules Editor to override attribute flow.
- **Source Anchor** (`ms-DS-ConsistencyGuid` or `objectGUID`) must be unique across forests — misconfiguration causes duplicate object errors in Entra ID.

⚠️ **Scenario:** Two forests sync the same user (matched on email). Forest A has `department = Finance`, Forest B has `department = Accounting`. The CA policy targets the Finance group via dynamic membership. The user is missing from the group. Why?
> The forest with lower precedence (Forest B) is winning for the `department` attribute due to a custom rule misconfiguration. Fix: review sync rule precedence in Synchronization Rules Editor.

#### Soft Match vs Hard Match
- **Soft match:** Entra Connect links an existing cloud user to an on-premises object by matching `proxyAddresses` or `userPrincipalName`. Used during initial hybrid setup.
- **Hard match:** Uses the `ImmutableID` (base64-encoded `objectGUID` or `ms-DS-ConsistencyGuid`). Required when soft match fails or when recovering from a broken sync.
- If both a cloud user and a synced object exist for the same person, the objects will **collide** unless you set the ImmutableID on the cloud object before sync.

📖 https://learn.microsoft.com/entra/identity/hybrid/connect/how-to-connect-syncservice-duplicate-attribute-resiliency

---

### 1.2 Cloud Sync — Limitations the Exam Tests

| Feature | Entra Connect | Cloud Sync |
|---|---|---|
| Group writeback | Yes (v2) | No |
| Device writeback | Yes | No |
| Exchange hybrid writeback | Yes | No |
| Password writeback | Yes | Yes (with SSPR) |
| Provisioning to multiple tenants | No | Yes |
| OU filtering | Yes | Yes |
| Attribute mapping customization | Full | Limited |

**Key exam trap:** Cloud Sync supports **password writeback** (users can reset passwords from cloud and write back to AD) but does NOT support group writeback or Exchange hybrid.

📖 https://learn.microsoft.com/entra/identity/hybrid/cloud-sync/what-is-cloud-sync

---

### 1.3 Dynamic Groups — Full Behavioral Rules

#### Rule Evaluation Triggers
Dynamic group membership is re-evaluated when:
- A user attribute changes
- The group rule is modified
- Manual re-evaluation is triggered via PowerShell (`Update-MgGroup`)

Processing time: up to **24 hours** for large tenants (10,000+ users). Not suitable for time-sensitive access scenarios.

#### Case Sensitivity (High-Frequency Exam Trap)
- `-eq` operator: **case-sensitive** for synced attributes, case-insensitive for cloud-only attributes in some scenarios — always treat as case-sensitive to be safe.
- `-match` operator: uses regex, case-insensitive by default.
- `-contains` operator: checks if the attribute value **contains** the string — not a whole-word match. `"Sr. Analyst"` **does** match `-contains "Analyst"`.

#### Dynamic Groups Cannot
- Have members manually added
- Be used as security groups for on-premises resources (cloud-only scope)
- Be used as role-assignable groups

📖 https://learn.microsoft.com/entra/identity/users/groups-dynamic-membership

---

### 1.4 Administrative Units — Deep Behavior

#### Restricted Management Administrative Units (RMAUs)
This is a gap in your cheat sheet. RMAUs are a **separate, advanced AU type** with distinct behavior:

- Objects in an RMAU can **only** be managed by admins scoped to that RMAU — even Global Admins are blocked by default.
- Use case: protect break-glass accounts, VIP executives, or high-value service accounts from accidental modification by tenant-wide admins.
- To manage an object in an RMAU, you must be explicitly assigned a role scoped to that RMAU.

⚠️ **Scenario:** A Global Admin tries to reset the password of a user in an RMAU and receives an "Insufficient privileges" error. Is this expected?
> Yes. RMAUs block even Global Admins unless they are explicitly scoped to the RMAU. This is by design.

#### Dynamic AU Membership
- Membership can be driven by a rule on `user.country`, `user.department`, etc.
- Devices and groups **cannot** use dynamic membership in AUs — only users.
- Changes take effect within ~24 hours, same as dynamic groups.

📖 https://learn.microsoft.com/entra/identity/role-based-access-control/admin-units-restricted-management

---

### 1.5 External Identities — Cross-Tenant Access Deep Dive

Your cheat sheet mentions cross-tenant access but doesn't cover the trust settings, which are heavily tested.

#### Trust Settings (Inbound)
You can configure your tenant to **trust MFA claims** from a partner tenant, meaning:
- If the partner tenant already completed MFA, your CA policy requiring MFA is satisfied without prompting again.
- You can also trust **compliant device** and **Hybrid Azure AD Joined** claims from partner tenants.

#### B2B Direct Connect vs B2B Collaboration

| | B2B Collaboration | B2B Direct Connect |
|---|---|---|
| Account created in your tenant | Yes (guest account) | No |
| Governed by your CA policies | Yes | Limited |
| Use case | General guest access | Teams shared channels only |
| Access packages | Yes | No |

⚠️ **Scenario:** Contoso wants to collaborate with Fabrikam on a Teams shared channel. Fabrikam users should not have guest accounts created in Contoso's tenant. What should you configure?
> B2B Direct Connect via cross-tenant access inbound/outbound settings. No guest accounts are created.

📖 https://learn.microsoft.com/entra/external-id/cross-tenant-access-overview

---

---

# DOMAIN 2 — Authentication & Access Management (25–30%)
## Gap Areas Not Covered in Your Cheat Sheet

---

### 2.1 Conditional Access — Advanced Scenarios

#### Authentication Strengths
This is **not in your cheat sheet** and is an active exam target.

Authentication Strengths are named policies that define **which authentication methods satisfy a CA grant control**, replacing the blunt "Require MFA" with precise method requirements.

- **Built-in strengths:**
  - Multifactor authentication (any MFA method)
  - Passwordless MFA (Authenticator, FIDO2, WHfB)
  - Phishing-resistant MFA (FIDO2, WHfB, certificate-based auth only)

- **Custom strengths:** You can create your own, e.g., "FIDO2 or WHfB only."

- **When to use:** Privileged roles or sensitive apps should require **Phishing-resistant MFA**, not just any MFA. The exam will present scenarios where Authenticator push satisfies MFA but NOT phishing-resistant MFA.

📖 https://learn.microsoft.com/entra/identity/authentication/concept-authentication-strengths

#### CA for Workload Identities
CA policies can now target **service principals** (workload identities), not just users. This is tested in Domain 3 but configured in the CA blade.

- Conditions available: IP location only (no device, no sign-in risk for service principals in most cases)
- Useful for: restricting which IPs a daemon app can authenticate from.

📖 https://learn.microsoft.com/entra/identity/conditional-access/workload-identity

#### Device Filter Conditions
Your cheat sheet lists "device filters" without explaining the syntax. The exam tests this.

- Filters use **Rule Expression Syntax**, e.g.:
  - `device.trustType -eq "ServerAD"` — Hybrid Azure AD joined only
  - `device.isCompliant -eq True` — Compliant devices
  - `device.extensionAttribute1 -eq "Privileged"` — Custom attribute filter
- You can **include or exclude** devices matching the filter in a single policy.
- Use case: apply stricter controls to devices NOT matching the filter.

📖 https://learn.microsoft.com/entra/identity/conditional-access/concept-condition-filters-for-devices

#### Sign-in Frequency vs Persistent Browser Session

These are two separate session controls that are **frequently confused on the exam.**

| Control | What it does | Default |
|---|---|---|
| Sign-in frequency | Forces re-authentication after N hours/days | Entra token lifetime |
| Persistent browser session | Controls whether browser sessions remain signed in after closing | Determined by "Stay signed in?" prompt |

- Setting sign-in frequency to 1 hour means the user re-authenticates every hour — even if their session is active.
- "Always persistent" browser session + short sign-in frequency = frequent MFA prompts.
- For **Privileged Access Workstations**, set sign-in frequency to **every time** (every session).

⚠️ **Scenario:** Admins complain they must MFA every time they open a new browser tab for the Azure portal. Which CA control is causing this?
> Sign-in frequency set too aggressively on the policy targeting Azure Management. Review and set an appropriate frequency (e.g., 4 hours for admins) or scope to specific high-risk actions.

#### Named Locations vs Trusted Locations vs Compliance-Marked

- **Named locations:** IP ranges or countries you define — used in CA conditions.
- **Trusted locations (MFA trusted IPs):** Legacy per-user MFA setting — bypasses MFA from specific IPs.
- **Mark as trusted:** A checkbox on a named location that causes it to count as "trusted" in sign-in risk evaluation, reducing risk score for sign-ins from that IP.

**Exam trap:** Marking a location as "trusted" does NOT automatically bypass CA policies — it only influences the **risk score**. You must explicitly create a CA exclusion to bypass CA.

📖 https://learn.microsoft.com/entra/identity/conditional-access/location-condition

---

### 2.2 Continuous Access Evaluation (CAE) — Deep Behavior

Your cheat sheet says "real-time session revocation" — the exam tests the mechanics.

#### How CAE Works
- Normally, access tokens are valid for **60–90 minutes** regardless of what happens to the user after issuance.
- CAE-enabled apps (Exchange Online, SharePoint, Teams, Graph) **subscribe to events** from Entra ID and can reject tokens immediately when:
  - User account is disabled
  - User password is changed
  - User session is revoked
  - User location changes to a non-trusted IP (if IP-binding is enforced)

#### CAE Limitations
- Only works with **CAE-capable clients** (modern Office apps, browser sessions in supported browsers).
- Legacy clients (SMTP, IMAP, older Outlook) do not support CAE — tokens remain valid until expiry.
- Revocation propagation can take up to **15 minutes** even with CAE (near real-time, not instant).

⚠️ **Scenario:** A user's account is disabled in Entra ID at 2:00 PM. At 2:05 PM they can still access SharePoint Online in their browser. Is this a misconfiguration?
> Not necessarily. CAE propagation can take up to 15 minutes. If they're still accessing at 2:20 PM, investigate whether the client is CAE-capable and whether token protection is enforced.

📖 https://learn.microsoft.com/entra/identity/conditional-access/concept-continuous-access-evaluation

---

### 2.3 Identity Protection — Full Scenario Reasoning

#### Risk Remediation vs Dismissal — Critical Distinction

| Action | Effect on Risk | When to Use |
|---|---|---|
| Remediate (user resets password) | Clears user risk | User was genuinely compromised, password changed |
| Dismiss risk | Closes the risk without remediation | False positive confirmed by investigation |
| Confirm compromise | Elevates risk, triggers additional controls | SOC confirmed the account was compromised |

**Exam trap:** Dismissing a risk does NOT change the user's password or revoke sessions. If a user was actually compromised and you dismiss the risk instead of remediating, the attacker may still have access.

#### Offline vs Real-Time Risk Detections

| Detection Type | When Evaluated | Example |
|---|---|---|
| Real-time | At sign-in | Unfamiliar sign-in properties, anonymous IP |
| Offline | After sign-in (hours/days later) | Impossible travel, leaked credentials, malware-linked IP |

**Why this matters for CA policy design:**
- A CA policy checking sign-in risk only catches **real-time** detections at the moment of sign-in.
- Offline detections elevate **user risk** — your CA user risk policy must be enabled to catch these.
- If you only have sign-in risk CA policy and no user risk CA policy, offline-detected compromised accounts can continue signing in unblocked.

#### Identity Protection vs CA Risk Conditions

These overlap but are **separate mechanisms**:

| | Identity Protection Policies (legacy) | CA with Risk Conditions (modern) |
|---|---|---|
| Location | Identity Protection blade | CA blade |
| Flexibility | Low (fixed: block or MFA) | High (any CA grant/session control) |
| Recommended | No (deprecated path) | Yes |
| Scope | All users or targeted | Fully scoped |

Microsoft's current recommendation: **disable Identity Protection risk policies** and use CA policies with risk conditions instead. The exam tests this distinction.

📖 https://learn.microsoft.com/entra/id-protection/concept-identity-protection-risks

---

### 2.4 Passwordless — Edge Cases

#### FIDO2 Attestation
- Organizations can restrict which FIDO2 keys are allowed via **FIDO2 security key restrictions** in Authentication Methods.
- You can allow/block specific key models using their AAGUID (authenticator attestation GUID).
- Use case: enterprise wants to allow only YubiKey 5 series, not cheaper unverified keys.

#### Windows Hello for Business Deployment Models

| Model | Requirement | Use Case |
|---|---|---|
| Cloud-only | Entra ID joined devices | Pure cloud orgs |
| Hybrid | Entra Connect + PKI or cloud trust | Hybrid domain joined |
| On-premises | AD + PKI | No cloud dependency |

**Cloud trust** (recommended for hybrid) eliminates the need for on-premises PKI by using Entra ID as the trust anchor. The exam tests when to use cloud trust vs certificate trust.

📖 https://learn.microsoft.com/entra/identity/authentication/concept-passwordless

---

---

# DOMAIN 3 — Workload Identities & App Access (15–20%)
## Gap Areas Not Covered in Your Cheat Sheet

---

### 3.1 App Permissions — Full Depth

#### Delegated vs Application Permissions

| | Delegated | Application |
|---|---|---|
| User present? | Yes | No (daemon/service) |
| Effective permission | Intersection of user's permissions and app's requested permissions | Full permission as granted |
| Consent | User or admin | Admin only |
| Token type | Access token with user claims | Access token without user claims |

⚠️ **Scenario:** An app has `Mail.Read` delegated permission. A user with no mailbox is signed in. Can the app read another user's mail?
> No. Delegated permission is bounded by what the signed-in user can do. If the user has no mailbox, the app gets nothing — even though the permission is granted.

#### Consent on Behalf of Organization
- When an admin grants consent for an application permission or delegates on behalf of all users, a **service principal** is created in the tenant with the granted permissions.
- This consent appears in Enterprise Apps > Permissions.
- Revoking consent: delete the service principal's permission grant, or remove the enterprise app.

#### Admin Consent Workflow
When users request permissions they can't self-consent to:
1. User is blocked and redirects to a request form.
2. Designated reviewers (set in Enterprise Apps > User settings) receive an email.
3. Reviewer approves or denies in the portal.
4. User is notified.

**Exam trap:** The admin consent workflow approvers are configured per-tenant, not per-app. Setting a reviewer in one app's settings applies to all requests unless the setting is scoped.

📖 https://learn.microsoft.com/entra/identity-platform/howto-configure-admin-consent-workflow

---

### 3.2 Managed Identities — Scenario Reasoning

#### System-Assigned vs User-Assigned

| | System-Assigned | User-Assigned |
|---|---|---|
| Lifecycle | Tied to resource | Independent |
| Sharing | One resource only | Multiple resources |
| Use case | Single resource needing identity | Many resources sharing same permissions |

⚠️ **Scenario:** 50 Azure Functions all need read access to the same Key Vault. Should you use system-assigned or user-assigned managed identity?
> User-assigned. Create one managed identity, assign Key Vault Reader to it, then assign that identity to all 50 functions. Using system-assigned would require 50 separate RBAC assignments.

#### Workload Identity Federation
This is **not in your cheat sheet** and is increasingly tested.

Workload identity federation allows **external workloads** (GitHub Actions, Kubernetes, AWS) to authenticate to Entra ID without a client secret or certificate by establishing a trust relationship:
1. Register an app in Entra ID.
2. Configure a federated credential on the app pointing to the external IdP (e.g., GitHub's OIDC issuer).
3. The external workload exchanges its identity token for an Entra access token.

**Why it matters:** Eliminates long-lived secrets in CI/CD pipelines. Zero Trust for non-human identities.

📖 https://learn.microsoft.com/entra/workload-id/workload-identity-federation

---

### 3.3 Application Proxy — Architecture Detail

#### Connector Groups
Your cheat sheet mentions connectors but not **connector groups**, which are tested.

- Connectors can be grouped by location or application type.
- Each published application is assigned to a connector group.
- Use case: publish an app hosted in a European datacenter through connectors installed in Europe, reducing latency.
- A connector group must have at least one active connector for the published app to be available.

#### Pre-Authentication Methods

| Method | Behavior | Use Case |
|---|---|---|
| Entra ID (default) | User must authenticate to Entra ID before reaching the app | Most apps, enforces CA |
| Passthrough | No Entra authentication required before reaching app | Apps with their own auth, legacy systems |

**Exam trap:** If pre-authentication is set to **Passthrough**, CA policies are NOT enforced for that app. Anyone with the external URL can reach the app's own login page.

#### Single Sign-On Modes for App Proxy

- **Kerberos Constrained Delegation (KCD):** SSO for apps using Windows Integrated Authentication. Requires the connector to be domain-joined and an SPN to be configured.
- **Header-based SSO:** App receives HTTP headers (e.g., `X-Forwarded-User`) populated by the connector.
- **SAML:** App Proxy can act as SAML relay.
- **Password-based:** Connector injects credentials (least preferred).

📖 https://learn.microsoft.com/entra/identity/app-proxy/application-proxy

---

---

# DOMAIN 4 — Identity Governance (25%)
## Gap Areas Not Covered in Your Cheat Sheet

---

### 4.1 Lifecycle Workflows — Full Coverage

This domain is **significantly under-covered** in your existing materials. Lifecycle Workflows are a high-priority exam area.

#### Trigger Types

| Trigger | When it fires | Example |
|---|---|---|
| Time-based (relative to attribute) | N days before/after an attribute date | 7 days before `employeeHireDate` |
| On-demand | Manually triggered | Testing, exceptions |
| Real-time (attribute change) | When a user attribute changes | Department change triggers mover workflow |

#### Built-in Workflow Templates

| Category | Template | Key Tasks |
|---|---|---|
| Joiner | Pre-hire onboarding | Send welcome email, generate TAP, add to groups |
| Joiner | New hire onboarding | Enable account, assign licenses |
| Mover | Department change | Add/remove group memberships |
| Leaver | Last day | Disable account, remove group memberships |
| Leaver | Post-offboarding | Delete account, remove licenses |

#### Built-in Tasks (Must Know)
- `Enable user account`
- `Disable user account`
- `Delete user account`
- `Add user to group`
- `Remove user from group`
- `Send welcome email`
- `Generate Temporary Access Pass` — critical for joiner workflow; new hires get a TAP to bootstrap MFA registration
- `Send offboarding email`
- `Remove user from all groups`
- `Remove user from all teams`

#### Custom Task Extensions
When built-in tasks aren't enough, you can call a **Logic App** via a custom task extension:
- The Logic App is registered as a custom extension in the workflow.
- Triggered synchronously or asynchronously.
- Use case: provision accounts in non-SCIM systems, send tickets to ITSM tools, notify HR systems.

⚠️ **Scenario:** When a new hire joins, their account must be created in ServiceNow. No SCIM provisioning exists. Which lifecycle workflow component handles this?
> A custom task extension invoking an Azure Logic App that calls the ServiceNow API.

📖 https://learn.microsoft.com/entra/id-governance/workflows-overview

---

### 4.2 PIM — Deep Scenario Behavior

#### Activation Settings (Per Role)
Each role in PIM has configurable activation settings:
- **Maximum activation duration:** 1–24 hours. After expiry, the activation ends and the user returns to eligible-only status.
- **Require MFA on activation:** User must complete MFA when activating, even if already MFA'd for the session.
- **Require justification:** User must enter a business reason.
- **Require ticket information:** User must enter a ticket number (integrates with ITSM).
- **Require approval:** Designated approvers must approve before activation takes effect.

⚠️ **Scenario:** A user's PIM activation request has been pending for 4 hours. The designated approver hasn't responded. What happens?
> The request **expires** based on the pending approval timeout (default: 24 hours). The activation is denied automatically. The user must resubmit.

#### Approval Expiry vs Activation Expiry
These are separate:
- **Pending approval timeout:** How long a request waits before auto-denying (configurable, default 24h).
- **Activation duration:** How long the role is active once approved.

#### Eligible vs Active Assignment — Exam Scenarios

| Scenario | Correct Assignment Type |
|---|---|
| Admin needs 24/7 permanent access to read audit logs | Active (permanent) |
| Admin occasionally needs Global Admin for break-glass | Eligible |
| Service account must always have Contributor on a subscription | Active (time-bound recommended) |
| User needs User Admin access for a 3-month project | Eligible (time-bound) |

#### PIM Alerts — Must Know
- **Roles are being activated too frequently:** Possible misuse or over-reliance on privilege.
- **Too many permanent active role assignments:** All privileged roles should be eligible, not permanently active.
- **Roles don't require MFA on activation:** Security gap.
- **There are too many Global Admins:** Microsoft recommends 2–4 maximum.

📖 https://learn.microsoft.com/entra/id-governance/privileged-identity-management/pim-configure

---

### 4.3 Entitlement Management — Edge Cases

#### Access Package Policy Types

| Policy Type | Who can request |
|---|---|
| For users in your directory | Members, guests already in tenant |
| For users not in your directory | External users via connected org |
| None (admin only) | Only admins can assign |

⚠️ **Scenario:** An external partner from a domain not registered as a connected organization requests access to a package scoped to connected orgs. What happens?
> They are blocked at the request stage. Only users from registered connected organizations can request. Either add their org as a connected org or create a separate policy scoped to "All users (including external)."

#### Separation of Duties in Entitlement Management
- You can configure **incompatible access packages** — if a user holds Package A, they cannot request Package B.
- Use case: enforce SoD between "Approve Purchase Orders" and "Create Purchase Orders" access packages.
- This is a **natively supported control** — no custom logic needed.

#### Access Package Lifecycle Settings
- **Expiration:** After N days, or on a specific date, or never.
- **Access reviews:** Can be triggered on a schedule for all active assignments.
- **Requestor justification:** Required business reason at request time.

📖 https://learn.microsoft.com/entra/id-governance/entitlement-management-access-packages

---

### 4.4 Access Reviews — Behavioral Nuances

#### Review Scope Options (Often Confused)

| Scope | What's reviewed |
|---|---|
| All members | All users in the group/app |
| Guest users only | Filter to only B2B guests |
| Inactive users | Users who haven't signed in for N days |

#### Decision Helpers
- **No action = approve (default):** If a reviewer doesn't respond, the system auto-approves. **This is a security risk and should be changed.**
- **No action = deny (recommended):** Inactive/unreviewed access is removed.
- **System recommendations:** Entra can recommend approve/deny based on sign-in activity. Reviewers can accept the recommendation with one click.

#### Upon Denial — What Happens to Users

| Object Type | Denial Action |
|---|---|
| Group member | Removed from group |
| App assignment | Removed from app |
| Guest user (if selected) | Block sign-in + delete after 30 days |
| Eligible PIM role | Role assignment removed |

⚠️ **Scenario:** An access review completes. 200 guests were reviewed. Reviewers denied 50. The security team checks the portal 7 days later and sees all 50 guest accounts are still active. Why?
> The access review was configured with **auto-apply results disabled**. Results must be manually applied, or auto-apply must be enabled in the review settings.

📖 https://learn.microsoft.com/entra/id-governance/access-reviews-overview

---

---

# MUST-KNOW TABLES — Exam Day Quick Reference

---

## Authentication Method Selection

| Scenario | Recommended Method |
|---|---|
| Highest security, phishing-resistant | FIDO2 security key |
| Windows device, no hardware key | Windows Hello for Business |
| Mobile-first users | Microsoft Authenticator (passwordless) |
| Bootstrap MFA for new hire | Temporary Access Pass (TAP) |
| Break-glass emergency admin | TAP + FIDO2 |

---

## CA Grant Controls — When Each Is Required

| Grant Control | Required When |
|---|---|
| Require MFA | Any sensitive app access |
| Require Authentication Strength | Need specific method (e.g., phishing-resistant only) |
| Require compliant device | Corporate data access, Zero Trust device signal |
| Require Hybrid Azure AD Joined | On-premises resource access from domain-joined devices |
| Require approved client app | Mobile app access (legacy control, being replaced) |
| Require app protection policy | MAM-enrolled devices without MDM enrollment |

---

## PIM Activation Decision Tree

```
User triggers activation
        ↓
Is MFA required on activation? → Complete MFA
        ↓
Is justification required? → Enter justification
        ↓
Is ticket number required? → Enter ticket
        ↓
Is approval required?
   Yes → Request sent to approver(s)
         ↓
         Approved within timeout? → Role activates for configured duration
         Denied / Timeout → Request expires, access not granted
   No  → Role activates immediately for configured duration
```

---

## Identity Protection Risk Policy Decision

| Situation | Correct Policy |
|---|---|
| Block all high user risk sign-ins | CA policy: user risk ≥ High → Block |
| Force password reset for high user risk | CA policy: user risk ≥ High → Require password change |
| Require MFA for medium sign-in risk | CA policy: sign-in risk ≥ Medium → Require MFA |
| Offline detection compromised user | Only caught by user risk policy (not sign-in risk) |

---

## App Permission Consent Reference

| Permission Type | Who Can Consent | Risk Level |
|---|---|---|
| Delegated (low sensitivity) | User (if user consent enabled) | Low |
| Delegated (high sensitivity, e.g., Mail.Read all) | Admin only | Medium |
| Application permission (any) | Admin only | High |
| Application permission + sensitive data | Admin + policy review recommended | Critical |

---

---

# TOP EXAM TRAPS — Extended List

Beyond what your cheat sheet covers:

1. **Authentication Strength ≠ MFA.** Requiring MFA allows any method. Requiring Phishing-Resistant MFA (Authentication Strength) blocks Authenticator push — only FIDO2, WHfB, or CBA qualify.

2. **CA Report-Only still creates sign-in logs.** Use it to measure impact before enabling. Do NOT skip this step in enterprise deployments.

3. **Managed identities cannot authenticate outside Azure.** They rely on the Azure Instance Metadata Service (IMDS). For non-Azure workloads, use workload identity federation.

4. **TAP is single-use by default.** Configurable to multi-use. Expires after the configured window (default 1 hour). After use, a permanent auth method must be registered.

5. **PIM for Azure roles ≠ PIM for Entra roles.** Different blades, different activation flows, different audit logs. Azure role activation goes through Azure RBAC; Entra role activation goes through the Entra directory.

6. **Access review auto-apply is not on by default.** Results sit pending until manually applied unless you explicitly enable auto-apply in review settings.

7. **Connected organization ≠ cross-tenant access policy.** Connected orgs are for Entitlement Management (access packages). Cross-tenant access policies govern B2B trust settings. They are separate configurations.

8. **Lifecycle Workflows do not replace provisioning.** Workflows trigger tasks (enable account, send email, generate TAP). They do not create AD accounts. Provisioning (Entra provisioning service + agent, or Connect Sync) creates the objects.

9. **CA does not apply to legacy authentication by default.** You must explicitly create a policy blocking legacy auth clients (Exchange ActiveSync, SMTP, IMAP). Without this, MFA policies can be bypassed.

10. **App Proxy Passthrough pre-auth = no CA enforcement.** If you set pre-auth to Passthrough on an App Proxy application, no Conditional Access policy will be evaluated for that app.

11. **Cloud Sync provisioning agent ≠ Connect provisioning agent.** Cloud Sync agent syncs AD to Entra. The Connect provisioning agent enables cloud-to-AD provisioning (e.g., Workday → AD). Different binaries, different purpose.

12. **Dynamic group rule errors stop all evaluation.** If a dynamic group rule contains a syntax error, the entire group stops updating — not just the broken rule. Monitor group processing errors in the portal.

---

---

# RECOMMENDED HANDS-ON LAB ADDITIONS

These supplement your existing lab checklist with scenarios specifically targeting gap areas:

| Lab | What It Tests |
|---|---|
| Configure Authentication Strength requiring FIDO2 only | Authentication Strengths vs MFA |
| Create CA policy with device filter excluding PAW devices | Device filter syntax |
| Configure CAE and test token revocation timing | CAE behavior |
| Dismiss vs remediate a risk in Identity Protection | Risk remediation decisions |
| Create Lifecycle Workflow with custom task extension (Logic App) | LCW custom extensions |
| Configure PIM with approval + ticket requirement, test expiry | PIM activation flow |
| Build access package with incompatible package (SoD) | Entitlement management SoD |
| Configure access review with auto-apply disabled, then re-enable | Access review auto-apply |
| Enable Workload Identity Federation for GitHub Actions | Workload identity federation |
| Configure RMAU, attempt modification as Global Admin | RMAU behavior |

---

---

# MICROSOFT LEARN LINKS FOR GAP AREAS

| Topic | Link |
|---|---|
| Authentication Strengths | https://learn.microsoft.com/entra/identity/authentication/concept-authentication-strengths |
| CA for Workload Identities | https://learn.microsoft.com/entra/identity/conditional-access/workload-identity |
| Device Filter Conditions | https://learn.microsoft.com/entra/identity/conditional-access/concept-condition-filters-for-devices |
| CAE Deep Dive | https://learn.microsoft.com/entra/identity/conditional-access/concept-continuous-access-evaluation |
| Identity Protection Risks | https://learn.microsoft.com/entra/id-protection/concept-identity-protection-risks |
| WHfB Cloud Trust | https://learn.microsoft.com/entra/identity/authentication/howto-windows-hello-for-business-deploy-cloud-trust |
| Workload Identity Federation | https://learn.microsoft.com/entra/workload-id/workload-identity-federation |
| Admin Consent Workflow | https://learn.microsoft.com/entra/identity-platform/howto-configure-admin-consent-workflow |
| App Proxy Connector Groups | https://learn.microsoft.com/entra/identity/app-proxy/application-proxy-connector-groups |
| Lifecycle Workflows Overview | https://learn.microsoft.com/entra/id-governance/workflows-overview |
| PIM Activation Settings | https://learn.microsoft.com/entra/id-governance/privileged-identity-management/pim-configure |
| Entitlement Mgmt SoD | https://learn.microsoft.com/entra/id-governance/entitlement-management-access-package-incompatible |
| Access Review Auto-Apply | https://learn.microsoft.com/entra/id-governance/access-reviews-overview |
| RMAU | https://learn.microsoft.com/entra/identity/role-based-access-control/admin-units-restricted-management |
| Duplicate Attribute Resiliency | https://learn.microsoft.com/entra/identity/hybrid/connect/how-to-connect-syncservice-duplicate-attribute-resiliency |

---

*Last updated for SC-300 exam objectives as of 2025–2026. Cross-reference with the official study guide at:*
*https://learn.microsoft.com/credentials/certifications/resources/study-guides/sc-300*
