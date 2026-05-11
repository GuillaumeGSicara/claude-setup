---
name: architecture-diagram
description: "Generate or update a C4 architecture diagram for the project. Use this skill when the user wants to create, update, or regenerate architecture documentation."
disable-model-invocation: false
---

# Architecture Diagram Skill

Creates or updates a C4-model architecture diagram in `docs/architecture/`,
generated with the `likec4` library.

The C4 model has four levels: C1 (System Context), C2 (Container),
C3 (Component), C4 (Code). Most projects need C1 + C2, sometimes C3.

---

## Workflow

1. **Preflight** — verify environment
2. **Handle existing diagram** (if present)
3. **Gather requirements**
4. **Generate files**
5. **Validate**

---

## Step 1 — Preflight

- Confirm the working directory is a git repo root.
- Check `likec4` CLI (`likec4 --version`). If missing, stop and tell the
  user to install it (`brew install likec4` or `npm i -g likec4`).
- Check the `likec4` MCP server is available. If missing, stop and point
  to https://likec4.dev/tooling/ai-tools/.
- Create `docs/architecture/` if absent.

## Step 2 — Handle existing diagram

If `docs/architecture/` already contains `.c4` files:

1. Read them and summarize: systems, containers, main views.
2. Show the summary and ask the user to pick one:
   - **Overwrite** — replace in place
   - **Version it** — rename current folder to `docs/architecture.bak/`
   - **Abort**

## Step 3 — Gather requirements

Ask in **one batched prompt**:

- **Audience** — new devs / stakeholders / devops / security / other
  (multi-select; each gets a tailored view)
- **Purpose** — onboarding / technical reference / security review / other
- **C4 levels needed** — C1 / C1+C2 / C1+C2+C3 / include C4
- **Highlights** — components, flows, or boundaries to emphasize
- **Custom abstractions** — anything non-standard that should appear

## Step 4 — Generate files

File layout in `docs/architecture/`:

    _spec.c4        # styles, element kinds, relationship kinds
    views.c4        # structural views + audience views
    <system>.c4     # one file per system
    icons/          # custom icons

**`views.c4` contains two kinds of views:**

1. **Structural views** — one per requested C4 level. Canonical model views.

2. **Audience views** — one per selected audience, scoped to what that
   reader needs:

   - **New devs** — C2 with containers labeled by repo/language; local
     dev entry points visible.
   - **Stakeholders** — C1 with business capabilities highlighted;
     infrastructure collapsed.
   - **Devops** — C2 emphasizing deployment boundaries, runtimes, scaling
     units, external dependencies.
   - **Security** — trust boundaries, authn/authz flows, data
     classifications, ingress/egress highlighted; internal links dimmed.
   - **Other** — ask what this audience needs to see.

   Name views explicitly (`view onboarding_devs`, `view security_review`)
   and give each a one-line `description` stating who it's for and what
   question it answers.

**Conventions:**

- Component descriptions state **what the component is**, not what it
  does relative to others.
- Relationship labels describe the **interaction type** (REST, gRPC,
  Kafka topic, S3 read), not implementation details.
- Multiple components per layer is fine; clarity beats hierarchy purity.

## Step 5 — Validate

Run:

    likec4 build docs/architecture

Fix any errors and re-run until clean. Then tell the user they can
inspect locally with `likec4 start docs/architecture`.

---

## Notes

- Custom icons live in `docs/architecture/icons/` and are referenced by
  relative path.
- Keep `_spec.c4` minimal at first — add styles as the diagram grows.
- For multi-system projects, one `.c4` file per system keeps diffs small.