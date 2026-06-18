# Code Intelligence Layer

This layer answers: "Does the AI actually understand the system before changing it?"

Use it only when the task needs system, module, API, dependency, or impact understanding.

## General Principles

- Read code first, infer second.
- Mark inferred or uncertain conclusions as "To confirm".
- Produce durable artifacts under `docs/project/`.
- Refresh only the relevant slice of the system.
- Respect existing boundaries documented in `docs/project/modules.md` and `docs/process/boundaries.md`.

---

## 1. system-map

### Goal

Answer: "What repositories, services, ports, dependencies, and deployment boundaries exist in this workspace?"

### Inputs

- `docs/project/tech-stack.md`, if present
- `docs/local-startup-commands.md`, if present
- Repository roots containing `package.json`, `pom.xml`, `go.mod`, `pyproject.toml`, or similar manifests
- Repo-local `README.md`, `AGENTS.md`, or `CLAUDE.md`
- Environment and deployment docs, if present

### Steps

1. Find repository roots and service manifests.
2. Read local docs to extract each repository's role.
3. Inspect dependency manifests to identify internal dependencies.
4. Inspect frontend proxy/config files to identify API targets.
5. Inspect app config files for ports and health checks.
6. Summarize the system into `docs/project/system-map.md`.

### Output Template

```markdown
# System Map

> Last refreshed: yyyy-MM-dd
> Scope: <repositories/modules covered>

## Repositories and Services

| Repository | Role | Port | Start command | Health check |
| --- | --- | --- | --- | --- |
| services/api | Backend API | `<api-port>` | `npm run dev` | `/health` |
| apps/web | Web app | `<web-port>` | `npm run dev` | `/` |

## Dependency Direction

~~~text
apps/web -> services/api -> shared/database
~~~

## Contract Points

| Contract | Provider | Consumer | Documentation |
| --- | --- | --- | --- |
| Public REST API | services/api | apps/web | docs/project/api-index/api.md |

## To Confirm

- <uncertain item>
```

### Refresh When

- A repository or service is added or removed.
- Ports, startup commands, or health checks change.
- Cross-repository contracts change.
- A new session needs full-system onboarding and no recent map exists.

---

## 2. module-map

### Goal

Answer: "What does this module own, who depends on it, and what boundaries matter?"

### Inputs

- `docs/project/modules.md`, if present
- Target module source tree
- Public interfaces, controllers, handlers, jobs, or adapters
- `docs/process/boundaries.md`, if present

### Steps

1. List the target module's package or folder structure.
2. Find public entry points such as controllers, handlers, commands, jobs, interfaces, or exported functions.
3. Identify outbound dependencies.
4. Identify consumers of the module's public contracts.
5. Identify owned data stores, tables, queues, topics, files, or external APIs.
6. Update the relevant section in `docs/project/modules.md`.

### Output Template

```markdown
### Module: <module-name>

| Dimension | Content |
| --- | --- |
| Responsibility | <one sentence> |
| Path | `<path>` |
| Public entry points | <controllers/handlers/interfaces> |
| Depends on | <modules/services> |
| Consumed by | <modules/services> |
| Owned data | <tables/queues/files/etc.> |
| High-risk areas | <from boundaries.md, if any> |
| Related version docs | docs/visions/<version>/ |
| To confirm | <uncertain item> |
```

### Refresh When

- Public interfaces or entry points change.
- Dependency direction changes.
- A module enters a new version of development.
- A review suggests a boundary issue.

---

## 3. api-index

### Goal

Answer: "What is the full chain for this API or public contract?"

### Inputs

- Controllers, route handlers, RPC handlers, or public interface definitions
- Service/business logic
- Data access layer
- Auth, validation, error-handling, and contract docs

### Steps

1. List relevant routes or public methods.
2. For each entry point, extract method/path, input, output, auth behavior, validation, and errors.
3. Trace the call chain through service/business logic.
4. Trace storage or external API calls.
5. Summarize in `docs/project/api-index/<module>.md`.

### Output Template

```markdown
# API Index - <module-name>

> Last refreshed: yyyy-MM-dd
> Scope: <controllers/handlers covered>

## Endpoints

### POST /example

| Dimension | Content |
| --- | --- |
| Handler | `ExampleController#create` |
| Auth | Required |
| Input | `CreateExampleRequest` |
| Output | `ExampleResponse` |
| Service | `ExampleService#create` |
| Storage | `examples` table |
| Errors | `VALIDATION_ERROR`, `NOT_FOUND` |
| Chain | Controller -> Service -> Repository -> Database |

## To Confirm

- <uncertain item>
```

### Refresh When

- Path, input, output, auth, error code, or storage behavior changes.
- A new public endpoint or contract is added.
- A review involves contract risk.

---

## 4. diff analyzer

### Goal

Answer: "What does this change affect, and where is the risk?"

### Inputs

- `git status --short`
- `git diff HEAD`
- Untracked new files
- Existing module-map or api-index artifacts

### Steps

1. Capture changed files and diff stats.
2. Read concrete diffs.
3. Read complete changed files when diff context is not enough.
4. Bucket files by concern:
   - data/storage/config/migrations
   - service/business/API layer
   - frontend/UI/client contracts
   - infrastructure/build/docs
5. Identify contract changes, high-risk areas, data changes, and cross-module dependencies.
6. Reverse-trace consumers of changed public methods, schemas, or data stores.
7. Produce an impact report inline or in the relevant version document.

### Output Template

```markdown
## Diff Analysis Report

> Generated: yyyy-MM-dd
> Scope: <repositories/modules>

### Change Summary

| Dimension | Content |
| --- | --- |
| Changed files | N |
| Added / Modified / Deleted | A / M / D |
| Modules | <list> |
| Diff size | +XXX / -YYY |

### Buckets

- Data/config/migrations: <files>
- Service/API: <files>
- Frontend/client: <files>
- Infrastructure/docs: <files>

### Impact

| Change | Type | Impact | Risk |
| --- | --- | --- | --- |
| `<symbol>` | Interface contract | Consumers: <list> | High |

### Dependency Trace

~~~text
Changed: ExampleService#create
  -> Consumer: ExampleController#create
  -> Storage: examples
~~~

### High-Risk Points

- <risk>

### To Confirm

- <uncertain item>
```

### Run When

- Before Layer 3 review for non-trivial changes.
- Before cross-module or cross-repository changes.
- Before release sealing.
- When the user asks about impact or risk.
