---
name: update-repo
description: Sincroniza README.md y glosario.md con todos los skills registrados en el ecosistema
license: Apache-2.0
metadata:
  author: transportationamerica-setup
  version: "1.0"
scope:
  - documentation
  - readme
  - glosario
  - skills
permissions:
  allow:
    - filesystem
  deny:
    - database
---

# Update Repository v1

You are a documentation synchronization expert. When this skill is invoked, **execute the full repo sync flow** to keep `glosario.json`, `glosario.md`, and `README.md` consistent with every registered skill.

---

## Flow to execute

### Step 1 — Inspect and sync descriptions in glosario.json

For each skill in `glosario.json`, read its source file at `src/skills/{name}/skill.json`.

Compare the `description` field in `skill.json` against the `description` stored in `glosario.json`:
- If they differ → update `glosario.json` with the value from `skill.json` (source of truth)
- If equal → no change needed

After inspecting all skills, save `glosario.json` if any description was updated.

### Step 2 — Regenerate glosario.md

Rewrite `glosario.md` from scratch using the (now up-to-date) `glosario.json`. Use this exact structure:

```markdown
# Glosario de Skills

## {displayName}

{description}

| Versión | Ubicación |
|---------|-----------|
| `{version}` | [src/skills/{name}/{version}/usage.md](src/skills/{name}/{version}/usage.md) |
```

Rules:
- One `##` section per skill
- One table row per version
- The link must point to the real `usage.md` path relative to the repo root
- Regenerate fully — do not preserve old content

### Step 3 — Update README.md skills table

Find the **Skills disponibles** section in `README.md` and replace only the skills table with:

```markdown
| Skill | Descripción |
|-------|-------------|
| `{name}-{version}` | {description} |
```

- One row per skill+version combination
- Do not modify any other section of README.md

### Step 4 — Update README.md project structure tree

Find the **Estructura del repositorio** section in `README.md` and replace only the fenced code block (` ``` `) that contains the directory tree. Regenerate it by scanning the actual `src/skills/` directory.

The tree must follow this exact format:

```
standars-repo/
│
├── glosario.json                        ← índice interno de skills y versiones (fuente de verdad)
├── glosario.md                          ← índice legible con links a cada skill
│
└── src/
    │
    ├── types/
    │   └── index.ts                     ← interfaces TypeScript (Glosario, SkillMeta, VersionMeta…)
    │
    ├── engine/
    │   ├── skill-reader.ts              ← lectura de skills, versiones y glosario
    │   ├── skill-writer.ts              ← escritura de skills, versiones y glosario (.json + .md)
    │   └── permission-resolver.ts       ← resolución de permisos por versión
    │
    ├── server/
    │   ├── index.ts                     ← servidor HTTP MCP (Streamable HTTP transport, puerto 3000)
    │   └── tools/
    │       ├── list-skills.ts           ← tool: list_skills
    │       ├── get-skill-usage.ts       ← tool: get_skill_usage
    │       ├── create-skill.ts          ← tool: create_skill
    │       ├── add-version.ts           ← tool: add_version
    │       └── edit-skill.ts           ← tool: edit_skill
    │
    └── skills/
        {dynamic skill entries}
```

For the `skills/` subtree, generate one entry per skill found in `src/skills/`. For each skill, list:
- `skill.json` at the skill root
- Each version subfolder, each containing `skill.json` and `usage.md`

Use proper tree characters (`├──`, `└──`, `│`) and align annotations with `←` as in the existing format.

Rules:
- Only replace the content inside the fenced code block — do not touch the section heading, the prose around it, or the **Roles de cada capa** table below it
- The fixed top-level entries (`glosario.json`, `glosario.md`, `src/types/`, `src/engine/`, `src/server/`) must always be present and unchanged
- Derive the dynamic `skills/` subtree from actual filesystem state, not from `glosario.json`

### Step 5 — Report to the user

After completing all steps, report:
- How many skills and versions were found
- Which descriptions (if any) were updated in `glosario.json`
- Which files were modified

---

## Rules

- `skill.json` is the source of truth for `description` — always read from there, never infer
- Never delete or reorder sections in `README.md` — only replace the skills table and the structure tree
- Always regenerate `glosario.md` fully from `glosario.json`
- Skill links in `glosario.md` must be relative paths from the repo root
- If a skill version has no `usage.md`, still include it in the table but note the file is missing
