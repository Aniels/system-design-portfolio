# Vendor Exitability Engineering

## ETO, Exit Readiness, Data Portability, and Control-Plane Independence

> **Reviewed:** 2026-09-15  
> **Status:** Working architecture framework  
> **Related background:** [Digital Sovereignty and Vendor Exit Strategy](digital-sovereignty-and-vendor-exit-strategy.md)

## Abstract

Vendor exitability is often discussed as if it were a variant of cross-cloud disaster recovery: if the primary cloud becomes unavailable, restore the workload somewhere else. That model is incomplete.

A stressed vendor exit may be triggered by provider access loss, contractual termination, licensing shock, legal or policy restrictions, product retirement, or an unacceptable jurisdictional dependency. In these scenarios, the organisation may have very little time to act and may not be able to rely on the primary provider's control plane during the migration.

This note develops a practical engineering framework for such scenarios. It proposes three working metrics:

- **Exit Readiness Objective (ERO):** the measurable readiness that must exist *before* an exit trigger occurs.
- **Exit Time Objective — Activation (ETO-A):** the maximum acceptable time from exit trigger to restoration of the minimum viable business capability.
- **Exit Time Objective — Completion (ETO-C):** the maximum acceptable time to remove the critical dependency on the original provider.

These terms are **proposed working metrics**, not established industry standards. The reviewed regulatory guidance uses concepts such as exit strategy, transition period, portability, technical analysis, estimated transition time, and regular testing rather than the ERO / ETO-A / ETO-C names.

The central engineering conclusion is:

> **A short stressed-exit time is primarily a pre-positioning problem, not a post-incident migration problem.**

## 1. Problem Statement

Suppose a critical system runs almost entirely on one cloud provider:

```text
Users
  |
  v
Provider DNS
  |
  v
Provider Load Balancer
  |
  v
Application
  |
  v
Managed Database
```

The common assumption is:

> If the provider becomes unacceptable or unavailable, migrate the system to another cloud.

The problem is that this assumption ignores two hard constraints:

1. **Data cannot necessarily be exported fast enough after the trigger.**
2. **The organisation may lose access to the control planes needed to perform the migration.**

A vendor exit therefore needs a different architecture question:

> What assets and control must already exist outside the provider's failure domain so that the business can continue if access is lost abruptly?

## 2. Exitability Is Broader Than Cross-Cloud DR

Cross-cloud DR and vendor exitability overlap, but the failure models differ.

| Concern | Typical trigger | Primary objective | Expected return to original provider? |
|---|---|---|---|
| Regional DR | Region outage | Restore workload | Usually yes |
| Cross-cloud DR | Provider-wide technical outage | Restore workload elsewhere | Often yes |
| Business continuity | Severe disruption | Preserve minimum business process | Not necessarily |
| Vendor exit | Contract, legal, policy, product, jurisdiction, strategic dependency | Permanently reduce or remove dependency | Usually no |
| Digital sovereignty | Long-term control and dependency risk | Preserve strategic control and optionality | Not applicable |

**[Inference]** Vendor exitability is best understood as a **strategic resilience** requirement. It may reuse DR mechanisms, but it also covers failure modes that traditional availability engineering does not normally model.

## 3. Existing Guidance: The Requirement Exists Even If "ETO" Does Not

The reviewed sources do not define a standard metric named `Exit Time Objective`.

However, the underlying requirement already appears in European financial regulation and supervisory guidance.

**[Fact]** DORA requires exit strategies for ICT third-party services supporting critical or important functions and requires an adequate transition period that allows the financial entity to migrate to another provider or move the service in-house [1].

**[Fact]** The ECB's 2025 cloud outsourcing guide recommends that organisations perform technical analysis and **estimate the time required for transition**. It also recommends that exit plans include critical milestones, required tasks and skills, rough estimates of time and cost, regular review and testing, and explicit consideration of data volume, application complexity, transfer method, and staffing [2].

**[Inference]** ETO can therefore be useful as an architecture metric, provided it is clearly labelled as an internal engineering construct rather than a regulatory term.

## 4. Proposed Metrics

### 4.1 Exit Readiness Objective (ERO)

**[Proposal]** `ERO` measures whether the exit path is sufficiently prepared *before* the exit trigger.

ERO should not be represented by a single number. It is better treated as a set of measurable readiness constraints.

Example:

