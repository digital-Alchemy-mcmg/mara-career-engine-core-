# MARA Pipeline State Schema — Airtable

This is the **canonical state surface** for the job-search pipeline. All AI agents read from and write to this schema.

---

## TABLE 1: FRAGMENT_INVENTORY

Tracks all code fragments, processes, and functions as they're ingested.

| Field | Type | Description |
|-------|------|-------------|
| FRAGMENT_ID | Single Line Text | Auto-generated or provided (e.g., `MARA_SCORER_V1`) |
| NAME | Single Line Text | Human-readable name (e.g., "ATS Constraint Scoring Engine") |
| TYPE | Single Select | `function` \| `schema` \| `process` \| `decision` \| `router` |
| DESCRIPTION | Long Text | What this fragment does |
| PLATFORM | Single Select | `JavaScript` \| `Python` \| `SQL` \| `Conceptual` |
| PHASE | Single Select | `pre_filter` \| `ingress` \| `transform` \| `projection` \| `assembly` \| `audit` \| `output` |
| INPUT_SIGNATURE | Long Text | JSON schema or description of expected input |
| OUTPUT_SIGNATURE | Long Text | JSON schema or description of output |
| DEPENDENCIES | Multiple Records Link | [Link to other FRAGMENT_INVENTORY records this needs] |
| SOURCE_URL | URL | Link to GitHub, Google Drive, or wherever code lives |
| CODE_SNIPPET | Long Text | Actual code (if short) or summary |
| OWNER_AI | Single Select | `Claude` \| `GPT` \| `Gemini` \| `Kimi` \| `Quinn` \| `Comet` |
| STATUS | Single Select | `pending_clarification` \| `normalized` \| `positioned` \| `ready_for_build` |
| CONFIDENCE | Single Select | `high` \| `medium` \| `low` |
| CONFLICTS | Multiple Records Link | [Link to FRAGMENT_INVENTORY records this conflicts with] |
| NOTES | Long Text | Issues, blockers, implementation notes |
| CREATED | Date | When this fragment was first logged |
| UPDATED | Date | Last modified |

---

## TABLE 2: PIPELINE_EXECUTION_LOG

Captures the full audit trail for each resume generation run.

| Field | Type | Description |
|-------|------|-------------|
| EXECUTION_ID | Single Line Text | UUID for this run (e.g., `EXEC_20250530_123456`) |
| TIMESTAMP | Date & Time | When execution started |
| CANDIDATE_ID | Single Line Text | Who is this for? (link to candidate profile if applicable) |
| JOB_POST_ID | Single Line Text | Which job post? |
| JOB_POST_URL | URL | Original job posting URL |
| PIPELINE_VERSION | Single Line Text | Which version of the pipeline ran? |
| PHASE_COMPLETED | Single Select | Tracks completion: `ingress` \| `transform` \| `projection` \| `assembly` \| `audit` \| `complete` |
| **— INGRESS PHASE —** | | |
| CANDIDATE_PROFILE_RECEIVED | Checkbox | Did we get valid candidate data? |
| RESUME_VERSIONS_LOADED | Multiple Select | `v1_operations` \| `v2_people_ops` \| `industry_specific` \| [other] |
| PROFILE_VALIDATION_PASS | Checkbox | Did profile pass schema validation? |
| **— TRANSFORM PHASE —** | | |
| JOB_REQUIREMENTS_EXTRACTED | Checkbox | Successfully parsed job post? |
| REQUIREMENT_FIELDS | JSON | Key extracted: title, required_skills[], preferred_skills[], experience_years, industry |
| **— PROJECTION PHASE (MARA) —** | | |
| CONSTRAINT_SCORES | JSON | { constraint_name: score, ... } |
| MARA_DIMENSION_1 | Percent | Skill match % |
| MARA_DIMENSION_2 | Percent | Experience relevance % |
| MARA_DIMENSION_3 | Percent | Industry alignment % |
| MARA_DIMENSION_4 | Percent | Role title proximity % |
| OVERALL_MATCH | Percent | Weighted average of dimensions |
| PROJECTION_STRATEGY | Single Select | Which strategy applied: `highlight_match_sections` \| `reorder_experience` \| `contextual_framing` \| `multiple_versions` |
| CONSTRAINTS_FAILED | Multiple Select | Which constraints didn't meet threshold? |
| **— ASSEMBLY PHASE —** | | |
| RESUME_VERSION_SELECTED | Single Select | Which base resume used? |
| SECTIONS_REORDERED | JSON | { section_name: [original_position → new_position], ... } |
| SECTIONS_MODIFIED | Multiple Select | Which sections had text adjustments (truthful only)? |
| SECTIONS_CULLED | Multiple Select | Which sections removed? Why? |
| **— AUDIT PHASE —** | | |
| TRUTH_VALIDATION_PASS | Checkbox | Did human/system confirm no embellishment? |
| IDEMPOTENCY_CHECK | Checkbox | Running again produces same output? |
| CONFLICTS_DETECTED | Multiple Records Link | [Link to conflict records, if any] |
| **— OUTPUT —** | | |
| RESUME_OUTPUT_PATH | URL | Where is the final resume stored? (Google Drive link?) |
| RESUME_OUTPUT_HASH | Single Line Text | SHA256 or MD5 of final output (for idempotency verification) |
| EXECUTION_STATUS | Single Select | `success` \| `partial_success` \| `failed` |
| ERROR_MESSAGE | Long Text | If failed, what went wrong? |
| NOTES | Long Text | Additional context, decisions made, human notes |
| CREATED | Date & Time | Auto-generated |

