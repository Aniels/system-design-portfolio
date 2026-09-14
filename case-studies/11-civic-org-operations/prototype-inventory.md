# CivicOrgOps Prototype Inventory

## Purpose

This file records the recoverable prototype artifacts and known version ambiguity from the archived ChatGPT Project.

## Recoverable artifact names

- `index.html`
- `finance.html`
- `erp.html`
- `mobilization.html`
- `members.html`
- `legal.html`
- `petitions.html`
- `shops.html`
- `donations.html`
- `broadcast.html`
- `style.css`
- `app.js`

## Confirmed prototype tracks

### Admin / back-office track

Recovered pages include a backend-management dashboard covering finance, materials, mobilization, membership, legal, petitions, and local shops. The administration-oriented mobilization page includes application approval and notification sending.

### Campaign / war-room track

A different `index.html` represents a「選戰作戰室」with one-click broadcast, fundraising, and material-dispatch actions. Related pages include campaign-oriented donations, ERP, mobilization, legal, petition, local-shop, and broadcast routes; some are explicit routing/layout placeholders.

### Participant-facing track

A separate mobilization page is labeled「參與者中心」and focuses on activity registration plus meeting time/location.

## Shared frontend assets

`style.css` provides the common card/grid/nav/table/button system. `app.js` contains mock state for donations, inventory, events, and one member plus simple DOM render helpers.

## Version ambiguity

Some same-name HTML files appeared with different content in the recoverable source set. Therefore:

- do not treat same filename as duplicate evidence;
- do not infer that the currently mounted filesystem copy is the only historical version;
- preserve admin / war-room / participant tracks separately until original conversation timestamps, zip files, or commits establish lineage.

## Migration status

| Asset group | Status | Reason |
|---|---|---|
| Admin prototype semantics | Migrated | Architecture context preserved in GitHub + Notion |
| War-room prototype semantics | Migrated | Architecture context preserved in GitHub + Notion |
| Participant prototype semantics | Migrated | Architecture context preserved in GitHub + Notion |
| Current static files | Needs Review | Files are recoverable, but duplicate-name historical lineage is ambiguous |
| Original historical file versions | Pending | Must be recovered from original conversations/export/repository if they are to be preserved byte-for-byte |

## Safe-delete implication

This prototype inventory is intentionally conservative. Until the original Project conversations are completely scanned or exported and the duplicate-name variants are resolved, the source Project is **not safe to delete**.
