# Digital Sovereignty and Vendor Exit Strategy

## A Background Review of European Public-Sector Technology De-Risking

> **Reviewed:** 2026-09-14  
> **Scope:** European public-sector digital infrastructure, workplace software, cloud services, and architecture implications.  
> **Note:** Switzerland is not an EU Member State; it is included as a European public-sector comparator.

## Abstract

Recent European public-sector initiatives are often described in the media as a movement to "de-Microsoft" government IT. Primary-source evidence supports a narrower and more useful interpretation: European institutions and governments are increasingly treating **technological sovereignty, supplier dependency, jurisdiction, interoperability, and exitability as architecture and procurement requirements**.

The trend is not a uniform rejection of Microsoft, US hyperscalers, or proprietary software. The European Commission has formalised technological sovereignty in 2026 policy, cloud procurement, and open-source strategy; Schleswig-Holstein has materially replaced Microsoft workplace components; Denmark is testing open-source office alternatives; France has announced a cross-government reduction of extra-European digital dependencies; and Switzerland is building an open-source workplace capability that will initially run **in parallel with Microsoft 365**.

The counter-evidence is equally important. The European Data Protection Supervisor confirmed in July 2025 that the European Commission had remedied the Microsoft 365 infringements identified in its 2024 decision. Microsoft has also introduced the EU Data Boundary and additional sovereign-cloud controls. Therefore, the architecture lesson is not "open source is always better" or "Microsoft is unusable in Europe." The stronger conclusion is that **critical public functions should treat vendor exitability as a first-class non-functional requirement and validate it proportionately to business criticality**.

## 1. Research Question and Scope

This review asks:

> Why are European public-sector organisations increasingly investing in alternatives to Microsoft and other non-European technology providers, and what architecture principles can be generalised from those initiatives?

The review prioritises primary sources: European Commission policy and procurement material, EDPS decisions, national and regional government announcements, US Department of Justice material on the CLOUD Act, and Microsoft product documentation. Technology media is used only as a discovery or framing source rather than as the primary evidence for architecture conclusions.

This is an architecture review, not legal advice. Legal applicability depends on the organisation, workload, contract, jurisdiction, data category, and current law.

## 2. From Data Residency to Technological Sovereignty

The European Commission defines technological sovereignty as Europe's ability to act independently in the digital domain by developing and controlling key technologies, data, and infrastructure while reducing reliance on non-EU providers [1]. In June 2026, the Commission introduced a broader Technological Sovereignty Package spanning semiconductors, cloud, AI, open source, and energy-system digitalisation [2].

The 2026 EU Open Source Strategy explicitly links open source with greater control, lower lock-in, and reduced dependency on non-EU technologies [3]. At the cloud layer, the Commission's Cloud Sovereignty Framework goes further by evaluating sovereignty through 48 criteria grouped into eight areas: strategic, legal and jurisdictional, data and AI, operational, supply chain, technological, security and compliance, and environmental sustainability [4].

This distinction matters architecturally. **Data residency is only one dimension of sovereignty.** A workload can store data in an EU region and still depend on a foreign-controlled provider, proprietary identity plane, closed document format, global support process, external software supply chain, or a legal regime outside the customer's jurisdiction.

A useful working model is:

| Dimension | Architecture question | Typical evidence |
|---|---|---|
| Data sovereignty | Where can data, backups, logs, and keys be stored or processed? | Data-flow map, residency commitments, key ownership |
| Operational sovereignty | Who can administer the service and under whose oversight? | Privileged-access model, support process, audit logs |
| Technological sovereignty | Can the organisation operate, modify, replace, or migrate the technology? | Open standards, APIs, export formats, source/code rights where relevant |
| Jurisdictional sovereignty | Which external legal regimes may affect the provider or data? | Provider structure, contracts, legal assessment |
| Supply-chain sovereignty | Which upstream vendors, components, and update channels are critical? | SBOM, dependency map, patch/update authority |
| Economic sovereignty | Can pricing or licensing changes create an unacceptable dependency? | Switching cost, migration cost, contract and licensing model |

**[Inference]** The policy shift is therefore better described as **dependency de-risking** than as a simple migration from one vendor to another.

## 3. Evidence Review

### 3.1 European Union: sovereignty becomes measurable procurement policy

