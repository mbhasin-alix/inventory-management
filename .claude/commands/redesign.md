---
description: Redesign the inventory management app UI into a modern SaaS-style layout with a light vertical sidebar replacing the top nav bar. Delegates all .vue changes to vue-expert.
---

Transform this app's UI from a horizontal top-nav layout into a modern SaaS-style interface with a **light vertical sidebar** on the left. This skill touches exactly two files and requires no backend changes.

---

## Constraints

- **MANDATORY**: Delegate ALL `.vue` file changes to the `vue-expert` subagent. Do not write or edit `.vue` files yourself.
- Do not modify any files in `client/src/views/` — the views have no top-nav dependencies and need no changes.
- Do not modify backend files.
- After the vue-expert completes its work, run the Playwright verification steps.

---

## Step 1 — Rewrite `client/src/App.vue`

Delegate the following full rewrite to **vue-expert**. Provide the complete spec below as context.

### New layout structure

Replace the current `.app` (flex-column with a sticky top nav) with a **flex-row** layout:

```
.app (display: flex, min-height: 100vh, background: #f8fafc)
  ├─ .sidebar  (240px, white, sticky full-height, border-right)
  │   ├─ .sidebar-logo
  │   ├─ .sidebar-nav (6 router-links with SVG icons)
  │   └─ .sidebar-footer (LanguageSwitcher + ProfileMenu)
  └─ .app-body (flex: 1, flex-column, overflow-x: hidden)
      ├─ <FilterBar />
      └─ <main class="main-content">
          └─ <router-view />
```

Remove the `<header class="top-nav">` block entirely (logo, `.nav-tabs`, LanguageSwitcher, ProfileMenu all move into `.sidebar`).

### Sidebar CSS specs

```css
.sidebar {
  width: 240px;
  min-height: 100vh;
  position: sticky;
  top: 0;
  align-self: flex-start;
  background: #ffffff;
  border-right: 1px solid #e2e8f0;
  display: flex;
  flex-direction: column;
  flex-shrink: 0;
  z-index: 100;
}

.sidebar-logo {
  padding: 1.25rem 1rem 1rem;
  border-bottom: 1px solid #e2e8f0;
}

/* Logo text: company name bold #0f172a, subtitle smaller #64748b */

.sidebar-nav {
  flex: 1;
  padding: 0.75rem;
  display: flex;
  flex-direction: column;
  gap: 0.125rem;
}

.sidebar-nav a {
  display: flex;
  align-items: center;
  gap: 0.625rem;
  padding: 0.5rem 0.75rem;
  border-radius: 6px;
  font-size: 0.875rem;
  color: #475569;
  text-decoration: none;
  transition: background 0.15s, color 0.15s;
  border-left: 3px solid transparent;  /* reserve space so active border doesn't shift layout */
}

.sidebar-nav a:hover {
  background: #f8fafc;
  color: #0f172a;
}

.sidebar-nav a.router-link-active {
  background: #eff6ff;
  color: #2563eb;
  font-weight: 600;
  border-left-color: #2563eb;
}

.sidebar-nav svg {
  width: 18px;
  height: 18px;
  flex-shrink: 0;
}

.sidebar-footer {
  border-top: 1px solid #e2e8f0;
  padding: 0.75rem 1rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.app-body {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow-x: hidden;
  min-width: 0;
}

.main-content {
  flex: 1;
  padding: 1.5rem 2rem;
  max-width: 1600px;
  width: 100%;
}
```

### Nav items with inline SVG icons

Use these exact 6 nav links. Each SVG uses `currentColor` and is `18×18px`, `fill="none"`, `stroke="currentColor"`, `stroke-width="1.75"`, `stroke-linecap="round"`, `stroke-linejoin="round"`.

| Path | Label | SVG path(s) |
|------|-------|-------------|
| `/` | Overview | Two rows of two squares: `M3 3h7v7H3z M13 3h7v7h-7z M3 13h7v7H3z M13 13h7v7h-7z` (scale to 18×18 viewBox) |
| `/inventory` | Inventory | Box/cube: `M21 8l-9-5-9 5v8l9 5 9-5V8z M12 3v18 M3 8l9 4 9-4` |
| `/orders` | Orders | Clipboard: `M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2 M9 5a2 2 0 002 2h2a2 2 0 002-2M9 12h6 M9 16h4` |
| `/spending` | Finance | Bar chart: `M18 20V10 M12 20V4 M6 20v-6` |
| `/demand` | Demand | Trending up: `M22 7l-9 9-4-4-5 5 M16 7h6v6` |
| `/reports` | Reports | Document lines: `M14 2H6a2 2 0 00-2 2v16a2 2 0 002 2h12a2 2 0 002-2V8l-6-6z M14 2v6h6 M16 13H8 M16 17H8 M10 9H8` |

**Important**: Use `router-link-active` (not `router-link-exact-active`) for the active class so `/inventory` and other sub-pages match correctly. For the Overview route `/` only, use `exact` prop or `router-link-exact-active` to prevent it from matching all routes.

### Modals

Keep `<ProfileDetailsModal />` and `<TasksModal />` at the bottom of the template, outside `.sidebar` and `.app-body` — they are full-page overlays and do not belong in either container.

### Imports to keep

Keep all existing imports: `FilterBar`, `ProfileMenu`, `LanguageSwitcher`, `ProfileDetailsModal`, `TasksModal`, `useFilters`, `useAuth`, `useI18n`. Remove references to the `isActive` helper or any route-checking logic that powered `.nav-tabs` active states — Vue Router handles this via `router-link-active` class automatically.

---

## Step 2 — Fix `client/src/components/FilterBar.vue`

One CSS change only — the FilterBar is no longer below a 70px top nav. Update the sticky offset:

```css
/* Change this: */
position: sticky;
top: 70px;

/* To this: */
position: sticky;
top: 0;
```

Delegate this single change to **vue-expert** as part of the same task, or make it directly if simpler given it is a one-line CSS update. The rest of FilterBar.vue is unchanged.

---

## Step 3 — Verify with Playwright

After vue-expert completes both changes:

1. Use `mcp__playwright__playwright_navigate` to open `http://localhost:3000`
2. Use `mcp__playwright__playwright_screenshot` to capture the full page — confirm:
   - Left sidebar is visible with logo and 6 nav items
   - Top nav bar is gone
   - FilterBar appears at the top of the content area (not offset)
   - Overview nav item is highlighted as active
3. Click `/inventory` via `mcp__playwright__playwright_click` — verify the Inventory nav item becomes active (blue highlight) and the page loads
4. Click back to `/` — verify Overview becomes active again
5. If anything looks wrong, read the browser console via `mcp__playwright__playwright_evaluate` to surface any JS errors, then fix before reporting done