```text
ERO for Tier-0 service

External replica lag       <= 5 minutes
Independent backup age     <= 1 hour
IaC coverage               = 100% of critical components
DNS control                outside primary provider
Registrar access           independently recoverable
Break-glass credentials    tested every quarter
Last restore exercise      <= 90 days
Last exit rehearsal        <= 12 months
```

ERO answers:

> If access disappears now, what do we already have?

### 4.2 ETO-A — Exit Time Objective, Activation

**[Proposal]** `ETO-A` is the maximum acceptable time between the exit trigger and restoration of the **minimum viable business capability**.

Example:

```text
Exit trigger
    |
    | 4 hours
    v
Minimum critical service restored
```

The target is not necessarily full production parity. It is the smallest service that prevents unacceptable business harm.

### 4.3 ETO-C — Exit Time Objective, Completion

**[Proposal]** `ETO-C` is the maximum acceptable time required to eliminate the critical dependency on the original provider.

Example:

```text
Day 0        Day 1          Day 30          Day 90
 |             |               |               |
Trigger      Minimum        Major workloads   Critical provider
             service        migrated          dependency removed
             restored
```

ETO-C is therefore closer to a strategic migration objective than an ordinary RTO.

### 4.4 Exit RPO

A short ETO without an acceptable data-loss objective can be misleading.

For example:

```text
ETO-A = 4 hours
Backup interval = 7 days
```

The application might start within four hours but still lose up to a week of transactions.

**[Proposal]** For stressed-exit scenarios, architecture reviews should state an **Exit RPO** together with ETO-A.

```text
ETO-A <= 4 hours
Exit RPO <= 5 minutes
```

These targets immediately imply much stronger steady-state readiness.

## 5. Why "Migrate After the Trigger" Often Fails

### 5.1 The bandwidth limit is physical

A simple lower-bound estimate is:

\[
T = \frac{\text{data size in bits}}{\text{effective throughput}}
\]

For a 10 TB database over a sustained 1 Gbit/s link:

\[
10 \times 10^{12} \times 8 / 10^{9}
\approx 80{,}000\text{ seconds}
\approx 22.2\text{ hours}
\]

That is only the theoretical transfer time. It excludes:

- snapshot or dump creation;
- provider throttling;
- encryption and protocol overhead;
- network contention;
- import time;
- index rebuilds;
- schema conversion;
- consistency validation;
- application testing;
- cutover coordination.

**[Inference]** If the requirement is "provider access may disappear in 24 hours", then a 10 TB database that exists only inside the provider is already close to an impossible stressed-exit target at 1 Gbit/s.

For 100 TB, the same theoretical transfer would take roughly nine days.

### 5.2 Provider access may disappear before export finishes

A stressed exit is not always a graceful migration.

Possible triggers include:

- account suspension;
- legal prohibition;
- contract termination;
- geopolitical restriction;
- provider control-plane outage;
- loss of approved operational support;
- loss of privileged access.

If the only authoritative backup, export API, encryption key, DNS zone, or identity administrator is still inside that provider, the exit plan can fail before data transfer even begins.

## 6. Architecture Principle: Pre-position Critical Assets

A short ETO requires assets to exist outside the primary failure domain before the incident.

```text
                    Normal operation
                           |
                           v
                    Primary provider
                           |
          +----------------+----------------+
          |                |                |
          v                v                v
   External backup   Delta replication   Dependency inventory
          |                |                |
          +----------------+----------------+
                           |
                           v
                  Exit-ready environment
```

The objective is not necessarily a fully active second production stack.

It is to maintain enough independent state and control to rebuild or activate the critical business process within the stated objective.

## 7. Data Pattern: Bulk Seed + Delta Replication

Replicating the full dataset repeatedly is usually wasteful.

A more efficient pattern is:

```text
Initial phase

Primary database
      |
      | large snapshot / physical export
      v
Bulk seed
      |
      v
Alternative environment

Steady state

Primary database
      |
      | WAL / CDC / logical replication / incremental backup
      v
Alternative environment
```

The first copy establishes the baseline; steady-state traffic contains only changes.

PostgreSQL logical replication follows this general pattern: it begins with an initial snapshot and then continuously transfers subsequent changes to the subscriber [3].

### 7.1 Why this matters for cost

If the database is 100 TB but only 200 GB changes per day, repeatedly copying 100 TB is economically irrational.

