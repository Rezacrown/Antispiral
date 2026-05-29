# Blueprint Format

Blueprint adalah structured YAML yang menjadi cetak biru arsitektur sebelum implementasi codebase. Berbeda dengan narrative design doc, blueprint bersifat machine-readable dan bisa direview secara high-level tanpa membaca code.

Output directory: `docs/superpowers/blueprints/`

---

## UI Blueprint — `ui.blueprint.yaml`

### Structure

```yaml
schemaVersion: 1
name: <string>
description: <string>
kind: <Web SPA | Mobile | CLI>

pages:
  - name: <string>
    route: <string>
    components:
      - <component_name>
    actions:
      - <action_name>

actions:
  - name: <string>
    page: <string>
    trigger: <string>
    validation: <string>
    onError: <string>

entities:
  - entity: <string>
    fields:
      - name: <string>
        type: <string>
        required: <boolean>

states:
  - name: <string>
    type: <loading | empty | error | success>
```

### Field Descriptions

#### Top-level

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `schemaVersion` | integer | yes | Must be 1 |
| `name` | string | yes | DApp / app name |
| `description` | string | no | Short description |
| `kind` | string | yes | Platform type: `Web SPA`, `Mobile`, `CLI` |

#### pages[]

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Page identifier |
| `route` | string | yes | URL path (e.g. `/dashboard`, `/profile/:id`) |
| `components` | [string] | no | UI components on this page |
| `actions` | [string] | no | References to actions defined below |

#### actions[]

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Action identifier |
| `page` | string | yes | Which page the action belongs to |
| `trigger` | string | no | Trigger event (e.g. "onClick", "onSubmit") |
| `validation` | string | no | Frontend validation (e.g. "wallet must be connected") |
| `onError` | string | no | Error handling (e.g. "display toast", "redirect to error page") |

#### entities[]

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `entity` | string | yes | Entity name |
| `fields` | [{name, type, required}] | no | Field definitions |

#### states[]

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | State identifier |
| `type` | enum | yes | One of: `loading`, `empty`, `error`, `success` |

### Validation Rules

1. `schemaVersion` must be 1
2. At least one page required (or explicitly empty)
3. Each page name must be unique
4. Action names must be unique
5. `actions[].page` must reference an existing page

---

## Backend Blueprint — `backend.blueprint.yaml`

### Structure

```yaml
schemaVersion: 1
name: <string>
description: <string>
language: <string>
framework: <string>
kind: <REST | GraphQL | EventBus>

operations:
  - name: <string>
    type: <query | mutation | event | job>
    method: <GET | POST | PUT | DELETE | PATCH>
    endpoint: <string>
    description: <string>
    input:
      fields:
        - name: <string>
          type: <string>
          required: <boolean>
          description: <string>
    validations:
      - step: <integer>
        name: <string>
        type: <auth | input | business | state>
        description: <string>
        onFail: <string>
    output:
      fields:
        - name: <string>
          type: <string>
          required: <boolean>
    errors:
      - name: <string>
        description: <string>
```

### Field Descriptions

#### Top-level

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `schemaVersion` | integer | yes | Must be 1 |
| `name` | string | yes | Service name |
| `description` | string | no | Service description |
| `language` | string | no | e.g. "TypeScript", "Go", "Python" |
| `framework` | string | no | e.g. "Express", "Fastify", "Gin" |
| `kind` | string | yes | API style: `REST`, `GraphQL`, `EventBus` |

#### operations[]

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Operation identifier (unique) |
| `type` | enum | yes | One of: `query`, `mutation`, `event`, `job` |
| `method` | enum | yes | HTTP method: `GET`, `POST`, `PUT`, `DELETE`, `PATCH` |
| `endpoint` | string | yes | URL path (e.g. `/api/proposals`) |
| `description` | string | no | What this operation does |
| `input` | object | no | Input schema with fields array |
| `validations` | [{step, name, type, description, onFail}] | no | Step-by-step validation logic |
| `output` | object | no | Output schema with fields array |
| `errors` | [{name, description}] | no | Error codes this operation can return |

#### operations[].validations[]

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `step` | integer | yes | Validation order (1, 2, 3...) |
| `name` | string | yes | Validation identifier |
| `type` | enum | yes | One of: `auth`, `input`, `business`, `state` |
| `description` | string | yes | What this validation checks |
| `onFail` | string | yes | Error code thrown when validation fails |

**Validation type meanings:**
- `auth` — Authentication / authorization check (e.g. "user must be admin")
- `input` — Input format / constraint validation (e.g. "title must be 1-200 chars")
- `business` — Business logic validation (e.g. "no duplicate title")
- `state` — System state validation (e.g. "proposal must be open for voting")

### Validation Rules

1. `schemaVersion` must be 1
2. Each operation name must be unique
3. `type` must be one of: `query`, `mutation`, `event`, `job`
4. `method` must be one of: `GET`, `POST`, `PUT`, `DELETE`, `PATCH`
5. `endpoint` must start with `/`

---

## Smart Contract Blueprint — `smart-contract.blueprint.yaml`

### Structure