**[Fact]** In June 2026, the European Commission presented the European Technological Sovereignty Package. The package includes the proposed Cloud and AI Development Act (CADA), Chips Act 2.0, EU Open Source Strategy, and a roadmap for digitalisation and AI in energy [2]. The CADA proposal includes a common EU-wide sovereignty assessment framework for cloud and AI and a public-sector adoption mechanism [5].

**[Fact]** In April 2026, the Commission awarded up to EUR 180 million of sovereign-cloud contracts to four provider groups. The Commission explicitly described provider diversification as a way to improve resilience and avoid lock-in to a single supplier. The associated Cloud Sovereignty Framework translates sovereignty into measurable assurance levels and 48 criteria [4][6].

This is significant because sovereignty is no longer only political language. It is being translated into **architecture-relevant procurement criteria**.

### 3.2 Switzerland: a contingency and exit path, not an immediate Microsoft replacement

Switzerland is not an EU Member State, but its federal programme illustrates the same architecture concern.

**[Fact]** The Swiss Federal Chancellery's BOSS proof of concept evaluates an independent open-source office environment for critical processes and explicitly examines the feasibility of a Microsoft 365 exit strategy [7]. In September 2026, the Federal Chancellery announced a programme intended to make sovereign workplace software available to about 3,000 employees from the end of 2027 [8].

The important detail is frequently lost in headlines: the sovereign workplace is initially intended to run **in parallel with Microsoft 365**. The feasibility study also found that the tested browser-based solution could contribute materially to an emergency solution in a Microsoft 365 outage, while noting limitations and significant migration effort [8].

**[Inference]** This is closer to a **business-continuity and strategic-options architecture** than to a blanket removal of Microsoft.

### 3.3 Schleswig-Holstein: evidence of both migration feasibility and lock-in cost

**[Fact]** Schleswig-Holstein reported in December 2025 that LibreOffice had become the mandatory office standard across the state administration and was used on nearly 80% of workplaces outside the tax administration. It also reported the migration of almost 44,000 mailboxes to Open-Xchange [9].

The same source is useful because it documents both benefits and constraints. The state reported more than EUR 15 million in licence-cost savings while budgeting EUR 9 million of one-time migration and open-source development investment for 2026. Approximately 20% of workplaces still depended on Microsoft Office in some areas because specialist applications retained technical dependencies [9].

**[Inference]** The remaining 20% is architecture evidence of switching cost: lock-in is not merely contractual. It can be embedded in document workflows, macros, integrations, identity, plugins, specialist applications, and operational knowledge.

### 3.4 Denmark: pilot first, interoperability before ideology

**[Fact]** Denmark's Ministry of Digital Affairs started a 2025 pilot of Collabora, based on LibreOffice, as an alternative to Microsoft Office in selected workflows [10]. The ministry explicitly stated that the pilot did not mean technology giants could be removed immediately. Testing focused on practical compatibility issues including templates, track changes, tables, and conversion to and from Word formats [10].

**[Inference]** Denmark's approach highlights a core architecture principle: **portability claims should be tested against real business artefacts and workflows, not only against feature checklists**.

### 3.5 France: dependency mapping across the technology stack

**[Fact]** In April 2026, the French government announced an acceleration of efforts to reduce extra-European digital dependencies. DINUM announced a move away from Windows toward Linux workstations, while ministries were directed to prepare dependency-reduction plans covering workplace systems, collaboration tools, antivirus, AI, databases, virtualisation, and network equipment [11].

This scope is notable. The objective is not limited to office software; it treats digital dependency as a **cross-stack architecture and supply-chain concern**.

## 4. Why Vendor Dependency Has Become an Architecture Problem

### 4.1 Vendor lock-in compounds across integrated platforms

A single proprietary service may be replaceable. A deeply integrated platform creates a dependency graph.

For example, an organisation may accumulate coupling across endpoint OS, office formats, identity, device management, collaboration, mail, document management, security tooling, automation, data services, and cloud APIs. Each integration can be rational in isolation while increasing the total migration surface.

**[Inference]** Lock-in should therefore be evaluated as a **system property**, not a product property. The relevant question is not only "Can this database be exported?" but "Can the organisation restore the business capability without the vendor within an acceptable time and cost?"

### 4.2 Data location does not eliminate jurisdictional questions