The desired cost profile is closer to:

```text
Initial cost:
100 TB once

Steady-state cost:
~200 GB/day + storage + replication overhead
```

rather than:

```text
100 TB/day
```

### 7.2 Physical transfer appliances as a seeding mechanism

Offline appliances can be useful when the initial dataset is too large for the available network window.

**[Fact]** Azure Data Box currently supports import and export with next-generation devices providing up to 120 TB or 525 TB usable capacity [4]. Microsoft notes that export orders may still incur additional egress charges, so physical transport should not be assumed to eliminate provider data-out charges [5].

**[Fact]** Google Cloud Transfer Appliance currently offers 40 TB and 300 TB appliances for data ingestion and export [6].

**[Fact]** AWS has announced that Snowball device support in commercial AWS Regions will end on 31 December 2026 [7]. This is a useful reminder that the exit mechanism itself can become a lifecycle dependency.

**[Recommendation]** Treat physical appliances as **bulk-seeding tools**, not as the complete exit strategy. They solve the baseline-transfer problem but not the ongoing data freshness problem.

## 8. Cost Model for Exit Readiness

A more useful cost model is:

```text
Exit Readiness Cost
=
Initial Seeding Cost
+ Ongoing Delta Transfer Cost
+ Alternative Storage Cost
+ Standby Compute / Platform Cost
+ Periodic Exercise Cost
+ Operational Skill Cost
```

This makes an important trade-off visible:

> Lower ETO normally requires higher recurring readiness cost.

### 8.1 Illustrative egress calculation

Using Azure's published Internet egress rates for Asia as of the review date, an illustrative 100 TB Internet egress transfer is roughly USD 8.7k before storage, appliance, shipping, engineering, or destination-cloud costs [8].

This number is only an example. Pricing varies by region, routing model, negotiated agreement, service, and time.

The architecture question is not whether exit readiness is free. It is:

> How much recurring optionality cost is justified by the business impact of being unable to exit?

## 9. DNS and Domain Control Are Separate Failure Domains

A common mistake is to treat "DNS" as part of the application deployment.

There are several distinct control layers:

```text
Registrant
   |
Registrar
   |
Registry / parent delegation
   |
Authoritative DNS provider
   |
Application endpoints
```

ICANN distinguishes the registrar's role in registering domain names from the registry's responsibilities and the delegation of authority to authoritative name servers [9][10].

This matters because an application can be perfectly restored on another cloud and still remain unreachable if the organisation cannot change DNS.

### 9.1 Bad coupling

```text
Primary cloud account
   |
   +-- application
   +-- authoritative DNS
   +-- identity administrators
   +-- secrets
   +-- certificates
   +-- domain/registrar recovery path
```

If access to the account is lost, all recovery controls may disappear together.

### 9.2 Better control-plane separation

```text
Independent registrar
        |
        v
Independent or multi-provider DNS
        |
        +------------------+
        |                  |
        v                  v
 Primary cloud       Alternative environment
```

**[Recommendation]** For Tier-0 workloads, domain registration, authoritative DNS control, break-glass identity, key recovery material, and exit runbooks should not all share the same provider or administrative failure domain as the application.

## 10. DNS TTL Is Only One Part of Cutover

RFC 1034 defines TTL as the maximum time a DNS resource record may remain cached and notes that reducing TTL before an anticipated change can reduce inconsistency during cutover [11].

Therefore:

```text
TTL = 86400 seconds
```

can leave cached answers in circulation for up to roughly a day.

A stressed-exit architecture may choose shorter TTLs for critical endpoints, for example:

```text
TTL = 60–300 seconds
```

but this has trade-offs:

- higher authoritative-query volume;
- increased dependence on the DNS provider;
- resolver behaviour may not be perfectly uniform;
- low TTL does not solve loss of registrar or delegation control.

**[Inference]** DNS cutover readiness is therefore a **control-plane design problem**, not merely a TTL tuning problem.

## 11. Identity, Keys, and Secrets Can Be Harder Than Compute

Moving compute is often the easiest part.

The harder dependencies may include:

- workforce identity;
- customer identity;
- service principals;
- RBAC policy;
- KMS/HSM keys;
- certificates;
- secrets;
- CI/CD credentials;
- endpoint management;
- security policy;
- observability;
- audit history;
- SaaS-specific integrations.

