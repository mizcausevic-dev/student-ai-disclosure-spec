# Student AI Disclosure v0.1 — Specification

**Status:** Draft
**Version:** 0.1.0
**Editor:** Miz Causevic
**License:** AGPL-3.0 (this document, schema, and examples). Implementations are unrestricted.

RFC 2119 keywords (**MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, **MAY**) apply throughout.

---

## 1. Scope

This specification defines a JSON document format for **Student AI Disclosures** — machine-readable declarations attached to (or accompanying) student-submitted work that state how AI was used in producing that work.

A Student AI Disclosure is consumed by:

- **Teachers and graders** reviewing whether an assignment complied with the course's AI policy
- **Learning Management Systems** (Canvas, Schoology, Google Classroom, Moodle) attaching the disclosure as submission metadata
- **Academic integrity offices** auditing patterns across submissions
- **Auditors and accreditation bodies** sampling disclosures as part of program review
- **Students themselves**, who get a defensible, structured record of what they did and didn't do

A Student AI Disclosure is **not** an honor pledge. It is a structured declaration of *AI usage facts* — what tool was used, in what role, for what fraction of the artifact, with optional prompt evidence. The teacher's interpretation of those facts against course policy is layered on top via `teacher_acknowledged`.

A Student AI Disclosure **SHOULD** reference the underlying [AI Tutor Card](https://github.com/mizcausevic-dev/ai-tutor-card-spec) (via `tools_used[].tutor_card_uri`) or [Agent Card](https://github.com/mizcausevic-dev/agent-cards-spec) (via `tools_used[].agent_card_uri`) when one exists, so a reviewer can chain through to vendor-side disclosure.

A Student AI Disclosure **SHOULD** reference the operative [Classroom AI Acceptable Use Policy](https://github.com/mizcausevic-dev/classroom-ai-aup-spec) (via `aup_uri`) when one exists, so the disclosure binds to the policy in force at submission time.

## 2. Terminology

- **Disclosure** — a single Student AI Disclosure document; one disclosure per artifact submission.
- **Artifact** — the student-submitted work (essay, code file, slide deck, etc.) the disclosure describes.
- **Role** — the function AI played in producing the artifact (brainstorm, draft, edit, etc.).
- **Assistance extent** — a coarse qualitative judgment of how much of the artifact AI is responsible for.
- **Assistance percentage** — an optional quantitative estimate (0–100) of the AI-originated fraction.
- **Prompt evidence** — the prompts the student gave to the AI tool, either in full text or as canonical SHA-256 hashes.
- **Hash mode** — privacy-preserving mode where prompt evidence is stored only as hashes, not full text. Useful when prompts contain personally identifiable information or trade-secret context.

## 3. Design philosophy

Three principles drive the design:

### 3.1 Disclosure is the lawful state

**The default expectation is that AI was used and disclosed honestly.** A disclosure with `ai_used: false` is a valid disclosure. A disclosure with `ai_used: true` and a list of roles is a valid disclosure. *No* disclosure attached to an assignment that requires one is the failure mode — not "AI was used."

This spec does not punish AI use. It punishes *omission*.

### 3.2 Bind disclosure to artifact

The disclosure carries an `artifact_hash` (SHA-256 over the submitted artifact). A teacher reviewing a submission can recompute the hash and prove the disclosure binds to the specific file in front of them. This forecloses the failure mode "the student updated the work after disclosing."

### 3.3 Privacy-preserving prompt evidence

Some prompts contain context the student doesn't want preserved (medical history, family information, draft sentences that reveal what the student already knew). The spec defines a **hash mode** for prompt evidence: store the canonical SHA-256 of each prompt instead of the raw text. The teacher cannot read the prompts, but a forensic investigation can later confirm whether a given prompt was used.

A disclosure **MUST** declare its prompt mode via `prompt_evidence_mode`.

## 4. Document structure

### 4.1 `disclosure_version` (required)

A semver string. **MUST** be `"0.1"` for documents conforming to this draft.

### 4.2 `disclosure_id` (required)

A stable identifier for this disclosure. **SHOULD** be a UUID v4 or a similarly unique identifier within the LMS.

### 4.3 `created_at` (required)

An ISO 8601 timestamp (with timezone, **SHOULD** be `Z` / UTC) for when the student authored the disclosure.

### 4.4 `student` (required)

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | string | yes | LMS-side student identifier. **SHOULD** be a pseudonymous ID (not full legal name) for FERPA hygiene. |
| `display_name` | string | no | Student's display name. **MAY** be omitted to minimize PII. |
| `grade_or_year` | string | no | E.g. `"9"`, `"undergrad"`, `"grad"`, `"adult-ed"`. |
| `institution_id` | string | no | School / district / university identifier. |

### 4.5 `assignment` (required)

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | string | yes | LMS-side assignment identifier. |
| `title` | string | yes | Human-readable assignment title. |
| `course_id` | string | yes | LMS course identifier. |
| `lms` | string | no | E.g. `"canvas"`, `"schoology"`, `"google-classroom"`, `"moodle"`, `"d2l-brightspace"`. |
| `due_at` | ISO 8601 | no | When the assignment was due. |

### 4.6 `ai_used` (required)

Boolean. `true` if any AI tool was used at any role on the artifact. `false` if no AI tool was used.

If `ai_used` is `false`, the fields `tools_used`, `roles`, `assistance_extent`, `assistance_pct`, `prompt_evidence_mode`, and `prompts` **MUST** be omitted.

### 4.7 `tools_used` (conditional)

**Required** when `ai_used` is `true`. **MUST** be a non-empty array. Each entry:

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | yes | Tool name as used by the student, e.g. `"Claude.ai"`, `"ChatGPT"`, `"GitHub Copilot"`. |
| `provider` | string | no | Vendor name. |
| `version` | string | no | Tool version or model identifier if known, e.g. `"claude-opus-4-7"`. |
| `agent_card_uri` | URI | no | Back-reference to the tool's [Agent Card](https://github.com/mizcausevic-dev/agent-cards-spec) declaration. |
| `tutor_card_uri` | URI | no | Back-reference to the tool's [AI Tutor Card](https://github.com/mizcausevic-dev/ai-tutor-card-spec) declaration when the tool is positioned as a tutor. |

### 4.8 `roles` (conditional)

**Required** when `ai_used` is `true`. **MUST** be a non-empty array of enum values:

| Value | Meaning |
|---|---|
| `brainstorm` | Idea generation; the AI suggested directions or angles. No AI-authored prose retained. |
| `outline` | Structural planning. The AI proposed an outline or scaffolding. |
| `draft` | The AI produced initial draft content the student then edited. |
| `edit` | The AI revised grammar, style, or clarity of student-authored prose. |
| `translate` | The AI translated text between languages. |
| `cite_check` | The AI verified citation formatting or existence. |
| `code_completion` | The AI auto-completed code (IDE-grade or chat-grade). |
| `code_review` | The AI reviewed student-authored code for bugs or improvements. |
| `research_synthesis` | The AI summarized or synthesized sources for the student. |
| `tutor_dialog` | The AI engaged in Socratic dialog or step-by-step tutoring without producing final-form work. |
| `image_generation` | The AI generated visual assets included in the artifact. |
| `data_analysis` | The AI performed exploratory data analysis the student used as input. |
| `other` | Used together with `roles_other_text` to describe a role this taxonomy doesn't cover. |

If `roles` contains `"other"`, then `roles_other_text` (string) **MUST** also be present.

### 4.9 `assistance_extent` (conditional)

**Required** when `ai_used` is `true`. Enum:

| Value | Meaning |
|---|---|
| `minor` | AI contributed peripherally; the artifact is substantially student-authored. |
| `substantial` | AI contributed meaningfully but the artifact is still recognizably student work. |
| `primary_author` | AI is the primary author of the artifact; the student edited / curated / verified. |

### 4.10 `assistance_pct` (optional)

Integer 0–100. The student's good-faith estimate of the AI-originated fraction of the artifact, measured by content volume (words, lines of code, slide content, etc.). **MAY** be omitted when impractical to estimate (e.g. dialog-only tutor use).

### 4.11 `prompt_evidence_mode` (conditional)

**Required** when `ai_used` is `true`. Enum:

| Value | Meaning |
|---|---|
| `full` | Prompts stored as full text in `prompts[].text`. |
| `hashed` | Prompts stored only as canonical SHA-256 hashes in `prompts[].hash`. |
| `omitted` | No prompt evidence preserved. The disclosure declares this consciously; reviewers know prompts are not retrievable. |

### 4.12 `prompts` (conditional)

**Required** when `prompt_evidence_mode` is `"full"` or `"hashed"`. **MUST** be a non-empty array. Each entry:

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | string | yes | Stable identifier within this disclosure, e.g. `"p1"`. |
| `text` | string | conditional | Required when `prompt_evidence_mode` is `"full"`. The literal prompt text. |
| `hash` | string | conditional | Required when `prompt_evidence_mode` is `"hashed"`. Canonical SHA-256, format `"sha256:<64 hex chars>"`. |
| `at` | ISO 8601 | no | Timestamp when the prompt was issued. |
| `tool_index` | integer | no | Index into `tools_used` indicating which tool received this prompt. |

#### 4.12.1 Canonical prompt hash

The canonical hash is the SHA-256 of the prompt text after:
1. Normalizing line endings to `\n` (LF).
2. Stripping a single trailing newline if present.
3. Encoding as UTF-8.

Implementations **MUST** produce identical hashes given identical canonical input.

### 4.13 `artifact_hash` (required)

A canonical SHA-256 of the submitted artifact's bytes, formatted as `"sha256:<64 hex chars>"`. The hash binds the disclosure to a specific submitted file. A grader can independently recompute the hash from the file and compare.

### 4.14 `artifact_uri` (optional)

A URI pointing to the artifact, when one exists (LMS attachment URL, repository URL, etc.).

### 4.15 `aup_uri` (optional but recommended)

URI of the operative [Classroom AI Acceptable Use Policy](https://github.com/mizcausevic-dev/classroom-ai-aup-spec) document at the time of submission. Binds the disclosure to the policy in force.

### 4.16 `policy_compliant` (optional)

The student's good-faith declaration of compliance with the operative AUP.

| Field | Type | Required | Description |
|---|---|---|---|
| `declared` | boolean | yes | Student's declaration. |
| `reason` | string | no | Brief justification, especially when `declared` is `false`. |

### 4.17 `signed_by_student` (required)

Boolean. **MUST** be `true` for a valid disclosure. Indicates the student authored and attested to this disclosure.

### 4.18 `student_signature_at` (required)

ISO 8601 timestamp of student signature.

### 4.19 `teacher_acknowledged` (optional)

A teacher acknowledgment **MAY** be added after the disclosure is signed.

| Field | Type | Required | Description |
|---|---|---|---|
| `acknowledged` | boolean | yes | True if the teacher reviewed the disclosure. |
| `by` | string | yes | Teacher identifier (LMS-side). |
| `at` | ISO 8601 | yes | When the acknowledgment was made. |
| `note` | string | no | Optional free-text note (graded as policy-compliant, follow-up needed, etc.). |

## 5. Conditional rules

### 5.1 No-AI declarations

If `ai_used` is `false`, the disclosure **MUST NOT** include `tools_used`, `roles`, `assistance_extent`, `assistance_pct`, `prompt_evidence_mode`, or `prompts`.

### 5.2 Required-when-AI-used fields

If `ai_used` is `true`, the disclosure **MUST** include `tools_used`, `roles`, `assistance_extent`, and `prompt_evidence_mode`.

### 5.3 Prompts presence rule

If `prompt_evidence_mode` is `"full"` or `"hashed"`, `prompts` **MUST** be present and non-empty.
If `prompt_evidence_mode` is `"omitted"`, `prompts` **MUST NOT** be present.

### 5.4 Hash format

All hash fields (`prompts[].hash`, `artifact_hash`) **MUST** match the pattern `^sha256:[a-f0-9]{64}$`. Hex characters **MUST** be lowercase.

### 5.5 Signature requirement

A disclosure is invalid if `signed_by_student` is `false` or `student_signature_at` is absent.

## 6. Distribution

Disclosures travel alongside the artifact, typically as:

- A sidecar JSON file submitted alongside the artifact (e.g. `essay.docx` + `essay.disclosure.json`).
- A field on the LMS submission record (e.g. Canvas `assignment_submission.ai_disclosure`).
- An embedded comment / metadata block within the artifact when the artifact format supports it (markdown front-matter, code-file header, document properties).

The spec is transport-agnostic. The LMS or grading system is responsible for retrieving the disclosure adjacent to the artifact.

## 7. Compatibility

This spec is the student-side counterpart to:

- [AI Tutor Cards](https://github.com/mizcausevic-dev/ai-tutor-card-spec) — vendor-side disclosure of what an AI tutor does.
- [Classroom AI AUP](https://github.com/mizcausevic-dev/classroom-ai-aup-spec) — district-side disclosure of what AI use is permitted (forthcoming).

The trio (vendor / district / student) forms a closed loop: a tutor declares its behavior, a district declares its policy, a student declares their actual use. A grader can mechanically check that all three line up.

## 8. Security considerations

- **PII minimization.** Implementations **SHOULD** prefer pseudonymous `student.id` values and omit `student.display_name` when possible.
- **Hashed prompts** prevent disclosed prompts from leaking sensitive student context while preserving forensic verifiability.
- **Artifact hash** prevents post-submission edits from invalidating disclosures.
- **Disclosure storage** is the LMS's responsibility. The disclosure itself does not define encryption-at-rest; the LMS's existing student-data protections apply.

## 9. Open questions for v0.2

- **Signing.** v0.1 uses a boolean attestation; v0.2 may add an optional cryptographic signature (BLS / Ed25519) tying the disclosure to a verifiable student key.
- **Multi-author submissions.** v0.1 assumes one student per disclosure; v0.2 may add a `co_authors[]` array for group projects.
- **Inline citation provenance.** v0.2 may add an optional `inline_provenance[]` array linking spans of the artifact to specific tools/roles for finer-grained transparency.

## 10. Versioning

The `disclosure_version` field is a semver identifying the spec revision. v0.1 is a draft; consumers **SHOULD** treat unknown future versions as an error rather than attempting forward-compatible parsing.