The US CLOUD Act made explicit that a company subject to US jurisdiction can, under valid legal process, be required to produce data within its possession, custody, or control regardless of where the company stores that data [12]. The same DOJ material also notes that comparable cross-border production principles exist in other jurisdictions; the issue should not be simplified into a claim that US authorities have unrestricted access to all foreign-hosted cloud data [12].

**[Inference]** For sovereignty-sensitive workloads, **data residency and provider jurisdiction must be analysed separately**. Region selection can address some residency requirements without, by itself, resolving every jurisdictional or operational-sovereignty requirement.

### 4.3 Availability includes more than datacentre failure

Traditional availability design focuses on machine, zone, region, network, and software failures. Public-sector sovereignty initiatives add other dependency failures:

- provider-wide service or control-plane outage;
- loss of access caused by contractual, licensing, or commercial change;
- product retirement or API incompatibility;
- legal or policy restriction on continued use;
- loss of required operational support within an approved jurisdiction;
- inability to migrate because of proprietary formats or tightly coupled specialist applications;
- upstream supply-chain or geopolitical disruption.

Not every workload requires engineering for every scenario. However, critical public functions may have continuity requirements that outlive a particular vendor relationship.

### 4.4 Open source is an option-enabler, not an automatic sovereignty guarantee

The EU Open Source Strategy treats open source as a sovereignty instrument because it can improve control, inspectability, reuse, and freedom from some forms of lock-in [3]. That does not make every open-source deployment sovereign or operationally superior.

An open-source system can still depend on a single managed-service operator, non-European infrastructure, a fragile maintainer community, proprietary identity integration, closed firmware, external update channels, or scarce internal expertise.

**[Inference]** The architecture value of open source is primarily **optionality**: under the right governance and operating model, it can expand the set of parties capable of operating, auditing, modifying, or migrating a system.

## 5. Counter-Evidence and Limits

A balanced review must account for evidence that contradicts the simplistic "Europe is abandoning Microsoft" narrative.

**[Fact]** In March 2024, the European Data Protection Supervisor found infringements in the European Commission's use of Microsoft 365, including issues involving purpose limitation and transfers of personal data outside the EU/EEA [13]. However, in July 2025 the EDPS concluded that the Commission and Microsoft had remedied the infringements examined in that decision [14]. The case therefore demonstrates both regulatory pressure **and the possibility of remediation**, not a general prohibition on Microsoft 365.

**[Fact]** Microsoft has also changed its architecture and product controls in response to European sovereignty requirements. The EU Data Boundary commits in-scope Microsoft enterprise services to store and process specified customer and personal data within the EU/EFTA boundary, subject to documented exceptions and continuing transfers [15]. Microsoft Sovereign Cloud adds controls such as Data Guardian; Microsoft describes Data Guardian as requiring European-resident oversight for defined remote-access scenarios and recording approved sessions in tamper-evident logs [16][17].

**[Fact]** The European Commission's own 2026 sovereign-cloud procurement states that non-European technologies can meet its minimum sovereignty level when operated within an appropriate framework [6]. This is direct evidence against interpreting technological sovereignty as an automatic nationality-based ban.

The alternatives also have costs. The Schleswig-Holstein and Swiss cases both document migration effort, specialist-application dependencies, compatibility constraints, and additional skills or investment requirements [8][9].

## 6. Architecture Synthesis

### 6.1 Sovereignty should be treated as a non-functional requirement

Security, availability, latency, cost, and compliance are routinely treated as non-functional requirements (NFRs). For sovereignty-sensitive systems, the evidence above supports adding **sovereignty and exitability** to the same design process.

A requirement such as "use an EU region" is too weak if the actual objective is continued national control of a critical capability. The architecture should instead state the constraint explicitly, for example:

> A Tier-0 public service must be able to continue its critical business process for 72 hours without the primary SaaS provider, and the organisation must be able to migrate authoritative data and identities to an approved alternative within an agreed recovery horizon.

The exact targets are workload-specific. What matters is that they are **testable**.

### 6.2 Exitability is different from active-active multi-vendor architecture

**[Recommendation]** Do not interpret sovereignty as a requirement to duplicate every workload across two providers. That can double cost and operational complexity while introducing consistency and security problems.

Use a proportional strategy:

1. **Critical continuity workloads:** maintain tested contingency tooling or an independently operable fallback for the minimum business process.
2. **Important but recoverable workloads:** maintain validated exports, migration automation, dependency inventory, replacement architecture, and periodic recovery exercises.
3. **Commodity workloads:** prefer standards and portable data, but accept vendor-specific capabilities when their value exceeds switching risk.