---

## TABLE 3: MARA_CONSTRAINT_DEFINITIONS

Defines the scoring dimensions and thresholds for the constraint engine.

| Field | Type | Description |
|-------|------|-------------|
| CONSTRAINT_ID | Single Line Text | e.g., `SKILL_MATCH`, `EXP_RELEVANCE`, `INDUSTRY_ALIGN` |
| NAME | Single Line Text | Human label |
| DESCRIPTION | Long Text | What does this dimension measure? |
| WEIGHT | Percent | Importance in overall scoring (should sum to 100%) |
| THRESHOLD_MIN | Percent | Minimum acceptable score |
| THRESHOLDIDEAL | Percent | Ideal threshold (target) |
| SCORING_METHOD | Single Select | `exact_match` \| `fuzzy_match` \| `keyword_presence` \| `categorical` \| `years_of_exp` |
| CALCULATION_LOGIC | Long Text | Pseudocode or description of how score is computed |
| RESUME_IMPACT | Single Select | How does failure affect resume: `culled` \| `reordered` \| `reframed` \| `optional` |
| TRUTH_RULE | Long Text | What is the truthfulness boundary? (Never claim what candidate doesn't have.) |
| OWNER_AI | Single Select | Which engine computes this? |
| CREATED | Date | |
| UPDATED | Date | |

**Example Records:**

```text
CONSTRAINT_ID: SKILL_MATCH
NAME: Technical Skill Alignment
DESCRIPTION: % of required skills candidate actually has
WEIGHT: 35%
THRESHOLD_MIN: 60%
THRESHOLD_IDEAL: 85%
SCORING_METHOD: fuzzy_match (tools/languages close to req'd)
RESUME_IMPACT: reordered (move matching skills to top)
TRUTH_RULE: "Only list skills demonstrably used in past roles"
```

```text
CONSTRAINT_ID: EXP_RELEVANCE
NAME: Experience Relevance
DESCRIPTION: Alignment of past job titles/domains with target
WEIGHT: 25%
THRESHOLD_MIN: 50%
THRESHOLD_IDEAL: 75%
SCORING_METHOD: categorical (domain → domain mapping)
RESUME_IMPACT: reframed (contextualize how past exp applies)
TRUTH_RULE: "Descriptions must not overstate or reverse scope of past role"
```

```text
CONSTRAINT_ID: YEARS_OF_EXP
NAME: Years in Field
DESCRIPTION: Total years in relevant domain
WEIGHT: 20%
THRESHOLD_MIN: job-specific (req'd years - 1)
THRESHOLD_IDEAL: job-specific (req'd years +2)
SCORING_METHOD: years_of_exp (exact from profile)
RESUME_IMPACT: optional (if below, may not highlight "years" callout)
TRUTH_RULE: "Years are factual; no padding"
```

```text
CONSTRAINT_ID: INDUSTRY_ALIGN
NAME: Industry Alignment
DESCRIPTION: Candidacy's industry background vs. target
WEIGHT: 15%
THRESHOLD_MIN: 40% (transferable skills OK)
THRESHOLD_IDEAL: 80% (direct industry match)
SCORING_METHOD: categorical (industry → industry proximity)
RESUME_IMPACT: culled (if <20%, remove non-core industry work)
TRUTH_RULE: "Do not misrepresent industry of past employers"
```

```text
CONSTRAINT_ID: TITLE_PROXIMITY
NAME: Role Title Proximity
DESCRIPTION: How close is candidate's past titles to target role?
WEIGHT: 5%
THRESHOLD_MIN: 30% (lateral moves count)
THRESHOLD_IDEAL: 70%
SCORING_METHOD: fuzzy_match (job title similarity)
RESUME_IMPACT: reframed (emphasize relevant duties over title)
TRUTH_RULE: "Duties must match historical job description"
```
