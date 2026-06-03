# Hot Start Awareness

Evaluates whether setup scripts handle hot-start and invite-link scenarios where user-specific environment variables are empty or unset.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | Setup scripts reference INSTRUQT_USER_EMAIL, INSTRUQT_USER_NAME, or INSTRUQT_PARTICIPANT_ID without any guards; hot-start environments fail or produce broken configuration |
| 2 | Below Standard | Some user env var references have guards but others do not; inconsistent handling across setup scripts |
| 3 | Adequate | Most user env var references have fallbacks, but fallback values are not meaningful (e.g., empty string) |
| 4 | Good | All user env var references use ${VAR:-fallback} with sensible default values; setup completes successfully in both cold-start and hot-start (production baseline) |
| 5 | Excellent | Setup works identically in cold-start, hot-start, and invite-link contexts; user-personalized elements degrade gracefully with meaningful placeholders |

## Guidance

Instruqt provides several user-specific environment variables:

- `INSTRUQT_USER_EMAIL` -- the learner's email address
- `INSTRUQT_USER_NAME` -- the learner's display name
- `INSTRUQT_PARTICIPANT_ID` -- unique ID for this track session

In **cold-start** (normal launch), these are populated before setup runs. In **hot-start** (pre-provisioned environments) and **invite-link** contexts, these variables may be empty or unset because the environment was created before a user claimed it.

Good -- fallback for user email in API provisioning:

```bash
USER_EMAIL="${INSTRUQT_USER_EMAIL:-learner@example.com}"
curl -X POST https://api.example.com/users \
  -d "{\"email\": \"${USER_EMAIL}\"}"
```

Good -- fallback for display name in welcome message:

```bash
USER_NAME="${INSTRUQT_USER_NAME:-Learner}"
echo "Welcome, ${USER_NAME}!" > /root/welcome.txt
```

Good -- conditional logic based on variable presence:

```bash
if [ -n "${INSTRUQT_USER_EMAIL:-}" ]; then
  # Personalized setup for cold-start
  create_user_account "$INSTRUQT_USER_EMAIL"
else
  # Generic setup for hot-start
  create_user_account "learner@example.com"
fi
```

Bad -- unguarded reference that fails in hot-start:

```bash
# INSTRUQT_USER_EMAIL is empty in hot-start; this creates a broken account
curl -X POST https://api.example.com/users \
  -d "{\"email\": \"${INSTRUQT_USER_EMAIL}\"}"
```

Bad -- using the variable in a file path without guard:

```bash
# Empty variable produces a broken path: /home//config
mkdir -p "/home/${INSTRUQT_USER_NAME}/config"
```

Bad -- assuming participant ID is always set for resource naming:

```bash
# Empty INSTRUQT_PARTICIPANT_ID creates unnamed resources
gcloud compute instances create "vm-${INSTRUQT_PARTICIPANT_ID}"
```

## What to Watch For

- Any reference to `INSTRUQT_USER_EMAIL`, `INSTRUQT_USER_NAME`, or `INSTRUQT_PARTICIPANT_ID` must use `${VAR:-fallback}` syntax
- Fallback values should be meaningful, not empty strings -- an empty fallback is as broken as no fallback
- If the variable is used to create unique resource names, the fallback must still produce a valid unique name (e.g., use `$RANDOM` or a timestamp)
- Test setup scripts with these variables explicitly unset to verify hot-start behavior
- Per-challenge setup scripts may also reference these variables -- the same guards apply
- Some tracks use these variables for external API calls (SaaS provisioning, SSO setup); empty values can create orphaned resources or broken accounts