```yaml
schemaVersion: 1
name: <string>
description: <string>
language: <string>
framework: <string>
kind: <EVM | StarkNet | Solana>

events:
  - name: <string>
    fields:
      - name: <string>
        type: <string>
        indexed: <boolean>

operations:
  - name: <string>
    type: <query | mutation>
    target: <string>
    function: <string>
    description: <string>
    input:
      fields:
        - name: <string>
          type: <string>
          required: <boolean>
    validations:
      - step: <integer>
        name: <string>
        type: <auth | input | state>
        description: <string>
        onFail: <string>
    output:
      fields:
        - name: <string>
          type: <string>
          required: <boolean>
    errors:
      - name: <string>
        description: <string>
```

### Field Descriptions

#### Top-level

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `schemaVersion` | integer | yes | Must be 1 |
| `name` | string | yes | Contract name |
| `description` | string | no | Contract description |
| `language` | string | no | e.g. "Solidity", "Cairo", "Rust" |
| `framework` | string | no | e.g. "OpenZeppelin", "StarkNet" |
| `kind` | string | yes | Chain type: `EVM`, `StarkNet`, `Solana` |

#### events[]

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Event name |
| `fields` | [{name, type, indexed}] | no | Event parameters |

#### operations[]

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Operation identifier (unique) |
| `type` | enum | yes | One of: `query`, `mutation` |
| `target` | string | yes | Contract name or reference |
| `function` | string | yes | Function signature (e.g. "createProposal(string,uint256)") |
| `description` | string | no | What this operation does |
| `input` | object | no | Input parameters |
| `validations` | [{step, name, type, description, onFail}] | no | Step-by-step on-chain validation |
| `output` | object | no | Return values |
| `errors` | [{name, description}] | no | Custom errors / revert reasons |

### Validation Rules

1. `schemaVersion` must be 1
2. Each operation name must be unique
3. `type` must be `query` or `mutation`
4. `target` is required
5. `function` must include parentheses with types

---

## Database Blueprint — `database.blueprint.yaml`

### Structure

```yaml
schemaVersion: 1
name: <string>
description: <string>
type: <SQL | NoSQL>
engine: <string>

collections:
  - name: <string>
    description: <string>
    fields:
      - name: <string>
        type: <string>
        required: <boolean>
        unique: <boolean>
        default: <any>
        relation: <string>
    indexes:
      - fields:
          - <string>
        unique: <boolean>
    relationships:
      - type: <belongsTo | hasMany | hasOne | manyToMany>
        target: <string>
        via: <string>
```

### Field Descriptions

#### Top-level

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `schemaVersion` | integer | yes | Must be 1 |
| `name` | string | yes | Database name |
| `description` | string | no | Database description |
| `type` | enum | yes | One of: `SQL`, `NoSQL` |
| `engine` | string | no | e.g. "PostgreSQL", "MongoDB", "SQLite" |

#### collections[]

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Collection/table name (unique) |
| `description` | string | no | Collection purpose |
| `fields` | [{name, type, required, unique, default, relation}] | yes | Field definitions |
| `indexes` | [{fields, unique}] | no | Index definitions |
| `relationships` | [{type, target, via}] | no | Relationship definitions |

#### collections[].fields[]

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Field name |
| `type` | string | yes | Data type (e.g. "uuid", "string", "integer", "boolean", "timestamp") |
| `required` | boolean | no | Default: false |
| `unique` | boolean | no | Default: false |
| `default` | any | no | Default value |
| `relation` | string | no | Foreign key reference (e.g. "users.id") |

### Validation Rules

1. `schemaVersion` must be 1
2. `type` must be `SQL` or `NoSQL`
3. Each collection name must be unique
4. Each field name within a collection must be unique
5. If `relationships` is specified, `via` must reference an existing field
6. If `relationships` is specified, `target` must reference an existing collection

---

## Integration Guide — `integration-guide.md`

### Format

File ini adalah markdown narrative yang auto-generated dari semua `.blueprint.yaml` files. Bukan YAML.

### Sections

1. **Architecture Overview** — Ringkasan layer yang ada dan hubungannya
2. **Layer Connections** — Tabel wiring antar layer (UI action → Backend endpoint → SC function)
3. **Data Flow** — Alur data untuk setiap operasi utama (step-by-step)
4. **SDK / Library Recommendations** — Library yang dibutuhkan untuk tiap koneksi (wagmi, ethers, axios, dll)
5. **Environment Requirements** — Network, API keys, env variables

### Example Outline

```markdown
# Integration Guide

## Architecture
Frontend (React) ↔ Backend (Express) ↔ Smart Contract (Solidity/Sepolia)

## Layer Connections
| UI Action | API Endpoint | SC Function | SDK |
|-----------|-------------|-------------|-----|
| createProposal | POST /api/proposals | createProposal(string,uint256) | wagmi + axios |

## Data Flow: createProposal
1. User fills form → click Submit
2. Frontend POST /api/proposals (JWT auth)
3. Backend validates input, creates DB record
4. Backend returns proposal ID
5. Frontend calls contract.createProposal(title, deadline)
6. Frontend waits for tx receipt
7. On success: redirect to /proposals/:id
8. On error: display toast with error message

## SDK
- wagmi + viem (wallet connect + contract calls)
- axios (REST API)
```
