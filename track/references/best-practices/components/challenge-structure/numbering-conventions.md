# Numbering Conventions

Evaluates whether challenge directories follow consistent sequential numbering and organizational conventions.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | No numbering scheme; directories use random or inconsistent names; order unclear from the filesystem |
| 2 | Below Standard | Some numbering present but inconsistent -- gaps in sequence, mixed formats (01- and 1-), or numbering does not match logical track order |
| 3 | Adequate | Sequential numbering present and consistent, but format does not match organizational convention or slugs are not descriptive |
| 4 | Good | Sequential, consistent numbering (01-, 02-, etc.); directory order matches logical challenge order; slug in assignment.md matches directory name (production baseline) |
| 5 | Excellent | Slug names are descriptive and URL-friendly; numbering format matches org convention; random suffix used where needed for cross-track collision avoidance |

## Guidance

Challenge directories must be numbered sequentially to make filesystem ordering match the logical track progression. The standard format is a zero-padded two-digit prefix followed by a hyphen and a descriptive slug.

### Standard Format

```
01-introduction/
02-configure-vault/
03-create-secrets/
04-rotate-keys/
05-cleanup-and-review/
```

The number prefix ensures `ls` output matches track order. The slug after the prefix should be descriptive and match the `slug:` field in the challenge's `assignment.md` frontmatter.

### How Slug Relates to Directory Name

The `slug:` field in each challenge's `assignment.md` frontmatter is the directory name minus the number prefix:

```
Directory: 03-create-secrets/
assignment.md slug: create-secrets
```

### Double-Number Format

Some organizations use a double-number format to group challenges by section:

```
01-01-welcome/
01-02-setup-environment/
02-01-deploy-application/
02-02-configure-networking/
03-01-verify-and-test/
```

Here the first number is the section, the second is the challenge within that section. Use this format only if the organization has established it as a convention.

### Random Suffix Convention

When challenges might be reused across multiple tracks (shared challenge libraries), a random suffix prevents slug collisions:

```
01-configure-vault-a1b2c3/
02-create-secrets-d4e5f6/
```

This is uncommon and only needed for organizations that share challenge directories across track repositories.

Good -- clean sequential numbering:

```
01-introduction/
02-install-terraform/
03-write-configuration/
04-plan-and-apply/
05-manage-state/
```

Good -- numbering matches assignment.md slugs:

```yaml
# Each challenge's assignment.md slug matches its directory name (minus prefix):
# 01-introduction/assignment.md       → slug: introduction
# 02-install-terraform/assignment.md   → slug: install-terraform
# 03-write-configuration/assignment.md → slug: write-configuration
```

Bad -- inconsistent numbering:

```
1-intro/
02-setup/
3-deploy/
04-verify/
```

Bad -- gaps in sequence:

```
01-introduction/
02-configure/
05-deploy/
08-verify/
```

Bad -- numbering does not match logical challenge order:

```
# Filesystem: 01-deploy/, 02-configure/
# But the track logically starts with configure, then deploy
```

Bad -- no numbering:

```
introduction/
configure-vault/
create-secrets/
```

## What to Watch For

- Mixed numbering formats (some zero-padded, some not) within the same track
- Directory order not matching the logical challenge progression -- this causes confusion when editing
- Non-descriptive slugs after the number prefix (01-lab1/, 02-lab2/) that give no hint about content
- Gaps in the numbering sequence suggesting deleted challenges that were not renumbered
- Slugs with special characters, uppercase letters, or spaces -- stick to lowercase, hyphenated, ASCII
- Double-number format used inconsistently (some challenges use it, others do not)
- Slug in assignment.md not matching the directory name (minus the number prefix)
