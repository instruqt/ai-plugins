# Account Pool Checkout

External account or tenant pool checkout for per-session access to pre-provisioned environments. Use when the track needs access to a SaaS product, external cluster, or pre-configured environment that cannot be provisioned from scratch per session.

## What It Provides

A pattern where setup scripts call an external API to reserve (check out) an account or tenant from a pool, store the credentials in agent variables for use during the session, and cleanup scripts release (check in) the account when the session ends.

## How It Works

1. **Setup script**: Calls an external pool manager API to reserve an account. Stores the returned credentials (URL, username, password, API key) as agent variables so they are accessible in subsequent challenges.
2. **Challenge scripts**: Read the credentials from agent variables and use them for learner exercises.
3. **Cleanup script**: Calls the pool manager API to release the account back to the pool and reset its state.

## Config.yml Example

```yaml
version: "3"
containers:
  - name: shell
    image: gcr.io/instruqt/cloud-client:2
    ports:
      - 8080
    memory: 512
    shell: /bin/bash

secrets:
  - name: POOL_API_KEY                            # auth for the pool manager API
  - name: POOL_API_URL                            # base URL of the pool manager
```

### Setup Script (Checkout)

```bash
#!/bin/bash
set -euxo pipefail

RESPONSE=$(curl -sf \
  -H "Authorization: Bearer ${POOL_API_KEY}" \
  -X POST "${POOL_API_URL}/checkout")

# Store credentials as agent variables for use across challenges
agent variable set TENANT_URL "$(echo "$RESPONSE" | jq -r '.url')"
agent variable set TENANT_USER "$(echo "$RESPONSE" | jq -r '.username')"
agent variable set TENANT_PASS "$(echo "$RESPONSE" | jq -r '.password')"
```

### Cleanup Script (Checkin)

```bash
#!/bin/bash

TENANT_URL=$(agent variable get TENANT_URL)

# Release the account back to the pool -- this MUST succeed
curl -sf \
  -H "Authorization: Bearer ${POOL_API_KEY}" \
  -X POST "${POOL_API_URL}/checkin" \
  -d "{\"url\": \"${TENANT_URL}\"}" || true
```

## Common Pitfalls

- **Missing or failing cleanup**. If the cleanup script does not release the account, the pool gradually depletes. Every checked-out account must be returned. Use `|| true` on the final curl to prevent cleanup failures from masking the release call, but also add monitoring/alerting on the pool manager side.
- **No retry on checkout failure**. The pool may be temporarily exhausted. Add a retry loop with backoff in the setup script rather than failing immediately.
- **Storing credentials only in environment variables**. Environment variables set in a setup script are not available in later challenges. Use `agent variable set` to persist values across the session lifecycle.
- **Pool size underestimated**. Peak concurrent sessions can exceed expectations during events or product launches. Size the pool for peak, not average, demand.
- **Account state not reset on checkin**. If the pool manager does not reset the account after checkin (delete learner-created resources, reset passwords), the next learner gets a dirty environment. Build reset logic into the checkin flow.

## Example Tracks

*These examples are illustrative, not prescriptive -- adapt to your specific needs.*

- SaaS administration workshop where each learner gets a pre-configured tenant
- Database cluster lab with pooled pre-provisioned clusters too slow to create per session
- Enterprise software trial where license keys are limited and must be recycled
- Multi-tenant platform tutorial with isolated namespaces checked out from a shared cluster