The objective is not zero dependency. The objective is **bounded dependency with a known failure and exit path**.

### 6.3 Architecture pattern: Primary platform + independent continuity path

The Swiss programme illustrates a useful pattern for selected critical workflows:

```text
                 Normal operations
                       |
                       v
              Primary SaaS platform
                       |
          +------------+------------+
          |                         |
          v                         v
   Regular exports           Dependency inventory
   + portable formats        + migration runbook
          |                         |
          +------------+------------+
                       |
                       v
             Tested exit/continuity path
                       |
        +--------------+--------------+
        |                             |
        v                             v
Sovereign fallback             Replacement platform
(minimum operations)           (planned migration)
```

This pattern is deliberately not active-active. Its purpose is to preserve **credible options** under provider, legal, contractual, or geopolitical failure modes.

### 6.4 A practical sovereignty and exitability review

For a critical cloud or SaaS dependency, an architecture review should be able to answer:

1. What business capability fails if the provider becomes unavailable, not merely what technical component fails?
2. What are the authoritative data, identity, configuration, policy, logging, and key dependencies?
3. Can those assets be exported in documented, usable formats, and has restoration into another environment been tested?
4. Which dependencies are based on proprietary protocols, file formats, APIs, macros, plugins, or specialist applications?
5. Which operators can administer the platform, from which jurisdictions, and how is privileged access approved and audited?
6. Which third-country laws or contractual terms may materially affect the workload? Legal counsel should validate this for regulated or sensitive systems.
7. What would a 2x or 3x licensing-price change do to the business case, and what is the estimated switching cost?
8. What is the minimum viable business process during a primary-provider outage or exit event?
9. What is the target time to activate a contingency path and the target time to complete a strategic migration?
10. When was the exit or continuity plan last exercised with production-representative data and integrations?

An untested export button is not an exit strategy.

## 7. Proposed Architecture Principle

**[Recommendation]**

> **For workloads whose loss would materially impair a critical public or regulated business function, vendor exitability should be a first-class NFR. The required exit capability should be proportional to criticality and validated through evidence such as portable data, dependency mapping, migration rehearsals, and continuity exercises.**

This principle does **not** imply avoiding proprietary platforms. A proprietary hyperscale cloud may still provide the strongest security, reliability, economics, and engineering velocity for a given workload. The decision should compare those benefits against the consequence and reversibility of dependency.

A useful decision model is therefore:

```text
Architecture value
= capability + reliability + security + delivery speed + economics
- dependency risk - exit cost - jurisdictional risk - operational complexity
```

The terms are not directly additive in a mathematical sense; the expression is a decision aid that makes previously hidden dependency costs explicit.

## 8. Current Conclusion

**[Fact]** European public-sector policy in 2025-2026 shows a real increase in attention to technological sovereignty, open source, provider diversification, and extra-European dependency reduction. The intensity varies considerably by jurisdiction: Schleswig-Holstein is executing a broad migration, Denmark has used a pilot, France has announced cross-stack dependency reduction, Switzerland is initially building a parallel sovereign capability, and the EU is translating sovereignty into cloud procurement criteria [4][8][9][10][11].

**[Inference]** The common architectural theme is not "remove Microsoft." It is **remove unexamined single-vendor dependency from the definition of normal architecture**.

**[Recommendation]** For solution architects, sovereignty-sensitive design should expand the usual NFR review. Availability should include vendor and legal dependency; disaster recovery should include business-capability continuity; procurement should consider switching cost; and architecture documentation should contain a tested exit path when the workload's criticality justifies one.

## 9. Open Questions

- How should organisations quantify an acceptable **Exit Time Objective (ETO)** analogous to RTO?
- Which workloads justify a live sovereign fallback versus a migration-ready cold path?
- How should portability tests measure semantic compatibility rather than only file export success?
- What percentage of total cost of ownership should reasonably be spent on preserving strategic optionality?
- Can sovereignty requirements be expressed as automated architecture-policy checks in the same way as security and compliance guardrails?
- How should architects score dependence on identity, endpoint management, observability, security tooling, and AI services that are harder to migrate than raw compute or storage?

