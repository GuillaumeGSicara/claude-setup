---
name: epic-writer
description: Grill the author with AskUserQuestion, then write a two-page EPIC in the WHY / WHAT WE DELIVER / DEPENDENCIES / VALIDATION / APPENDIX format, de-slopped with de-slop-writing before delivery. Use whenever someone asks to write, draft or structure an EPIC, a feature brief or an initiative one-pager, even if they only give a feature name.
---
 
# EPIC writer
 
The EPIC is written in English, in the format below, and must be readable by someone outside the team who holds nothing but the document. Every quality control point (QPC) is a gate: the EPIC is not delivered while one fails, unless the author has explicitly left it blank.
 
## Procedure
 
1. **Read inputs.** Open every attached doc, ticket, diagram or link before asking anything. Never ask for something the inputs already contain.
2. **Target surface.** Ask with AskUserQuestion where the EPIC will live (Confluence, Notion, Jira, GitHub/Markdown, Google Docs, Other). The answer sets the syntax for diagrams (Mermaid where rendered, otherwise an image placeholder with a textual spec the author can draw from), the callout block, and tables.
3. **Grill.** Use AskUserQuestion for every question; never ask in free text. There is no question budget: keep asking until every gate below is satisfied or the author answers "leave blank". Group questions by section (one AskUserQuestion call per section), ask the WHY section last. Offer concrete options where they exist (priority, type of gain, check moment); the author can always type "Other". Record "unknown" once as a blank; do not ask it again. For WHAT, ask first what ships, who acts on it, what they can do the day it ships, and what that is worth in a unit from the WHY list; a deliverable the author cannot value is challenged (name the value, descope it, or drop it), not recorded. For each candidate deliverable, ask who outside the delivery team opens, reads or runs it; when the answer is "only the team", it is a means, not a deliverable: record it for the APPENDIX indicative approach and ask again what it produces for the beneficiary.
4. **Write** the EPIC. Fix WHAT first, starting with the Deliverable table, then VALIDATION, then DEPENDENCIES, then WHY, then APPENDIX (indicative approach first). Everything the grill learned about the HOW is written once, in the APPENDIX.
5. **De-slop.** Run the `de-slop-writing` skill on the full EPIC. Apply every finding that does not lengthen the text. Findings that need the author (W14 case, W08 source, W24 relation) go into one final AskUserQuestion call; after it, output.
6. **Output** the EPIC only, in the surface's syntax. No preamble, no commentary, no mention of the grill, the inputs or this skill (QPC-S2).

## Global QPC
 
- S1 Target size is two pages. The skill enforces it as a budget: prose under 600 words, no table over 10 rows (unless in APPENDIX). Cut WHAT captions, then APPENDIX, before anything else.
- S2 Self-supporting: no reference to the creation process, the conversation, the author's inputs or "as discussed". An external reader understands every term from the text or the glossary.
- S3 Priority scale, used in WHAT and VALIDATION:
  - P1 on the critical path, no applicable countermeasure (workaround or fallback)
  - P2 on the critical path with a countermeasure, or off the critical path with none
  - P3 off the critical path, countermeasure exists
The grill establishes, for each requirement, whether it is on the critical path and whether a countermeasure exists.

## Section-specific QPC

- QPC-WHY: The WHY section must clearly state the problem, the opportunity, or the motivation behind the EPIC. It should be concise and compelling.
- QPC-WHAT: The WHAT WE DELIVER section opens with the Deliverable table: the concrete artefacts that ship, who acts on them, the action they enable, and the business value each one carries. It then lists requirements with their priority (P1, P2, P3). A deliverable nobody can act on, or whose business value cannot be named, does not pass. WHAT states the final WHAT, the one with business value; the HOW (pipelines, jobs, tables, schemas, services, components, architecture) stays indicative and lives in the APPENDIX, never in the core of the document.
- QPC-DEPENDENCIES: The DEPENDENCIES section must enumerate all external and internal dependencies, including teams, systems, and third-party services.
- QPC-VALIDATION: The VALIDATION section must describe how success will be measured, including metrics, tests, and review processes.
- QPC-APPENDIX: The APPENDIX section should contain supplementary information such as diagrams, tables, and glossary terms, but must not exceed the global size constraints.

## Format
 
```
# <Title>  (verb + object, under 10 words)
## WHY
## WHAT WE DELIVER
## DEPENDENCIES
## VALIDATION
## APPENDIX
```
 
### WHY
 
Max 700 characters, no bullets. Four key points, each on its own line separated by a blank line, in this order: the value proposition; the expected gain as a number with unit and baseline (delivery gain in days or lead time, productivity gain in hours, tickets or incidents, business value in euros, users or conversion); the beneficiary; one sentence answering "Why now?". One or two sentences per line, never a single block paragraph.
 
QPC
- W1 Under 700 characters.
- W0 Four lines, one per key point (value, gain, beneficiary, why now), separated by blank lines.
- W2 One quantified gain with unit and baseline, or "no baseline" written explicitly.
- W3 Beneficiary: a named role or population, plus the sponsor as function (name optional).
- W4 A "Why now?" sentence: trigger, deadline or cost of waiting.

### WHAT WE DELIVER
 
Visuals, tables and captions only. The visuals represent the target state the EPIC achieves, not the current one.

Open with the **Deliverable table**: what ships, who acts on it, and what it is worth.

| Deliverable | Used by | Enables | Business value |
|---|---|---|---|