A system whose database is portable but whose authentication path depends entirely on the failed provider may still be unusable.

**[Recommendation]** Exit dependency inventories should explicitly separate:

```text
Data plane
Control plane
Identity plane
Security plane
Operational plane
```

## 12. A Tiered Exitability Model

Not every workload deserves the same exit architecture.

| Tier | Business impact | Suggested readiness | Typical pattern |
|---|---|---|---|
| Tier 0 | Critical function materially impaired | Continuous or frequent external state, independent control planes, rehearsed activation | Warm/hot minimum-capability fallback |
| Tier 1 | Important but recoverable | Bulk seed, regular delta sync, validated restore, IaC, tested DNS/identity plan | Cold/warm migration-ready environment |
| Tier 2 | Commodity / replaceable | Portable export, dependency inventory, documented rebuild process | Planned migration |
| Tier 3 | Low criticality | Contractual portability and data export may be sufficient | Rebuild when needed |

The correct design depends on business impact, not ideology.

## 13. Example: 100 TB PostgreSQL Workload

### Requirements

```text
Dataset                    100 TB
Daily change rate          200 GB
Exit RPO                   <= 15 minutes
ETO-A                      <= 8 hours
ETO-C                      <= 30 days
Primary provider           Cloud A
Target                     Cloud B or on-prem
```

### Naive design

```text
Cloud A PostgreSQL
      |
      X  provider access lost
      |
      v
Start exporting 100 TB
```

This cannot credibly satisfy an eight-hour ETO-A.

### Exit-ready design

```text
                         Independent registrar
                                |
                         Independent DNS
                                |
               +----------------+----------------+
               |                                 |
               v                                 v
        Primary Cloud A                   Exit Environment
               |                                 |
        PostgreSQL primary                PostgreSQL standby
               |                                 ^
               | initial bulk seed               |
               +---------------------------------+
               |                                 ^
               | WAL / logical replication       |
               +---------------------------------+
```

Additional requirements:

```text
Independent backup        <= 1 hour old
Replication lag           <= 15 minutes
IaC                        complete for Tier-0 path
Secrets/key recovery      provider-independent
DNS cutover               rehearsed
Restore validation        quarterly
Exit rehearsal            annually
```

The expensive 100 TB transfer happens primarily during initial seeding. Normal steady-state transfer follows the change rate.

## 14. Failure Modes the Exit Plan Must Test

A credible plan should test more than "region down".

### Provider and account failures

- provider API unavailable;
- tenant/subscription/account suspended;
- privileged administrator unavailable;
- billing or contract access terminated.

### Data failures

- replica lag exceeds objective;
- replication slot breaks;
- backup cannot restore;
- schema or extension is incompatible;
- encryption key cannot be recovered.

### Control-plane failures

- DNS provider unavailable;
- registrar credentials unavailable;
- certificate issuance path depends on failed provider;
- secret store cannot be accessed;
- alternative IAM is not ready.

### Operational failures

- staff do not know the runbook;
- destination quotas are insufficient;
- destination environment has never handled production scale;
- observability is missing during cutover;
- compliance approval blocks activation.

### Cost failures

- egress cost is materially above assumptions;
- standby environment cost makes readiness unsustainable;
- migration requires scarce vendor-specific specialists.

## 15. Exit Exercise Levels

An untested export button is not an exit strategy.

A practical maturity model is:

```text
Level 0 — Documentation only
Level 1 — Tabletop exercise
Level 2 — Restore representative data
Level 3 — Partial service activation
Level 4 — Production-scale migration rehearsal
Level 5 — Controlled live exit / failover exercise
```

The required level should be proportional to criticality.

## 16. Architecture Review Checklist

For a critical provider dependency, ask:

1. What business capability must survive?
2. What event triggers a stressed exit?
3. What is the minimum viable business process?
4. What are ETO-A, ETO-C, and Exit RPO?
5. What ERO conditions must remain true during normal operation?
6. Which authoritative data exists outside the provider today?
7. How is the initial bulk dataset seeded?
8. How are incremental changes replicated?
9. What is the expected steady-state transfer cost?
10. What are the provider-independent backups?
11. Who controls the domain registrar?
12. Who controls authoritative DNS?
13. Are DNS TTL and cutover procedures compatible with ETO-A?
14. Can identity operate without the primary provider?
15. Can keys, certificates, and secrets be recovered independently?
16. Is the replacement environment defined, or is the plan merely "move somewhere else"?
17. Are quotas, capacity, security controls, and observability pre-created or reproducible?
18. When was the last restore test?
19. When was the last exit rehearsal?
20. Which dependency is most likely to make the stated ETO impossible?