These questions are intentionally left open because the correct thresholds depend on business criticality, regulatory context, and the cost of maintaining alternatives.

## References

1. European Commission, **Strengthening Europe's Tech Sovereignty**. https://digital-strategy.ec.europa.eu/en/policies/eu-tech-sovereignty
2. European Commission, **Communication on European Tech Sovereignty, accompanied by an EU Open Source Strategy**, 3 June 2026. https://digital-strategy.ec.europa.eu/en/library/communication-european-tech-sovereignty-accompanied-eu-open-source-strategy
3. European Commission, **The EU Open Source Strategy**. https://digital-strategy.ec.europa.eu/en/policies/open-source-strategy
4. European Commission, **Sovereign Cloud Framework explained**, 1 June 2026. https://commission.europa.eu/news-and-media/news/sovereign-cloud-framework-explained-2026-06-01_en
5. European Commission, **Proposal for the Cloud and AI Development Act (CADA)**, 2026. https://digital-strategy.ec.europa.eu/en/library/proposal-cloud-and-ai-development-act-cada
6. European Commission, **Commission advances cloud sovereignty through strategic procurement**, 17 April 2026. https://commission.europa.eu/news-and-media/news/commission-advances-cloud-sovereignty-through-strategic-procurement-2026-04-17_en
7. Swiss Federal Chancellery, **BOSS PoC feasibility study**. https://www.bk.admin.ch/en/boss-poc-feasibility-study
8. Swiss Federal Chancellery / Federal Administration, **Bundeskanzlei lanciert Programm für einen digital souveränen Arbeitsplatz**, 2 September 2026. https://www.epa.admin.ch/de/newnsb/EgX1XHIfGtUN
9. State Government of Schleswig-Holstein, **LibreOffice ersetzt Microsoft: Schon fast 80 Prozent der Arbeitsplätze auf quelloffene Office-Lösung umgestellt**, 4 December 2025. https://www.schleswig-holstein.de/DE/landesregierung/ministerien-behoerden/I/Presse/PI/2025/cds/251204_cds_open-source
10. Danish Ministry of Digital Affairs, **Digitaliseringsministeriet sætter gang i pilotprojekt om digital suverænitet**, 19 June 2025. https://www.digmin.dk/digitalisering/nyheder/nyhedsarkiv/2025/jun/digitaliseringsministeriet-saetter-gang-i-pilotprojekt-om-digital-suveraenitet
11. French Ministry of Economy and Finance, **Souveraineté numérique : l'État accélère la réduction de ses dépendances extra-européennes**, 9 April 2026. https://presse.economie.gouv.fr/souverainete-numerique-letat-accelere-la-reduction-de-ses-dependances-extra-europeennes/
12. U.S. Department of Justice, **Justice Department Announces Publication of White Paper on the CLOUD Act**, 10 April 2019. https://www.justice.gov/archives/opa/pr/justice-department-announces-publication-white-paper-cloud-act
13. European Data Protection Supervisor, **European Commission's use of Microsoft 365 infringes data protection law for EU institutions and bodies**, 11 March 2024. https://www.edps.europa.eu/press-publications/press-news/press-releases/2024/european-commissions-use-microsoft-365-infringes-data-protection-law-eu-institutions-and-bodies_fr
14. European Data Protection Supervisor, **European Commission brings use of Microsoft 365 into compliance with data protection rules for EU institutions and bodies**, 28 July 2025. https://www.edps.europa.eu/press-publications/press-news/press-releases/2025/european-commission-brings-use-microsoft-365-compliance-data-protection-rules-eu-institutions-and-bodies_de
15. Microsoft Learn, **What is the EU Data Boundary?** https://learn.microsoft.com/en-us/privacy/eudb/eu-data-boundary-learn
16. Microsoft Learn, **What is Microsoft Sovereign Cloud?** https://learn.microsoft.com/en-us/azure/azure-sovereign-clouds/microsoft-sovereign-cloud
17. Microsoft Learn, **Data Guardian overview**. https://learn.microsoft.com/en-us/azure/azure-sovereign-clouds/public/data-guardian
18. TechRadar, **Swiss federal government is ditching Microsoft on thousands of devices to switch to open source**, 8 September 2026. Secondary reporting used as the starting point for this review. https://www.techradar.com/pro/swiss-federal-government-is-ditching-microsoft-on-thousands-of-devices-to-switch-to-open-source
