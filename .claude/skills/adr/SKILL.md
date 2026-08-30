---
name: adr
description: Use when the user asks to write, create, or update an Architecture Decision Record (ADR) or a design decision document under docs/design — e.g. "document this decision", "write an ADR for X", "record why we chose Y". Produces a long-lived record in docs/design/adr/, written in Simplified Technical English, that separates the specific problem from the durable forces so the record still makes sense after the codebase moves on.
---

# Architecture Decision Records (ADR)

An ADR records a decision so a future reader — who does not have today's Slack
thread or ticket — can understand it. Most ADRs rot because they only explain
today's problem. This skill produces ADRs that also state the forces that will
still be true after the triggering problem is gone, so the record stays
useful through many codebase evolutions.

## When to use this skill

Use it for decisions with real consequences: choice of architecture,
technology, data model, integration pattern, or a reversal of a previous
decision. Do not write an ADR for a routine implementation detail with no
lasting trade-off.

## Workflow

1. **Find the next number.** List `docs/design/adr/*.md`. Numbers are
   4-digit, zero-padded, sequential (`0001`, `0002`, ...). If the directory
   does not exist, create it and start at `0001`.

2. **Gather the decision before writing.** Do not invent context. If any of
   these are missing or unclear, ask the user (do not guess):
   - The status: `Proposed`, `Accepted`, `Rejected`, `Deprecated`, or
     `Superseded by ADR-NNNN`.
   - The actual decision, in one sentence.
   - At least one alternative that was considered and rejected, and why.

3. **Separate the trigger from the durable forces.** This is the part most
   ADR writers skip, and the reason most ADRs go stale. Before filling the
   template, explicitly work out two different things:
   - **Problem** — the specific situation that forced a decision *now*. This
     will likely stop being true: the code will be refactored, the
     constraint may disappear, the ticket will close.
     Keep this section short (one paragraph, two or three sentences).
   - **Durable context** — the forces that exist independently of the
     trigger, and that a *different* problem would run into just the same:
     quality attributes (performance, security, compliance, cost), other
     components or teams that depend on this area, domain rules, and
     invariants that must hold. This section is what keeps the ADR readable
     once the original problem is forgotten. If you find yourself unable to
     name any durable force, ask the user what the surrounding system
     constraints are before writing this section — do not leave it as a
     restatement of the problem.

4. **Fill the template** at `templates/adr-template.md` (read it before
   writing — do not reconstruct it from memory). Copy it to
   `docs/design/adr/NNNN-<kebab-case-title>.md` and complete every section.
   Do not leave a section blank; write "None identified" rather than
   deleting a section, so future readers know it was considered.

5. **Rewrite the prose in Simplified Technical English (STE).** ADRs are read
   by people under time pressure, months or years later, sometimes not
   native English speakers. Load
   `references/ste-writing-rules.md` and apply it as an editing pass over
   every prose paragraph (tables and code are exempt from sentence-length
   rules, but must still use consistent terms). Do this as a distinct pass
   after drafting content — do not try to compose STE-compliant prose and
   figure out the decision at the same time.

6. **Run the closing check** before presenting the file to the user:
   - Every prose sentence follows the STE rules in the reference file.
   - The Problem section and the Durable Context section do not say the same
     thing twice — Durable Context must survive if the Problem section were
     deleted.
   - The Decision is one sentence, active voice, and testable (a reader can
     tell whether the codebase still complies with it).
   - Consequences include at least one negative or risk — a decision with no
     downside was not evaluated honestly.

7. If an `docs/design/adr/README.md` or index file exists, add a row linking
   the new ADR (number, title, status). If none exists, do not create one
   unless the user asks.

## Updating or superseding an ADR

Never edit an `Accepted` ADR's Decision or Context to match new reality.
Write a new ADR that supersedes it: set the new ADR's `Supersedes` field, and
edit only the old ADR's `Status` line to `Superseded by ADR-NNNN`. The old
ADR's history must stay intact — it is a record of what was true, not a
living document.
