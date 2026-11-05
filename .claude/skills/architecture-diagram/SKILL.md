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
- Ask with AskUserQuestion whether the user wants **sequence diagrams**.
  If yes, before writing any file, pin down what each one illustrates:
  the target architecture handles one or more flows, and each flow the
  user names becomes one `dynamic view`. For each flow record, in one
  line: the trigger (who or what starts it), the outcome the reader must
  see at the end, and the elements it crosses. A flow the user cannot
  describe in that one line is not drawn.

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

**`views.c4` contains three kinds of views:**

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

3. **Sequence diagrams** — one `dynamic view` per flow pinned down in
   Step 1, with `variant sequence`. Steps carry the interaction type as
   `technology`; return arrows use `<-`. Name the view after the flow
   (`view seq_checkout`) and give it a `description` stating the trigger
   and the outcome from Step 1. Put them in `views.c4` after the audience
   views. Export them with `likec4 export png --seq`; without `--seq`
   a dynamic view is rendered as a flow, not as lifelines.

**Conventions:**

- Every box carries a logo when one exists: set `icon` on each element.
  Two sources, in this order:
  1. **Bundled packs** shipped with likec4: `tech:` for languages,
     frameworks, databases and tools, `aws:`, `gcp:`, `azure:` for
     hyperscaler services. An element that uses Python gets
     `icon tech:python`, a PostgreSQL store `icon tech:postgresql`.
  2. **This skill's `public/` folder** (next to this SKILL.md), an
     extension of the bundle for logos it lacks: snowflake, litellm,
     langfuse, qdrant, openai, anthropic, aws-bedrock, aws-s3, fast-api,
     java-spring, go, rust, angular, vite, clickhouse, linkup,
     pingidentity, rest-api. Snowflake,
     for example, is not in `tech:`; use `public/snowflake.svg`. Copy the
     asset to `docs/architecture/icons/` and reference it by relative
     path (`icon ./icons/snowflake.svg`). List the folder before deciding
     a logo is missing; it grows over time.
  A bare box is allowed only when neither source has a match; add the
  logo to `public/` when you find one so the next diagram has it.

- Component descriptions state **what the component is**, not what it
  does relative to others.
- Relationship labels describe the **interaction type** (REST, gRPC,
  Kafka topic, S3 read), not implementation details.
- Colors should largely echo the key components they represent. Use
  recognizable hyperscaler colors when relevant, such as AWS yellow, Google
  blue, and Azure blue. Use a technology's characteristic colors when those
  colors are strongly associated with it. Use neutral colors when no clear
  component or technology color exists.
- Do not describe particular tooling that pertains to the developer environment or local setup.
- Multiple components per layer is fine; clarity beats hierarchy purity.

## Step 5 — Validate

Run:

    likec4 build docs/architecture

Fix any errors and re-run until clean, then delete the generated
`docs/architecture/dist/` folder (build output, not source). Then check:

- every element has an `icon`, or a one-line reason in the reply why no
  logo matches;
- every flow pinned down in Step 1 has its `dynamic view`, and every
  element a step names appears in a structural view.

Then tell the user they can inspect locally with
`likec4 start docs/architecture`.

---

## Notes

- Created artifacts must be self-supporting. Do not reference components,
  files, or assets outside `docs/architecture/`; copy required resources into
  that folder and use relative paths. Do not reference files outside of the likeC4 architecture directory, especially design records or other implementation or instructions from the repo or the rest of the context, as this breaks the self-contained nature of the architecture documentation.
- Icons from this skill's `public/` folder are never referenced in place:
  copy the asset to `docs/architecture/icons/` first, so the diagram stays
  self-contained.
- Custom icons live in `docs/architecture/icons/` and are referenced by
  relative path.
- Keep `_spec.c4` minimal at first — add styles as the diagram grows.
- For multi-system projects, one `.c4` file per system keeps diffs small.