# deps.dev Connector

This source queries package metadata, dependency graphs, and security advisories from the [deps.dev API](https://docs.deps.dev/api/v3/).
No credentials are required.

## Start querying

Inspect metadata for a package version:

```sql
SELECT version, published_at, licenses, advisory_keys, related_projects
FROM deps_dev.versions
WHERE system = 'NPM'
  AND package_name = 'minimist'
  AND version = '0.0.8'
LIMIT 1;
```

Fetch the dependency graph nodes for a package version:

```sql
SELECT dependency_system, dependency_name, dependency_version, relation
FROM deps_dev.dependencies
WHERE system = 'NPM'
  AND package_name = 'minimist'
  AND version = '0.0.8';
```

Fetch full advisory details for an OSV ID or GHSA:

```sql
SELECT title, description, cvss3_score, aliases
FROM deps_dev.advisories
WHERE advisory_id = 'GHSA-vh95-rmgr-6w4m'
LIMIT 1;
```

## Tables

### By required filter

| Filter pattern | Tables | Example |
|---|---|---|
| `system` + `package_name` | 1 | `WHERE system = 'NPM' AND package_name = 'minimist'` |
| `system` + `package_name` + `version` | 2 | `WHERE system = 'NPM' AND package_name = 'minimist' AND version = '0.0.8'` |
| `advisory_id` | 1 | `WHERE advisory_id = 'GHSA-vh95-rmgr-6w4m'` |

### versions

Fetches metadata for one package version. Maps to
`GET /v3/systems/{system}/packages/{package_name}/versions/{version}`.

Useful columns include:
- `licenses`
- `advisory_keys`
- `links`
- `related_projects`
- `registries`

### packages

Fetches project-level details and a list of all available versions. Maps to
`GET /v3/systems/{system}/packages/{package_name}`.

### dependencies

Fetches the resolved dependency graph (nodes) for a specific package version. Maps to
`GET /v3/systems/{system}/packages/{package_name}/versions/{version}:dependencies`.

### advisories

Fetches detailed vulnerability/advisory metadata given an advisory key (e.g., OSV/GHSA ID). Maps to
`GET /v3/advisories/{advisory_id}`.

## Supported systems

deps.dev supports package systems such as `NPM`, `PYPI`, `MAVEN`, `GO`, and
`CARGO`. Use the system names exactly as expected by deps.dev (usually uppercase).

For package names that require path escaping (such as scoped npm packages like `@types/react`), ensure you URL-encode them as `%40types%2Freact` when using this source.

