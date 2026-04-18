# Detection Patterns

Comprehensive patterns for detecting imports, API endpoints, and API calls across frameworks.

## Table of Contents
- [Import / Dependency Patterns](#import--dependency-patterns)
- [API Endpoint Patterns (Backend)](#api-endpoint-patterns-backend)
- [API Call Patterns (Frontend)](#api-call-patterns-frontend)

---

## Import / Dependency Patterns

### JavaScript / TypeScript (ES Modules)

```regex
import\s+.*\s+from\s+['"](\.[^'"]+)['"]
import\s*\{[^}]*\}\s*from\s+['"](\.[^'"]+)['"]
export\s*\{[^}]*\}\s*from\s+['"](\.[^'"]+)['"]
import\s*\(['"](\.[^'"]+)['"]\)
```

Examples:
```
import App from './App'
import { useState } from 'react'          ← external, skip in graph
import { UserType } from '../types/user'
export { helper } from './utils'
const module = await import('./lazy')
```

### JavaScript / TypeScript (CommonJS)

```regex
require\(['"](\.[^'"]+)['"]\)
```

Examples:
```
const express = require('express')         ← external, skip in graph
const router = require('./routes/auth')
```

### Vue Single File Components (.vue)

Scan the `<script>` or `<script setup>` block for standard ES import patterns above. Also detect:
```regex
components:\s*\{([^}]+)\}
```
This registers local components that may be imported above.

### HTML

```regex
<script\s+src=['"]([^'"]+)['"]
<link\s+[^>]*href=['"]([^'"]+\.css)['"]
```

### CSS / SCSS / Less

```regex
@import\s+['"]([^'"]+)['"]
@use\s+['"]([^'"]+)['"]
```

### Python

```regex
from\s+(\S+)\s+import
import\s+(\S+)
```

Filter to relative imports (starting with `.`) or same-package imports.

### Go

```regex
import\s+"([^"]+)"
```

Filter to internal packages (same module path prefix).

---

## API Endpoint Patterns (Backend)

### Express.js / Koa / Hono

```regex
(app|router)\.(get|post|put|delete|patch|all|use)\s*\(\s*['"`]([^'"`]+)['"`]
```

For route prefixes, also detect mounting:
```regex
app\.use\s*\(\s*['"`]([^'"`]+)['"`]\s*,\s*(\w+)
```
This tells you the prefix applied to all routes in that router.

Examples:
```js
app.use('/api/auth', authRouter)     // prefix: /api/auth
router.get('/', getAll)              // full path: /api/auth/
router.post('/login', login)         // full path: /api/auth/login
app.get('/health', healthCheck)      // full path: /health
```

### Flask (Python)

```regex
@(app|blueprint|\w+_bp)\.route\s*\(\s*['"]([^'"]+)['"].*methods\s*=\s*\[([^\]]+)\]
@(app|blueprint|\w+_bp)\.(get|post|put|delete|patch)\s*\(\s*['"]([^'"]+)['"]
```

### FastAPI (Python)

```regex
@(app|router)\.(get|post|put|delete|patch)\s*\(\s*['"]([^'"]+)['"]
```

Also detect router prefixes:
```regex
app\.include_router\s*\(\s*(\w+)\s*,\s*prefix\s*=\s*['"]([^'"]+)['"]
```

### Django (Python)

```regex
path\s*\(\s*['"]([^'"]+)['"]
re_path\s*\(\s*r?['"]([^'"]+)['"]
```

In `urls.py`, also detect `include()` for nested URL configs:
```regex
path\s*\(\s*['"]([^'"]+)['"],\s*include\s*\(
```

### NestJS (TypeScript)

```regex
@(Get|Post|Put|Delete|Patch)\s*\(\s*['"]?([^'")\s]*)['"]?\s*\)
@Controller\s*\(\s*['"]([^'"]+)['"]
```

The `@Controller` decorator provides the route prefix for all methods in that class.

### Ruby on Rails

```regex
(get|post|put|patch|delete)\s+['"]([^'"]+)['"]
resources?\s+:(\w+)
```

---

## API Call Patterns (Frontend)

### fetch API

```regex
fetch\s*\(\s*[`'"](\/[^'"`$]*)[`'"]
fetch\s*\(\s*[`'"]\$\{[^}]+\}([^'"`]*)[`'"]
fetch\s*\(\s*(\w+)\s*[,)]
```

The method defaults to GET unless specified in the options:
```regex
method:\s*['"]?(GET|POST|PUT|DELETE|PATCH)['"]?
```

### Axios

```regex
axios\.(get|post|put|delete|patch)\s*\(\s*[`'"](\/[^'"`]*)[`'"]
axios\s*\(\s*\{[^}]*url:\s*[`'"](\/[^'"`]*)[`'"]
axios\s*\(\s*\{[^}]*method:\s*['"]?(get|post|put|delete|patch)['"]?
```

Also detect axios instance creation for base URL:
```regex
axios\.create\s*\(\s*\{[^}]*baseURL:\s*[`'"](\/[^'"`]*)[`'"]
axios\.defaults\.baseURL\s*=\s*[`'"]([^'"`]+)[`'"]
```

### Custom API Services

Many projects wrap fetch/axios in a service module. Detect the common pattern:
```regex
(api|apiClient|httpClient|http|request)\.(get|post|put|delete|patch)\s*\(\s*[`'"](\/[^'"`]*)[`'"]
```

### Angular HttpClient

```regex
this\.http\.(get|post|put|delete|patch)\s*[<(]\s*[`'"](\/[^'"`]*)[`'"]
```

### jQuery

```regex
\$\.(ajax|get|post|getJSON)\s*\(\s*[`'"](\/[^'"`]*)[`'"]
\$\.ajax\s*\(\s*\{[^}]*url:\s*[`'"](\/[^'"`]*)[`'"]
```

### Vue Resource (legacy)

```regex
this\.\$http\.(get|post|put|delete|patch)\s*\(\s*[`'"](\/[^'"`]*)[`'"]
```

### Environment Variables (Base URL)

Check these files for API base URL configuration:
```
.env
.env.local
.env.development
.env.production
```

Common variable names:
```regex
(VITE_API_URL|REACT_APP_API_URL|NEXT_PUBLIC_API_URL|VUE_APP_API_URL|API_BASE_URL|BASE_URL)\s*=\s*(.+)
```

Also check for proxy configuration in:
- `vite.config.ts` / `vite.config.js` → `server.proxy`
- `vue.config.js` → `devServer.proxy`
- `next.config.js` → `rewrites` or `redirects`
- `package.json` → `proxy` field (Create React App)