## 17. Trade-offs

### Short ETO

Benefits:

- lower business interruption;
- stronger sovereignty and provider resilience;
- more credible response to abrupt provider loss.

Costs:

- replication egress;
- standby storage/compute;
- duplicated operational knowledge;
- additional security surface;
- more testing and governance.

### Long ETO

Benefits:

- lower recurring cost;
- lower operational complexity;
- greater ability to use provider-native features aggressively.

Costs:

- greater exposure to forced or stressed exit;
- larger migration critical path;
- higher switching risk;
- greater dependence on provider cooperation.

There is no universally correct target.

## 18. Proposed Architecture Principle

**[Recommendation]**

> **A short vendor-exit objective must be backed by pre-positioned state and independent control. Do not claim an ETO shorter than the time required to recreate any asset that still exists only inside the provider failure domain.**

A related design rule is:

> **Separate the business's recovery control plane from the provider that may need to be exited.**

This includes, where justified by criticality:

- domain and DNS control;
- break-glass identity;
- data backups and replicas;
- key recovery;
- IaC and deployment artefacts;
- observability and runbooks.

## 19. Current Conclusion

Vendor exitability should not default to permanent active-active multi-cloud.

For many systems, the more defensible architecture is:

```text
Primary provider
      |
      +-- bulk seed once
      +-- delta replication continuously
      +-- independent backup
      +-- independent DNS / registrar control
      +-- provider-independent recovery credentials
      +-- IaC + tested runbook
      |
      v
Migration-ready or minimum-capability alternative
```

This preserves optionality without paying the full complexity cost of running two identical production systems.

The decisive insight is that **exitability is a steady-state property**. If the organisation waits until the exit trigger to discover whether data, DNS, identity, keys, and operational knowledge are portable, the stated ETO is not an objective; it is a hope.

## 20. Unknowns and Further Research

- How should ERO be expressed as a quantitative score without hiding critical single points of dependency?
- Should ETO-A and ETO-C become separate governance requirements for Tier-0 systems?
- How should organisations price the annual cost of strategic optionality?
- What percentage of production data must be continuously replicated versus recoverable from cold backup?
- How should identity portability be tested across Entra ID, Okta, cloud IAM, and self-hosted identity systems?
- Can architecture policy automatically reject a design whose registrar, DNS, keys, backup, and workload all share one provider failure domain?
- How should stressed-exit exercises handle contractual or legal scenarios where provider cooperation is intentionally assumed unavailable?

## References

1. European Union, **Regulation (EU) 2022/2554 — Digital Operational Resilience Act (DORA)**. https://eur-lex.europa.eu/eli/reg/2022/2554/oj
2. European Central Bank, **Guide on outsourcing cloud services to cloud service providers**, July 2025. https://www.bankingsupervision.europa.eu/ecb/pub/pdf/ssm.supervisory_guides202507.en.pdf
3. PostgreSQL Global Development Group, **Logical Replication — Architecture**. https://www.postgresql.org/docs/current/logical-replication.html
4. Microsoft Learn, **What is Azure Data Box?** https://learn.microsoft.com/en-us/azure/databox/data-box-overview
5. Microsoft Learn, **Azure Data Box FAQ**. https://learn.microsoft.com/en-us/azure/databox/data-box-faq
6. Google Cloud, **Transfer Appliance specifications and pricing**. https://cloud.google.com/transfer-appliance/pricing
7. Amazon Web Services, **AWS Snowball — End of support notice**. https://aws.amazon.com/snowball/
8. Microsoft Azure, **Bandwidth pricing**. https://azure.microsoft.com/en-us/pricing/details/bandwidth/
9. ICANN, **The Domain Name Registration Process**. https://www.icann.org/resources/pages/domain-name-registration-process-2023-11-02-en
10. ICANN, **Delegation — Acronyms and Terms**. https://www.icann.org/en/icann-acronyms-and-terms/delegation-en
11. RFC Editor, **RFC 1034 — Domain Names: Concepts and Facilities**. https://www.rfc-editor.org/rfc/rfc1034.html
