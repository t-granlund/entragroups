# EntraGroups

> **Groups that govern. Access that scales.** An automated group-lifecycle engine
> for Microsoft Entra ID — from smart creation to persona-based RBAC. Stop
> managing groups in spreadsheets.

[![License: MIT](https://img.shields.io/badge/License-MIT-success.svg)](LICENSE)
[![Part of TenantFleet](https://img.shields.io/badge/TenantFleet-Body-2e7d32)](https://t-granlund.github.io/tenantfleet/)
[![Live demo](https://img.shields.io/badge/demo-live-1f6feb)](https://t-granlund.github.io/entragroups/)
[![Secrets](https://img.shields.io/badge/hardcoded%20secrets-0-success)](#security)

EntraGroups is the **Body** pillar of the
[TenantFleet](https://t-granlund.github.io/tenantfleet/) multi-tenant Microsoft
Entra ID governance ecosystem. It turns group management — usually a manual,
spreadsheet-driven chore — into a rule-driven engine that keeps membership and
access correct as people change roles, locations, and brands.

---

## Why it exists

Multi-brand portfolios, MSPs, and PE-backed groups all hit the same wall:
security-group sprawl. Groups get created by hand, membership drifts, and RBAC
becomes a guessing game. EntraGroups replaces that with **attribute- and
persona-driven groups** so access is a function of *who someone is*, not who
remembered to add them.

## What it delivers

| Capability | What it does |
| --- | --- |
| **Smart group creation** | Rule-driven groups that self-maintain membership based on user attributes and lifecycle state. |
| **Persona-based RBAC** | Map personas to security groups, application roles, and SharePoint permissions without manual assignment. |
| **Dynamic membership** | Attribute-based rules keep membership current as users change roles, locations, or brands. |
| **Audit logging** | Every change tracked with before/after state, actor identity, and Graph API correlation ID. |
| **TenantFleet integration** | Native hooks into the TenantFleet topology engine for brand-aware group scoping. |
| **SharePoint sync** | Synchronize Entra group membership to SharePoint permission levels automatically. |

**At a glance:** 4 personas supported · 0 hardcoded secrets · ∞ groups managed ·
&lt; 30s time to provision.

## Install

```bash
git clone https://github.com/t-granlund/entragroups.git
cd entragroups
```

EntraGroups authenticates to Microsoft Graph using **OIDC workload identity
federation** — there are no client secrets to store. Provide your tenant via
environment variables:

```bash
export TENANT_ID="<your-entra-tenant-id>"
# Auth is via federated OIDC credentials (GitHub Actions → Azure), not secrets.
```

## Usage

The `PersonaRBAC` middleware provisions and reconciles groups on user
lifecycle events:

```ts
import { PersonaRBAC } from "@tenantfleet/entra-groups";

const rbac = new PersonaRBAC({
  tenantId: process.env.TENANT_ID,
  personas: ["msp-core", "pe-portfolio", "franchisee"],
  fallbackGroup: "all-users",
});

// Auto-provision groups on user lifecycle events
await rbac.syncGroupsForUser(userId, persona);
```

A user is mapped to a persona; the persona resolves to a set of security
groups, app roles, and SharePoint permission levels; EntraGroups makes reality
match the rule.

## Architecture

```
attributes (Entra) ──► persona resolver ──► desired group set
                                              │
                          current group set ◄─┤ reconcile
                                              ▼
              Microsoft Graph (groups, members, app roles)
                                              │
                                              ▼
                    audit log (before/after + correlation ID)
```

- **Persona resolver** — maps user attributes (brand, location, role) to a
  persona and the access bundle it implies.
- **Reconciler** — diffs desired vs. current group membership and applies the
  minimal set of Graph changes.
- **Graph adapter** — batched, retry-aware Microsoft Graph calls for groups,
  members, and app-role assignments.
- **Audit layer** — records every mutation with before/after state, actor, and
  the Graph correlation ID for traceability.

## Security

- **Zero hardcoded secrets** — federated OIDC only.
- **Least privilege** — Graph permissions scoped to group lifecycle operations.
- **Full audit trail** — every change is attributable and reversible.

## Part of the TenantFleet ecosystem

EntraGroups is one of seven open-source repositories:

| Repo | Pillar | Focus |
| --- | --- | --- |
| [TenantFleet](https://t-granlund.github.io/tenantfleet/) | — | Root governance framework |
| [HubForge](https://t-granlund.github.io/hubforge/) | Mind | Azure SWA + Entra deployment templates |
| **EntraGroups** | Body | Group lifecycle + persona RBAC |
| [TenantForge](https://t-granlund.github.io/tenantforge/) | Mind | Terraform tenant provisioning |
| [DNSGuard](https://t-granlund.github.io/dnsguard/) | Spirit | Domain + DMARC intelligence |
| [RampGuard](https://t-granlund.github.io/rampguard/) | Mind | Finance compliance + spend |
| [SharePointAgent](https://t-granlund.github.io/sharepoint-agent/) | Body | SharePoint indexing + Teams |

See the full ecosystem on the portfolio:
[tylergranlund.com/work#ecosystem](https://tylergranlund.com/work#ecosystem).

## License

MIT — see [LICENSE](LICENSE). Fork it, deploy it, make it yours.
