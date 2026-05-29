# Blueprint Engine — Implementation Plan

> **For agentic workers:** Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add structured YAML blueprint generation to Antispiral's brainstorming flow — letting users review architecture at a high level without reading code, and giving implementers/subagents a precise validation reference.

**Architecture:** Minimal changes to existing Antispiral. 0 new skills. 1 new reference file. 2 modified skills (brainstorming + writing-plans). New `docs/superpowers/blueprints/` output folder.

**Files to create:** 6 (blueprint-format.md, 4 example YAML blueprints, 1 example integration guide)
**Files to modify:** 2 (brainstorming/SKILL.md, writing-plans/SKILL.md)

---

### Task 1: Create `blueprint-format.md` reference

**Files:**
- Create: `skills/brainstorming/references/blueprint-format.md`

**Content:** Single reference file with sectioned format for all 4 layers:

- **UI Blueprint** — `docs/superpowers/blueprints/ui.blueprint.yaml`
  - Top-level: `schemaVersion`, `name`, `description`, `kind` (Web SPA, Mobile, CLI)
  - `pages[]`: name, route, components, actions
  - `actions[]`: name, page, trigger, validation, onError
  - `entities[]`: entity, fields (name, type, required)
  - `states[]`: name, type (loading/empty/error/success)
  - Validation rules

- **Backend Blueprint** — `docs/superpowers/blueprints/backend.blueprint.yaml`
  - Top-level: `schemaVersion`, `name`, `description`, `language`, `framework`, `kind` (REST, GraphQL, EventBus)
  - `operations[]`: name, type (query/mutation/event/job), method, endpoint, description
  - `operations[].input.fields[]`: name, type, required, description
  - `operations[].validations[]`: step, name, type (auth/input/business/state), description, onFail
  - `operations[].output.fields[]`: name, type, required
  - `operations[].errors[]`: name, description
  - Validation rules

- **Smart Contract Blueprint** — `docs/superpowers/blueprints/smart-contract.blueprint.yaml`
  - Top-level: `schemaVersion`, `name`, `description`, `language`, `framework`, `kind` (EVM, StarkNet, Solana)
  - `events[]`: name, fields (name, type, indexed)
  - `operations[]`: name, type (query/mutation), target, function, description
  - `operations[].validations[]`: step, name, type (auth/input/state), description, onFail
  - `operations[].output.fields[]`, `operations[].errors[]`
  - Validation rules

- **Database Blueprint** — `docs/superpowers/blueprints/database.blueprint.yaml`
  - Top-level: `schemaVersion`, `name`, `description`, `type` (SQL/NoSQL), `engine`
  - `collections[]`: name, description
  - `collections[].fields[]`: name, type, required, unique, default, relation
  - `collections[].indexes[]`: fields, unique
  - `collections[].relationships[]`: type (belongsTo/hasMany/hasOne/manyToMany), target, via
  - Validation rules

- **Integration Guide** — `docs/superpowers/blueprints/integration-guide.md`
  - Auto-generated dari .blueprint.yaml files setelah semua approve
  - Sections: Architecture overview, Layer connections, Data flow, SDK recommendations, Wiring table

- **Full example** — Voting DApp (all 4 layers + integration guide example)

---

### Task 2: Modify `brainstorming/SKILL.md` — add blueprint step

**Files:**
- Modify: `skills/brainstorming/SKILL.md`

**Changes:**
- Checklist step 6 (write design doc) dan step 7 (existing: user review), sisipkan step baru:

```
7. Offer to generate structured blueprints:
   "I can also create YAML blueprints from our design — structured
    blueprints that map out architecture, operations, validations, 
    and data flow per layer. You can review the architecture at a 
    high level without reading code. Want me to generate them?"
   
   If yes:
   a. Ask which layers need blueprints (ui / backend / smart-contract / database)
   b. Read `references/blueprint-format.md`
   c. Generate each layer to `docs/superpowers/blueprints/{layer}.blueprint.yaml`
   d. Present each blueprint for user review → approve / request changes
   e. After all blueprints approved, generate `docs/superpowers/blueprints/integration-guide.md`

8. Spec self-review — cover BOTH narrative doc AND blueprints (check consistency, no placeholder, no contradiction)
9. User reviews both spec doc and blueprints → approve
10. Invoke writing-plans
```

---

### Task 3: Create example blueprints for reference

**Files:**
- Create: `docs/superpowers/blueprints/EXAMPLE-ui.blueprint.yaml`
- Create: `docs/superpowers/blueprints/EXAMPLE-backend.blueprint.yaml`
- Create: `docs/superpowers/blueprints/EXAMPLE-smart-contract.blueprint.yaml`
- Create: `docs/superpowers/blueprints/EXAMPLE-database.blueprint.yaml`
- Create: `docs/superpowers/blueprints/EXAMPLE-integration-guide.md`

**Content:** Full Voting DApp example blueprints — realistic, detailed, with validations step-by-step. These serve as both reference for users AND test data.

**Format:** Each file follows exactly the format defined in `blueprint-format.md`.

---

### Task 4: Modify `writing-plans/SKILL.md` — reference blueprints

**Files:**
- Modify: `skills/writing-plans/SKILL.md`

**Changes:**
- Add 2 baris setelah file structure mapping section:
  ```
  - Read any `.blueprint.yaml` files in `docs/superpowers/blueprints/`
    and reference operations/validations/entities in task descriptions.
  ```
- Tujuan: Setiap task yang involve operation tertentu, task description-nya include validasi logic dari blueprint (step-by-step validations, onFail errors). Implementer subagent gak perlu nebak.

---

### Known Decisions

- **Existing projects** → Not in scope. Future: `blueprint-from-codebase` skill.
- **Subagent prompts** → Not modified for now. Will update in a later phase. (Info validasi udah masuk via task description dari writing-plans)
- **Codebase validation** → Not in scope. Future: `codebase-validation` skill.
- **context.md** → Not in scope. Will add when needed.
- **Documentation** → UI docs not needed yet.
