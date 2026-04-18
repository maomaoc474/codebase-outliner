# Codebase Outliner

Generate a structured Markdown outline of one or more project directories, showing file relationships, API endpoint definitions, API call references, and cross-project connections -- all visualized with Mermaid diagrams.

The output is a single `.md` file the user can open in any Mermaid-compatible viewer (GitHub, VS Code preview, Obsidian, etc.).

## Workflow

Follow these phases in order. Each phase builds on the previous one.

### Phase 1: Discovery

Determine which project directories to scan.

- If the user specified paths, use those.
- If working in a monorepo, detect top-level packages/apps (`packages/`, `apps/`, `client/`, `server/`).
- If in a single project, scan the current working directory.

For each project, determine its **role** and **framework**:

| Check | What it reveals |
|-------|----------------|
| `package.json` dependencies | react, vue, angular, express, nestjs, next, nuxt |
| `requirements.txt` / `pyproject.toml` | flask, django, fastapi |
| `go.mod`, `Cargo.toml` | Go / Rust frameworks |
| Directory conventions | `src/routes/` -> backend, `src/components/` -> frontend |

Label each project with role + stack, e.g. "Frontend (React + TypeScript)", "Backend (Express + TypeScript)".

### Phase 2: Structure Scan

For each project directory:

1. **Build file tree** -- List all source files using your file search capabilities. Exclude these directories:
   `node_modules/`, `dist/`, `build/`, `.git/`, `__pycache__/`, `venv/`, `.next/`, `.nuxt/`, `coverage/`, `.cache/`

2. **Identify key files** -- Flag entry points, config files, route definitions, store/state files, and service/API files. These anchor the diagrams.

3. **Categorize** -- Count files by role: X components, Y routes, Z services, etc.

### Phase 3: Dependency Analysis

Scan source files for import/require statements to build a dependency graph. Refer to `references/detection-patterns.md` for the full list of import patterns to detect.

Focus on **internal** imports (within the project). External package imports appear in the tech stack summary but are not graphed individually.

**Diagram sizing rule**: When a project has more than ~40 source files, group nodes by directory rather than individual files to keep Mermaid diagrams readable.

### Phase 4: API Endpoint Detection

Scan backend/fullstack project(s) for route definitions. Refer to `references/detection-patterns.md` for framework-specific patterns.

For each endpoint, capture:
- **HTTP method** (GET, POST, PUT, DELETE, PATCH)
- **Route path** (resolve prefixes from `app.use('/prefix', router)` or `@Controller('/prefix')`)
- **Source file and line number**
- **Handler name** (function or method name)
- **Middleware** (auth, validation -- if identifiable on the same line or decorator)

### Phase 5: API Call Detection

Scan frontend project(s) for outgoing HTTP calls. Refer to `references/detection-patterns.md` for call patterns.

For each call, capture:
- **HTTP method** (from function name or options object)
- **URL/path** (resolve string literals; note template variables as `{param}`)
- **Source file and line number**
- **Enclosing function or component name**

Also check for a **base URL** or **API prefix** configured in:
- `.env` / `.env.local` files (`VITE_API_URL`, `REACT_APP_API_URL`, `NEXT_PUBLIC_API_URL`)
- Axios defaults (`axios.defaults.baseURL`, `axios.create({ baseURL })`)
- Service/API modules (a central file that exports configured HTTP client)

Note the base URL for cross-referencing in Phase 6.

### Phase 6: Cross-Reference

Match frontend API calls to backend endpoints:

1. Normalize both sides: strip the base URL prefix from frontend calls, prepend route prefixes to backend endpoints.
2. Match by **method + path**. Handle path parameters flexibly -- `/users/:id` matches `/users/${userId}`.

Produce three categories:
- **Matched**: frontend call has a corresponding backend endpoint (list both files)
- **Unmatched frontend calls**: no backend match found (likely external APIs -- label them)
- **Unused backend endpoints**: no frontend caller detected (could be used by other clients, webhooks, or not yet wired up)

### Phase 7: Generate Output

Produce a single Markdown file. Refer to `references/output-template.md` for the exact structure to follow.

Save to the location the user specifies, or default to `PROJECT-OUTLINE.md` in the current working directory.

After writing the file, tell the user where it was saved and give a brief summary of what was found (e.g., "Found 12 API endpoints, 15 frontend calls, 11 matched pairs, 1 unmatched call, 1 unused endpoint").

## Mermaid Diagram Guidelines

Mermaid has practical rendering limits. Diagrams with 60+ nodes become unreadable. To manage this:

- **Group by directory** when there are many files. Show `components/` as one node, not 30 individual files.
- **Use subgraphs** to separate projects, layers, or feature areas.
- **Limit cross-reference diagrams** to the top ~25 API connections. If more exist, mention the count and point to the tables for the full list.
- **Use short labels**: `api.ts\nfetchUsers` rather than full paths.
- **Style nodes** to distinguish roles:
  - Frontend components: `fill:#e1f5fe` (light blue)
  - Backend routes: `fill:#e8f5e9` (light green)
  - Services/middleware: `fill:#fff3e0` (light orange)
  - Data/models: `fill:#f3e5f5` (light purple)
  - Shared/types: `fill:#fce4ec` (light pink)
- **Direction**: Use `LR` (left-to-right) for dependency flows, `TB` (top-to-bottom) for layered architecture overviews.

## Multi-Project Handling

When scanning multiple projects:

- Each project gets its own **Directory Structure** section
- The **File Dependency Graph** uses subgraphs per project; cross-project imports appear as dashed arrows (`-.->`)
- **API Endpoints** and **API Calls** tables are grouped by project
- The **Cross-Reference** section bridges projects -- this is where multi-project analysis is most valuable
- In a monorepo with `packages/` or `apps/`, treat each package/app as a separate project

## Handling Large Projects

For projects with 100+ source files:

1. Show only the top 2 levels of directory structure, with file counts per subdirectory
2. Group the dependency graph by module/directory, not individual files
3. Always itemize every API endpoint and every API call -- these are the highest-value output
4. In the architecture overview, focus on layers and data flow rather than individual files
5. Consider splitting the Mermaid diagrams: one for the dependency overview, separate ones per feature area if the cross-reference is large