- Deliverable is a shippable artefact the beneficiary can open, call, run or read: a screen, an API, a report, a number, a contract, a process. Never an activity ("improve", "study", "migrate"), and never a means only the team touches (a pipeline, an ingestion job, an intermediate table, a schema, a library): those go to the APPENDIX indicative approach.
- Used by is a named role or system.
- Enables starts with a verb and names the action the user takes the day the deliverable ships ("Approve a refund without leaving the ticket").
- Business value is a gain with a unit from the WHY list (days, hours, tickets, incidents, euros, users, conversion), or names the WHY gain this deliverable contributes to.
- Max 5 rows. One row per artefact; split a row that bundles two artefacts.

Then include, when applicable:
1. A visual model if the feature has a user-facing or flow component (flowchart, sequence, wireframe), drawn from the beneficiary's side: what they see, receive or decide, not the components behind it.
2. A before/after for every existing artefact the beneficiary uses that changes (API, screen, contract, process).
The data schema, the processing flow and the architecture are HOW: they go to the APPENDIX indicative approach, unless the data itself is what the beneficiary reads (then a field table of that output, nothing upstream).
Then the **Requirements table** (the brief calls it RFC table):
 
| ID | Type | Requirement | Priority |
|---|---|---|---|
 
- Type is FR or NFR; IDs are FR-01, NFR-01, ...
- Every NFR carries a measurable target.
- Requirements state outcomes the beneficiary can observe ("every X receives Y", "Z is delivered with its period"), not mechanisms ("ingest via API Z", "compute score S"). A mechanism the author insists on is a constraint: write it as an NFR with its target, or move it to the indicative approach.
- Max 10 rows. Above 10, keep P1 only here and move P2/P3 to the APPENDIX. If there are more than 10 P1 rows, stop and ask with AskUserQuestion whether to split the EPIC; do not overflow P1 rows.
Then the **out-of-scope callout**, in the surface's callout syntax: one sentence per excluded item, built from what the grill ruled out. The callout is the only place in this section where a list is allowed.
 
QPC
- D0 Deliverable table present; every Deliverable cell names an artefact, not an activity, and every Enables cell starts with a verb and describes an action someone can take once it ships.
- D0.1 Every row carries a business value with a unit, or traces to the WHY gain. A row with no nameable business value fails the gate: ask the author with AskUserQuestion to name it, move it to the out-of-scope callout, or drop it. This gate has no "leave blank" exit.
- D0.2 Every P1 requirement belongs to one row of the Deliverable table.
- D1 No bullet list outside the callout.
- D2 Visual model present if there is a visual component.
- D3 Output field table present when the beneficiary reads the data itself; any other schema sits in the APPENDIX.
- D4 Before/after present for every changed artefact the beneficiary uses.
- D5 No caption over 250 characters.
- D6 Every diagram passes the visual check below.
- D7 Every P1 requirement is linked to a success criterion in VALIDATION.
- D8 No HOW in WHAT: no pipeline, job, intermediate table, schema, service, library or architecture element appears in the Deliverable table, the visual model or the Requirements table unless the beneficiary in Used by opens or runs it. Hits are moved to the APPENDIX indicative approach, not deleted.
Visual check (until a `de-slop-visual` skill exists): one idea per diagram; every box carries a name the reader owns; labelled arrows; at most 12 nodes; no decorative icons, gradients or colour without meaning; readable at phone width.
 
### DEPENDENCIES
 
| Item | Dependency | Owner | Action | Needed by |
|---|---|---|---|---|
 
- Dependency reads "Need X from Y". When Y is unknown, write "Need X from (owner to identify)" and Action is "Identify owner".
- Owner, Action and Needed by may stay blank. Never fill a blank with "TBD" prose.
QPC
- P1 Every row follows "Need X from Y".
- P2 Every filled Action starts with an action verb (Confirm, Deliver, Open, Provision, Sign, Identify).

### VALIDATION
 
First a bullet list of validation actions, then the **Success criteria table**:
 
| ID | Requirement | Criterion | Threshold | Checked when | Checked by |
|---|---|---|---|---|---|
 
- Requirement is the FR/NFR ID the criterion proves.
- Threshold is binary (yes/no) or a number with unit.
- Checked when is `validation` or `rollout`. A `rollout` row names who returns to check it.
QPC
- V1 Every validation bullet names a team role and starts with a verb.
- V2 Every criterion has a binary or numeric threshold checkable by a team member.
- V3 Every P1 requirement appears in the Requirement column at least once.

### APPENDIX
 
- Indicative approach, first: the HOW the team has in mind, marked non-binding. Opens with one sentence saying the team may change it as long as the deliverables and the success criteria hold. Holds the processing flow or architecture diagram, the data schema, and a two-column `| Step | Intended approach |` table. Every element moved out of WHAT by D8 lands here.
- External references: documents, tickets, prior EPICs, data sources, each with a link or exact title. Nothing from the conversation.
- Requirements P2/P3 moved from WHAT, same columns.
- Glossary, two columns `| Term | Definition |`, for every acronym and team-owned term the text keeps. Terms not in the glossary are rewritten in plain words (de-slop W18).

## Guards
 
- Never invent a number, an owner, a source, a term or a diagram element. Blank beats guessed.
- Keep the author's terms and the author's superlatives when true (de-slop W27).
- The author edits the output: deliver the EPIC, not an explanation of it.
- Make the final document breathe: ensure line return, bold headings, proper spacing, the final document is intented for human read as well as agent, make sure to reduce the human mental load when reading the doc.