# MARA Execution Constraints

## Directory Write Limits
Modules are strictly forbidden from writing outside their assigned directories.

Specifically:
- Module B is strictly limited to reading from `01_CANONICAL_DNA` and writing to `02_ARTIFACT_REGISTRY`.
- All changes to governance, specifications, or schemas must be recorded in the `/04_AUDIT_AND_TRUTH/` directory with a corresponding audit entry.
