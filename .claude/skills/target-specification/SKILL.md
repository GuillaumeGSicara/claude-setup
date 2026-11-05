---
name: target-specification
description: Turn a sponsor-approved EPIC (output of epic-writer) into a Technical specification: a C4 target architecture plus the sequence diagrams of the flows the EPIC needs, drawn with likec4 via the architecture-diagram skill, and an RFC 2119 requirements table where every row is allocated to an architecture element and carries a verification method. Use whenever someone asks for a target architecture, a technical specification, a system design or RFC 2119 requirements from an EPIC, even if they only hand over the EPIC file.
---

# Target specification

Step "Target architecture" of the delivery flow: EPIC in, Technical specification out. The specification lives next to the EPIC and is written in English for an architect who holds both documents. Every quality control point (QPC) is a gate: the specification is not delivered while one fails, unless the author has explicitly left it blank.

The EPIC states the WHAT with business value and keeps the HOW indicative. The specification makes the HOW binding: a target architecture, the sequences it runs to serve the EPIC's need, and the requirements the system satisfies, each one verifiable. Nothing here changes the WHAT; a requirement that cannot be verified is sent back to the EPIC, not patched here.

The architecture diagram is the main artifact of ticket refinement: every task the team carries out must be readable from it as an element the team builds or a relation it implements. The diagram therefore details what the team builds and only outlines what it consumes.

## Inputs

- The EPIC, in the epic-writer format (WHY / WHAT WE DELIVER / DEPENDENCIES / VALIDATION / APPENDIX). Without one, stop and run `epic-writer` first.
- Sponsor approval status. Ask with AskUserQuestion when the EPIC does not state it; "not yet approved" is recorded at the top of the specification, it does not block the work.
- The codebase and `docs/architecture/` when they exist: the current state the target departs from.

## Procedure

1. **Read inputs.** Open the EPIC and every document it references, then `docs/architecture/*.c4` and `docs/design/adr/` if present. Extract: the Deliverable table, the Requirements table (FR/NFR with priority), the Success criteria table (SC), the DEPENDENCIES rows and the APPENDIX indicative approach. Never ask for something the EPIC already contains.
2. **Preflight.** Run the Step 1 preflight of the `architecture-diagram` skill (`likec4` CLI, `docs/architecture/`). Create `docs/specification/` if absent.
3. **Grill.** Use AskUserQuestion for every question; never ask in free text. One call per theme, in this order; stop asking when every gate below is satisfied or the author answers "leave blank":
   - **Boundary.** Which systems already exist and which ship with this EPIC; which external systems and actors the target talks to (start from DEPENDENCIES and the indicative approach, offer them as options).
   - **Containers.** For each system that changes: runtime units, their technology, how they communicate (REST, gRPC, queue, file, SQL). Offer the APPENDIX approach as the default option. For each element the team builds, ask what it integrates with and how it authenticates (client credentials, API key, service account, token exchange): each answer is a relation on the diagram and a step in a sequence. For each element the team only consumes (an existing SaaS, a dashboard tool, a shared platform), ask nothing about its internal setup: it appears as a boundary with the one relation the team's code or data crosses.
   - **Sequences.** The EPIC answers one need; name the flows the target runs to serve it (one per Deliverable table row is the default offer). For each flow: trigger, outcome the reader sees at the end, elements crossed. At least one flow is required. A flow is worth drawing when its steps cross elements the team builds; a flow that only crosses consumed elements (a reader signing in to an existing tool) is not drawn.
   - **Constraints.** Hosting, data residency, authentication, availability, retention, budget ceilings. Each confirmed constraint becomes an NFR-type row with a number.
   - **Verification.** For each EPIC success criterion with no obvious test: who checks it, with what method, against what pass condition. A criterion nobody can check is the rework signal below.
   - **Decisions.** Where the author hesitates between two designs, ask until one is chosen or the author names the rule that will choose it (a test, a date, an owner); that rule is written into the requirement row. Never pick one silently, and never leave a decision outside the table.
