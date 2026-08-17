# Azure Public IP: Basic → Standard SKU Upgrade

## Context

Azure Basic SKU Public IP addresses were retired on 2025-09-30. A recurring migration concern is whether upgrading a Basic SKU Public IP to Standard changes the public IPv4 address and what operational risks exist during the transition.

## Status

- **[Confirmed]** Microsoft documents that a supported Public IP resource upgrade retains the IP address.
- **[Confirmed]** The Basic Public IP must use **Static** allocation before the SKU upgrade.
- **[Confirmed]** The Public IP must be disassociated from its attached resource for the documented upgrade flow.
- **[Confirmed]** Basic → Standard SKU upgrade is not reversible.
- **[Confirmed]** Standard SKU uses a secure-by-default inbound model; NSG/routing configuration must be validated to avoid an unexpected outage.
- **[Needs Review]** Microsoft Learn currently contains inconsistent wording about whether upgraded Basic Public IPs become zone-redundant or remain non-zonal. Do not rely on a blanket assumption; verify the target resource and region.

## Architecture Decision

### Context

The goal is to migrate an existing Basic SKU Public IP while preserving an externally depended-on IP address where the resource type supports the documented in-place SKU upgrade path.

### Constraints

- Existing clients, DNS allowlists, partner firewalls, or SaaS integrations may depend on the current IP.
- The migration can require temporary disassociation and therefore planned downtime.
- Standard SKU does not support dynamic allocation.
- Security semantics differ between Basic and Standard.
- Some Azure resource types require a resource-specific migration rather than a generic Public IP SKU update.

### Options

1. **Supported in-place Public IP SKU upgrade**
   - Convert Basic allocation to Static if required.
   - Disassociate the Public IP.
   - Upgrade SKU to Standard.
   - Reassociate and validate networking/security.

2. **Create a new Standard Public IP**
   - Appropriate when the resource-specific migration path requires replacement, or when new zonal/zone-redundant characteristics are required.
   - Changes the IP and therefore requires dependency migration.

### Decision

Prefer the documented in-place upgrade when preserving the public IP is a hard requirement **and** the attached Azure resource supports that path. Use a replacement Standard Public IP when the resource-specific migration guidance requires replacement or when desired availability-zone characteristics cannot be achieved by the existing IP.

### Rationale

The generic Public IP upgrade documentation explicitly states that upgrading the Public IP resource retains the IP address. This avoids unnecessary downstream DNS/allowlist changes, but it does not remove the need for downtime planning and security validation.

### Trade-offs

**Benefits**
- Preserves external dependencies on the existing IP.
- Avoids broad DNS/firewall/partner allowlist migration.

**Costs / Risks**
- Temporary network interruption while disassociating/reassociating.
- Standard SKU security defaults may block traffic if NSGs are not prepared.
- Upgrade is irreversible.
- Resource-specific constraints can override the generic Public IP procedure.

### Failure Modes

- Public IP is still dynamically allocated and cannot be upgraded.
- Public IP remains associated with a resource when attempting the generic upgrade.
- NSG rules are missing after moving to Standard SKU.
- The attached service requires its own migration procedure (for example, a Basic Load Balancer or another legacy SKU dependency).
- Availability-zone assumptions are incorrect because documentation/resource behavior differs.

### Revisit Conditions

Re-evaluate the migration plan if:

- the IP is attached to a service with a dedicated SKU migration procedure;
- zone-redundancy is a requirement;
- zero downtime is required;
- Microsoft changes the Basic Public IP post-retirement migration guidance.

## Verification Checklist

1. Inventory the Public IP resource and its attached Azure resource.
2. Confirm allocation is Static.
3. Read the **resource-specific** migration guidance, not only the generic Public IP page.
4. Record the current IP address before change.
5. Validate NSG/routing requirements for Standard SKU.
6. Schedule the required interruption.
7. Perform the supported upgrade or replacement path.
8. Verify SKU, IP address, reachability, NSG behavior, health probes, DNS, and external allowlists.
9. Confirm actual availability-zone properties/behavior rather than relying on assumptions.

## Sources

- Microsoft Learn — Upgrade a public IP address: https://learn.microsoft.com/en-us/azure/virtual-network/ip-services/public-ip-upgrade
- Microsoft Learn — Upgrade Basic Public IP Address to Standard SKU in Azure: https://learn.microsoft.com/en-us/azure/virtual-network/ip-services/public-ip-basic-upgrade-guidance
- Microsoft Learn — Public IP addresses in Azure: https://learn.microsoft.com/en-us/azure/virtual-network/ip-services/public-ip-addresses

_Last verified: 2026-08-18._
