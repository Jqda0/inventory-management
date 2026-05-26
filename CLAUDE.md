# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Factory Inventory Management System — full-stack demo with Vue 3 frontend, Python FastAPI backend, and in-memory mock data (no database).

## Critical Tool Usage Rules

### Subagents
- **vue-expert**: **MANDATORY** for creating or significantly modifying any `.vue` file
- **code-reviewer**: Use after writing significant code
- **Explore**: Use for codebase structure questions and pattern searches
- **general-purpose**: Use for complex multi-step tasks

### Skills
- **backend-api-test**: Use when writing or modifying tests in `tests/backend/` with pytest and FastAPI TestClient

### MCP Tools
- **ALWAYS use GitHub MCP tools** (`mcp__github__*`) for ALL GitHub operations
  - Exception: Local branches only — use `git checkout -b` instead of `mcp__github__create_branch`
- **ALWAYS use Playwright MCP tools** (`mcp__playwright__*`) for browser testing
  - Test against: `http://localhost:3000` (frontend), `http://localhost:8001` (API)

## Stack
- **Frontend**: Vue 3 + Composition API + Vite (port 3000)
- **Backend**: Python FastAPI (port 8001)
- **Data**: JSON files in `server/data/` loaded into memory at startup via `server/mock_data.py`

## Commands

```bash
# One-command startup (macOS/Linux)
./scripts/start.sh
./scripts/stop.sh

# Backend (manual)
cd server && uv venv && uv sync   # first-time setup
cd server && uv run python main.py

# Frontend (manual)
cd client && npm install          # first-time setup
cd client && npm run dev
cd client && npm run build        # production build → client/dist/

# Tests
cd tests && uv run pytest backend/ -v          # all backend tests
cd tests && uv run pytest backend/test_foo.py  # single test file
```

## Architecture

### Frontend Routes (client/src/main.js)
| Path | View |
|------|------|
| `/` | Dashboard.vue |
| `/inventory` | Inventory.vue |
| `/orders` | Orders.vue |
| `/demand` | Demand.vue |
| `/spending` | Spending.vue |
| `/reports` | Reports.vue |

### Composables (client/src/composables/)
- **useFilters**: Global filter state (warehouse, category, time period, status) — shared across all views via module-level refs, not Vuex/Pinia
- **useAuth**: Current user state and auth helpers
- **useI18n**: Translation function `t()` — all UI strings go through this

### Key Components (client/src/App.vue level)
- **FilterBar**: Global filter UI rendered on every page; state in `useFilters`
- **ProfileMenu**: User profile + triggers TasksModal
- **LanguageSwitcher**: Toggles i18n locale
- **TasksModal**: In-app tasks backed by `/api/tasks` (CRUD via POST/DELETE/PATCH)

### Data Flow
```
FilterBar (useFilters composable)
  → view's watch(filters, loadData)
  → api.js (builds URLSearchParams, skips 'all' values)
  → FastAPI apply_filters() + filter_by_month()
  → Pydantic validation → response
  → raw refs (allOrders, inventoryItems)
  → computed properties for display
```

### Backend Data Files (server/data/)
- `inventory.json` — items with warehouse, category, sku, quantity, reorder_point
- `orders.json` — orders with status, order_date, warehouse, category, total_value
- `demand_forecasts.json` — item demand with trend direction
- `backlog_items.json` — delayed items needing purchase orders
- `purchase_orders.json` — POs linked to backlog items by `backlog_item_id`
- `spending.json` — contains `spending_summary`, `monthly_spending`, `category_spending`
- `transactions.json` — recent spending transactions

### API Endpoints
- `GET /api/inventory` — Filters: warehouse, category
- `GET /api/orders` — Filters: warehouse, category, status, month (supports `Q1-2025` quarters)
- `GET /api/dashboard/summary` — All filters; aggregates inventory value, low stock, pending orders
- `GET /api/demand`, `GET /api/backlog` — No filters
- `GET /api/spending/{summary|monthly|categories|transactions}`
- `GET /api/reports/{quarterly|monthly-trends}` — Computed from orders data
- `POST /api/purchase-orders` — Create PO linked to backlog item
- `GET /api/purchase-orders/{backlogItemId}`
- `GET/POST/DELETE/PATCH /api/tasks`

## Key Patterns & Gotchas

**Filter values**: Passing `'all'` is the same as omitting the filter — both frontend (`api.js`) and backend (`apply_filters`) skip 'all' values.

**Month filtering**: Supports `YYYY-MM` direct match or `Q1-2025` through `Q4-2025` quarters. Inventory has no time dimension — don't add month filter to inventory endpoints.

**Pydantic ↔ JSON sync**: When changing a JSON data file's structure, update the corresponding Pydantic model in `server/main.py` or the endpoint will return a validation error.

**Purchase orders**: The backlog endpoint dynamically computes `has_purchase_order` by checking `purchase_orders` list — it's not stored in `backlog_items.json`.

**v-for keys**: Use `sku`, `id`, `month`, etc. — never array index.

**Dates**: Validate before `.getMonth()` — `new Date(str)` can return `Invalid Date`.

**Revenue goals**: $800K/month (single month selected), $9.6M YTD (all months).

## Design System
- Colors: Slate/gray (`#0f172a`, `#64748b`, `#e2e8f0`)
- Status badges: green/blue/yellow/red
- Charts: Custom SVG elements, CSS Grid for layouts
- No emojis in UI

Edit my CLAUDE.md file to add "Always document non-obvious logic changes with comments"