4. **Rework gate.** If an EPIC P1 requirement or success criterion cannot yield one row whose Verification cell passes QPC-R6, stop. Output a one-line note per blocked item: `criterion not testable: <SC-xx> — <why>`, name `epic-writer` as the next step, and do not deliver the specification. Everything else already drafted is kept on disk for the next run.
5. **Write the requirements table** first (QPC-R), then the Introduction.
6. **Draw.** Invoke the `architecture-diagram` skill with its questions pre-answered: audience architect, purpose technical reference, levels C1 + C2 (C3 for any container owning four or more MUST rows), sequence diagrams = the flows from the Sequences grill, highlights = elements carrying a MUST row. Target state only; untouched elements in a neutral colour. Scope of what is drawn:
   - Built elements carry their technology, their icon, and every relation the team implements, including machine-to-machine authentication (client ID and secret to the identity provider, token returned, bearer token on the API call) as separate labelled relations and sequence steps.
   - Consumed elements carry their icon and the one relation the team's code or data crosses; nothing about their own setup (user sign-on, hosting, administration).
   - Human actors appear only as consumers of a deliverable, one actor per distinct deliverable audience. No team, process or log that is not a technical artifact is drawn; a success criterion that needs one is verified in the Requirements table, not in the diagram.
   Export views:

       likec4 export png --flat --seq -o docs/specification/img docs/architecture

7. **Cross-check** traceability both ways (QPC-T), then `likec4 build docs/architecture` until clean; delete `docs/architecture/dist/`.
8. **De-slop.** Run `de-slop-writing` on the prose parts (Introduction, element and view descriptions in the `.c4` files). Apply every finding that does not lengthen the text. Findings needing the author go into one final AskUserQuestion call.
9. **Output** the specification only, at `docs/specification/<epic-slug>.md`, with the likec4 sources in `docs/architecture/`. No preamble, no commentary, no mention of the grill, the inputs or this skill.

## Format

```
# <EPIC title> — Technical specification
## Introduction
## Target architecture
### C1 — System context
### C2 — Containers
### C3 — <container>          (only when drawn)
### Sequence — <flow>         (one per flow)
## Requirements
```

Three parts, nothing else: the Introduction, the diagrams, the Requirements table. No text floats outside them: no captions, no notes, no lists, no section other than the headings above. Everything a reader needs about an element or a flow is inside the image, set in the `.c4` files.

Header line under the title: `Source EPIC: <path or link> · Sponsor approval: <approved on YYYY-MM-DD | not yet approved>`.

### Introduction

Max 500 characters, no bullets. Three lines separated by blank lines: the systems that change and the ones that stay; the deliverables the architecture serves (copied names from the EPIC Deliverable table); what is out of scope (copied from the EPIC callout, one sentence).

### Target architecture

One exported image under each sub-heading, in the order of the format: C1, C2, every C3 drawn, then one sequence per flow from the grill. The sub-heading is the only text; the view's `title` and `description` in `views.c4` carry the question the view answers and, for a sequence, the trigger and the outcome. Elements the team builds or changes are tagged `#new` or `#changed` in the model and highlighted; untouched ones are tagged `#existing` and drawn in a neutral colour; kind, technology, icon and a one-sentence description of what the element is (not what it does relative to others) are set in the `.c4` files.

### Requirements

Open with this sentence, always the same: "Requirement keywords follow [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)."

Then the **Requirements table**, grouped by Element, MUST rows first:

| ID | Requirement | Element | Traces to | Verification | Pass condition |
|---|---|---|---|---|---|

- ID is SR-01, SR-02, ... in table order.
- Requirement is one sentence: `<Element> <KEYWORD> <observable behaviour>`. The keyword is in capitals and appears once. No "and" joining two behaviours; split the row.
- Keyword follows the EPIC priority: P1 → MUST or MUST NOT; P2 → SHOULD or SHOULD NOT; P3 → MAY. Every EPIC NFR becomes a MUST with its number and unit in the sentence. A constraint from the grill becomes a MUST with its number.
- Element is the likec4 element name, as written in the `.c4` file.
- Traces to lists the EPIC IDs the row serves: FR-xx, NFR-xx, SC-xx, or a DEPENDENCIES row. Never empty.
- Verification is one of `Test`, `Demonstration`, `Inspection`, `Analysis`, followed by who runs it.
- Pass condition is binary (yes/no) or a number with unit, checkable without the author present.

