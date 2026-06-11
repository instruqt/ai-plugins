# Config Errors

YAML validation errors and resource misconfigurations in Instruqt track config files.

## Empty cloud env vars

**Symptom:** Silent auth failures in AWS/Azure/GCP CLI calls.

**Cause:** Referencing `INSTRUQT_AWS_ACCOUNT_*` (or equivalent) without a matching `aws_accounts:` block in `config.yml`.

**Fix:** Ensure the resource block exists in `config.yml` and the name matches the environment variable reference.

---

## Invalid GCP IAM role

**Symptom:** Sandbox provisioning fails or permissions are missing.

**Cause:** Using a non-existent role like `roles/writer`.

**Fix:** Verify against the GCP IAM predefined roles list. Use the correct role name (e.g., `roles/storage.objectCreator`).

---

## Null bytes in YAML (Windows editing)

**Symptom:** `instruqt track push` fails with "invalid byte sequence for encoding UTF8: 0x00".

**Cause:** Files edited on Windows-mounted volumes accumulate trailing null bytes.

**Fix:** Strip null bytes before pushing (e.g., `tr -d '\0' < file > file.clean && mv file.clean file`).

---

## Machine type not available

**Symptom:** Sandbox fails to provision the VM.

**Cause:** Invalid or unavailable `machine_type` string in the config.

**Fix:** Use standard GCP machine types (`n1-standard-2`, `e2-standard-4`, etc.).

---

## Missing provision_ssl_certificate

**Symptom:** HTTPS service tabs fail to load.

**Cause:** The VM doesn't have an SSL certificate provisioned.

**Fix:** Add `provision_ssl_certificate: true` to the VM config block.

---

## Website tab URL not HTTPS

**Symptom:** `instruqt track push` validation fails.

**Cause:** `type: website` tab `url:` must use HTTPS.

**Fix:** Ensure all website tab URLs use `https://`.

---

## Secrets in environment block

**Symptom:** Credentials visible in git history and debug log.

**Cause:** API keys placed in the `environment:` block of `config.yml` instead of `secrets:`.

**Fix:** Move sensitive values to the `secrets:` block.
