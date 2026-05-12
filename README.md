# Student AI Disclosure

A draft specification for **Student AI Disclosure** — machine-readable declarations attached to student-submitted work that state how AI was used in producing it.

A teacher reviewing a submission, an LMS attaching the disclosure as metadata, or an academic-integrity office auditing patterns can all read one document format. The student gets a defensible, structured record of what they did and didn't do.

This spec is the **student-side** counterpart to [AI Tutor Cards](https://github.com/mizcausevic-dev/ai-tutor-card-spec) (vendor-side) and the [Classroom AI AUP](https://github.com/mizcausevic-dev/classroom-ai-aup-spec) spec (district-side). The three together form a closed loop: the vendor declares what its tutor does, the district declares what AI use it permits, and the student declares their actual use on each artifact.

## The four pillars

| Pillar | What it does |
|---|---|
| **AI usage facts** | `ai_used` boolean, `tools_used[]` (with optional back-refs to Agent Cards / Tutor Cards), `roles[]` from a 13-value taxonomy, `assistance_extent`, optional `assistance_pct` |
| **Prompt evidence** | Three modes — `full` (literal text), `hashed` (canonical SHA-256 only, privacy-preserving), `omitted` (consciously declared as not retained) |
| **Artifact binding** | Canonical SHA-256 (`artifact_hash`) ties the disclosure to a specific submitted file. Post-submission edits become detectable. |
| **Acknowledgment** | Student signature is required; an optional `teacher_acknowledged` block records the grader's review |

## Design principles

1. **Disclosure is the lawful state.** A disclosure with `ai_used: false` is valid. A disclosure with `ai_used: true` and a list of roles is valid. *No* disclosure attached to an assignment that requires one is the failure mode — not "AI was used." This spec punishes omission, not use.
2. **Bind disclosure to artifact.** SHA-256 over the submission so the disclosure cannot drift away from the work it describes.
3. **Privacy-preserving prompt evidence.** Hash mode lets students disclose *that* prompts were used without exposing sensitive context.

## Quickstart

1. Generate a `disclosure_id` (UUID v4 or any unique string).
2. Compute `artifact_hash` as canonical SHA-256 over the submitted file's bytes.
3. Author a disclosure conforming to [`student-ai-disclosure.schema.json`](student-ai-disclosure.schema.json). Start from one of the [examples](examples/).
4. Validate with any JSON Schema 2020-12 validator (e.g. `ajv`, `jsonschema`):
   ```bash
   npx -p ajv-cli -p ajv-formats ajv validate \
     -s student-ai-disclosure.schema.json \
     -d examples/no-ai-essay.json \
     -c ajv-formats --spec=draft2020 --strict=false
   ```
5. Submit the disclosure alongside the artifact (sidecar JSON file, LMS metadata field, or embedded comment block).

## The role taxonomy

| Value | Meaning |
|---|---|
| `brainstorm` | Idea generation; the AI suggested directions. No AI-authored prose retained. |
| `outline` | Structural planning; the AI proposed an outline. |
| `draft` | The AI produced initial draft content the student then edited. |
| `edit` | The AI revised grammar, style, or clarity of student-authored prose. |
| `translate` | Language translation. |
| `cite_check` | Citation formatting / existence verification. |
| `code_completion` | IDE-grade or chat-grade code completion. |
| `code_review` | The AI reviewed student-authored code. |
| `research_synthesis` | Summarization or synthesis of sources. |
| `tutor_dialog` | Socratic / step-by-step tutoring; no final-form work produced. |
| `image_generation` | Visual assets included in the artifact. |
| `data_analysis` | Exploratory data analysis used as input. |
| `other` | Used with `roles_other_text` for roles this taxonomy doesn't cover. |

## Files in this repo

- [`SPEC.md`](SPEC.md) — full v0.1 specification
- [`student-ai-disclosure.schema.json`](student-ai-disclosure.schema.json) — JSON Schema (draft 2020-12), with conditional rules for `ai_used`, `prompt_evidence_mode`, and `other` role
- [`examples/`](examples/) — reference disclosures:
  - [`no-ai-essay.json`](examples/no-ai-essay.json) — student declares no AI use
  - [`ai-edit-hashed-prompts.json`](examples/ai-edit-hashed-prompts.json) — minor AI assist with hashed prompts (privacy-preserving)
  - [`ai-primary-author-full-prompts.json`](examples/ai-primary-author-full-prompts.json) — AI-led code generation with full prompt history disclosed

## Status

**v0.1 draft.** Issues and pull requests welcome.

## License

AGPL-3.0. Specification text is freely implementable; schema and examples are AGPL-3.0.

## Kinetic Gain Protocol Suite

A family of open specifications for the answer-engine and agent era:

| Spec | What it does |
|---|---|
| [AEO Protocol](https://github.com/mizcausevic-dev/aeo-protocol-spec) | Entity declaration at `/.well-known/aeo.json` |
| [Prompt Provenance](https://github.com/mizcausevic-dev/prompt-provenance-spec) | Versioned, lineaged, reviewable LLM prompt records |
| [Agent Cards](https://github.com/mizcausevic-dev/agent-cards-spec) | Declarative agent capability + refusal disclosure |
| [AI Evidence Format](https://github.com/mizcausevic-dev/ai-evidence-format-spec) | Structured citations for LLM-generated claims |
| [MCP Tool Cards](https://github.com/mizcausevic-dev/mcp-tool-card-spec) | Per-tool disclosure for Model Context Protocol servers |
| [AI Tutor Cards](https://github.com/mizcausevic-dev/ai-tutor-card-spec) | EdTech-specialized agent disclosure (vendor-side) |
| **Student AI Disclosure** (this) | Student-side disclosure attached to submitted work |
| [Classroom AI AUP](https://github.com/mizcausevic-dev/classroom-ai-aup-spec) | District / school / course AI policy (third leg of the EdTech trio) |

---

**Connect:** [LinkedIn](https://www.linkedin.com/in/mirzacausevic/) · [Kinetic Gain](https://kineticgain.com) · [Medium](https://medium.com/@mizcausevic/) · [Skills](https://mizcausevic.com/skills/)