- A choice the grill left open is carried by the row it affects: the sentence names the options and the rule that selects one ("scoring MUST use the metric, cosine of embeddings or ROUGE-L, that reaches ≥ 19 of 20 on the hand-labelled pairs"). No row reads "to be decided".

Max 30 rows. Above 30, ask with AskUserQuestion whether to split the specification by system; do not overflow. The table closes the document.

## QPC

Global
- S1 Self-supporting with the EPIC: no reference to the creation process, the conversation or "as discussed". Terms come from the EPIC; a technology or element name the EPIC does not define is explained in passing where it first appears.
- S2 The WHAT is untouched: no deliverable added, removed or renamed relative to the EPIC. A gap found here is a rework note to `epic-writer`, not an edit.
- S3 Target state only: every diagram and every row describes the system once the EPIC ships. The current state appears only through `existing` elements.

Requirements (QPC-R)
- R1 Every row carries exactly one RFC 2119 keyword, in capitals; no lowercase "must", "should" or "may" anywhere else in the document.
- R2 Keyword matches the EPIC priority of the row's Traces to; a row stricter than its source is a new constraint and needs an author answer recorded in the grill.
- R3 Every row names one Element drawn in a structural view of the diagram.
- R4 Every row traces to at least one EPIC ID; every EPIC P1 requirement, and every success criterion proving one, is traced by at least one MUST row; every other success criterion is traced by at least one row of the keyword its priority maps to.
- R5 One behaviour per row; a row with "and", "or", "etc." between behaviours is split.
- R6 Verification names a method and a runner, and Pass condition is binary or numeric. A row failing this gate is not reworded here: it triggers the rework gate.
- R7 Every number in a Pass condition appears in the Requirement sentence or in the EPIC; none is invented.

Traceability (QPC-T)
- T1 Every element tagged `#new` or `#changed` owns at least one row; an element with no row is removed from the diagram.
- T2 Every DEPENDENCIES row of the EPIC maps to an external actor or system in C1.
- T3 Every interaction drawn in C2 is covered by a row on at least one of its two ends.

Diagram (QPC-D)
- D1 `likec4 build docs/architecture` is clean.
- D2 Each view holds at most 12 nodes; one question per view; labelled relationships naming the interaction type (REST, gRPC, Kafka topic, S3 read), never the implementation detail.
- D3 Every element in the `.c4` files carries a one-sentence description stating what it is, and a `technology` when it is built or changed.
- D4 The `docs/architecture/` folder stays self-contained: no reference to the specification, the EPIC or any file outside it.
- D5 Every box carries a logo when one exists (`icon tech:…`, `aws:…`, `gcp:…`, `azure:…`); a bare box is justified in one line.
- D6 At least one sequence diagram exists, and every Deliverable table row of the EPIC is served by at least one of them; every element a sequence step names is drawn in a structural view.
- D7 Every element tagged `#new` or `#changed` shows its technology, its icon and every relation the team implements; an integration named in the grill (external API, identity provider, model gateway, queue) that is missing from the diagram fails the gate.
- D8 No relation or element describes the setup of a consumed system: no user sign-on to an existing tool, no hosting or administration of it. The only relations touching a consumed element are the ones the team's code or data crosses.
- D9 Every element and relation on the diagram maps to at least one task the team will carry out, or is an `existing` boundary the task reaches; a reader can list the refinement tickets from the diagram alone.

## Hand-off

- **Next step:** the Verification and Pass condition columns are the verifiable criteria that slice grouping consumes.
- **Rework:** `criterion not testable` goes back to `epic-writer` with the one-line notes from the rework gate. The specification is not delivered until the EPIC returns with every blocked criterion fixed.

## Guards

- Never invent a technology, a number, an owner or an element. Blank beats guessed.
- Keep the author's element names and the EPIC's terms.
- The architect edits the output: deliver the specification, not an explanation of it.
- Make the document breathe: one blank line between a heading, its image or its table, and the next heading.
