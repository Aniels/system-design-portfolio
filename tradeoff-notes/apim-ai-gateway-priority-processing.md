# APIM AI Gateway and Foundry Priority Processing: Quota Scope Validation

## Status

**Research / validation note.** Do not treat deployment-level quota behavior as confirmed until current Microsoft documentation or controlled testing proves it.

## Question

If Azure AI Foundry Priority Processing has a ramp rule described as limiting how quickly capacity/priority traffic can increase over a 15-minute window, can Azure API Management AI Gateway load-balance across two deployments to increase the effective allowance?

The answer depends on one thing first: **what scope owns the quota/ramp rule?**

Possible scopes include:

- deployment;
- model deployment family;
- region;
- subscription/account;
- another service-defined quota domain.

## Gateway vs quota domain

APIM AI Gateway can provide:

- backend routing;
- policy enforcement;
- load balancing;
- retries/circuit breaking where appropriate;
- observability and request correlation.

But a gateway does not redefine a backend platform quota. It only increases usable aggregate capacity if the backends are genuinely independent quota domains.

```text
Client
  |
  v
APIM AI Gateway
  |-------------------|
  v                   v
Deployment A        Deployment B
  |                   |
  +------ quota/ramp enforcement scope ? ------+
```

Therefore:

- If A and B are independent quota domains, splitting traffic may increase aggregate usable capacity.
- If the rule is enforced at a broader shared scope, adding deployments does not bypass the rule.

## Proposed controlled experiment

### Setup

Create two otherwise equivalent model deployments, A and B.

Capture at minimum:

- timestamp;
- deployment identifier;
- request count/rate;
- token rate;
- response status;
- response headers relevant to throttling/priority;
- latency;
- Azure metrics for each deployment;
- APIM backend-selection logs when the gateway is introduced.

### Phase 1 — direct-to-deployment isolation test

1. Establish a stable baseline on both A and B.
2. Increase only A toward the suspected ramp threshold.
3. Keep B unchanged.
4. Observe whether throttling/priority behavior appears only on A or also on B.

Interpretation:

- A affected, B unaffected -> evidence consistent with a narrower enforcement scope.
- A and B affected together -> evidence consistent with a shared higher-level quota scope.

This is evidence, not proof; repeat runs and region/subscription controls are required.

### Phase 2 — APIM routing test

Repeat the same traffic pattern through APIM with explicit backend routing/correlation so gateway policy can be separated from model-service behavior.

The experiment should answer two independent questions:

1. Can APIM distribute requests as intended?
2. Does the backend service treat the deployments as independent Priority Processing quota domains?

## Failure modes / cautions

- APIM retries can accidentally amplify token/request load and distort the experiment.
- Unequal deployment configuration can invalidate comparison.
- Traffic needs to be measured in the same unit used by the quota rule (requests, tokens, PTU-equivalent, etc.).
- Region/subscription differences may create false conclusions.
- A transient 429 does not by itself identify which quota was hit.

## Current conclusion

Do **not** frame “two deployments behind APIM” as a quota-bypass architecture. Frame it as a load-distribution design whose effective aggregate limit depends on backend quota scope.

## Open validation

- Exact Priority Processing ramp-rule wording.
- Enforcement scope.
- Region/subscription isolation.
- Throttling response signals and metrics.
- Whether gateway retries/fallback are compatible with the service’s intended Priority Processing semantics.
