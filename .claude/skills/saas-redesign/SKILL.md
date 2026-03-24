---
name: saas-redesign
description: Redesign a Vue 3 application UI into a modern SaaS-style interface with vertical sidebar navigation, consistent spacing, and a polished professional look. Transforms top-nav layouts into sidebar-driven layouts.
---

# SaaS UI Redesign Skill

Redesign the Vue 3 frontend into a modern SaaS-style interface. This skill converts the existing top-navigation layout into a vertical sidebar layout with consistent spacing, refined typography, and a polished professional appearance.

**MANDATORY: Delegate ALL .vue file creation and modification to the `vue-expert` subagent.** Provide the subagent with the full design specification and let it handle implementation.

## Design Philosophy

The target aesthetic is a clean, enterprise SaaS dashboard — think Linear, Vercel, or Notion's admin panels. Restrained color palette, generous whitespace, clear visual hierarchy. No decorative elements. Let the data breathe.

## Architecture: Layout Transformation

### Before (Current)
```
┌──────────────────────────────────────────────┐
│  top-nav (sticky, 70px)                      │
│  [Logo] [Nav Links...] [Lang] [Profile]      │
├──────────────────────────────────────────────┤
│  FilterBar (sticky below nav)                │
├──────────────────────────────────────────────┤
│                                              │
│              main-content                    │
│           (router-view, 1600px)              │
│                                              │
└──────────────────────────────────────────────┘
```

### After (Target)
```
┌────────┬─────────────────────────────────────┐
│        │  top-bar (compact, ~56px)           │
│  side  │  [Search] [FilterBar] [Profile]     │
│  bar   ├─────────────────────────────────────┤
│        │                                     │
│ [Logo] │         main-content                │
│ [Nav]  │        (router-view)                │
│ [Nav]  │                                     │
│ [Nav]  │                                     │
│ [Nav]  │                                     │
│ [Nav]  │                                     │
│ [Nav]  │                                     │
│        │                                     │
│[bottom]│                                     │
│[Lang]  │                                     │
│[User]  │                                     │
└────────┴─────────────────────────────────────┘
```

## Implementation Steps

Follow this exact sequence. Each step should be completed before moving to the next.

### Step 1: Read Current State

Before making any changes, read these files to understand the current implementation:

```
client/src/App.vue              # Root layout, nav, global styles
client/src/components/FilterBar.vue   # Filter bar component
client/src/components/ProfileMenu.vue # Profile menu component
client/src/components/LanguageSwitcher.vue # Language toggle
client/src/main.js              # Router configuration (route paths + names)
client/src/composables/useAuth.js     # User data
client/src/composables/useI18n.js     # Translation keys
```

### Step 2: Create the Sidebar Component

Create `client/src/components/SidebarNav.vue` — a new vertical navigation sidebar.

**Requirements:**

```
Structure:
├── .sidebar (fixed left, full height)
│   ├── .sidebar-header (logo area)
│   │   ├── App name (from i18n: nav.companyName)
│   │   └── Optional: subtitle or version badge
│   ├── .sidebar-nav (main navigation, flex-grow)
│   │   ├── .nav-section-label ("MAIN" or "NAVIGATION")
│   │   └── .nav-item (one per route, with icon + label)
│   │       ├── SVG icon (inline, 20x20)
│   │       ├── Route label (from i18n keys)
│   │       └── Optional: badge/count
│   └── .sidebar-footer (bottom-pinned)
│       ├── LanguageSwitcher
│       └── User avatar + name (compact, clickable)
```

**Sidebar Dimensions & Style:**
- Width: `240px` (fixed), collapsible to `64px` on small screens (optional)
- Background: `#0f172a` (slate-900, dark sidebar)
- Text: `#94a3b8` (slate-400, muted)
- Active item: left border accent `#3b82f6` (blue-500), background `rgba(59, 130, 246, 0.1)`, text `#ffffff`
- Hover: background `rgba(255, 255, 255, 0.05)`, text `#e2e8f0`
- Section labels: `#475569` (slate-600), 11px, uppercase, letter-spacing 0.05em
- Logo text: `#ffffff`, font-weight 700, font-size 1.125rem
- Nav items: 14px font-size, 500 weight, padding `0.625rem 1rem`, border-radius `6px`, margin `2px 8px`
- Sidebar footer: border-top `1px solid rgba(255,255,255,0.08)`

**Navigation Items (map to existing routes):**
Each nav item needs an inline SVG icon. Use simple, clean SVG paths (no icon library needed):

| Route | Label (i18n key) | Icon Concept |
|-------|------------------|--------------|
| `/` | nav.overview | Grid/dashboard (4 squares) |
| `/inventory` | nav.inventory | Box/package |
| `/orders` | nav.orders | Clipboard/list |
| `/spending` | nav.finance | Dollar/chart |
| `/demand` | nav.demandForecast | Trending up |
| `/reports` | Reports | Bar chart |

**Script Requirements:**
- Use Composition API (`setup()`)
- Import `useI18n` for translated labels
- Import `useAuth` for user display name and initials
- Use `vue-router`'s `useRoute()` for active state detection
- Emit events for profile/tasks modals: `@show-profile-details`, `@show-tasks`

### Step 3: Create the Top Bar Component

Create `client/src/components/TopBar.vue` — a slim horizontal bar for the content area.

**Requirements:**

```
Structure:
├── .top-bar (sticky, top of content area)
│   ├── .top-bar-left
│   │   └── Page title (dynamic, based on current route)
│   ├── .top-bar-center (or right-aligned)
│   │   └── FilterBar (existing component, adapted)
│   └── .top-bar-right
│       └── ProfileMenu (existing, or simplified)
```

**Top Bar Style:**
- Height: `56px`
- Background: `#ffffff`
- Border-bottom: `1px solid #e2e8f0`
- Padding: `0 1.5rem`
- Page title: 16px, font-weight 600, color `#0f172a`

**Page Title Mapping:**
Use route meta or a computed map:
```javascript
const pageTitles = {
  '/': 'Dashboard',
  '/inventory': 'Inventory',
  '/orders': 'Orders',
  '/spending': 'Finance',
  '/demand': 'Demand Forecast',
  '/reports': 'Reports'
}
```
Integrate with i18n if translation keys exist for these.

### Step 4: Adapt FilterBar for Inline Layout

Modify `client/src/components/FilterBar.vue` to work inside the top bar.

**Changes:**
- Remove sticky positioning (parent handles placement now)
- Make it more compact: smaller select heights (32px), smaller font (13px)
- Use `flex-wrap: wrap` with smaller gap (0.5rem)
- Remove background/border (inherits from top bar)
- The reset button should be smaller and more subtle

### Step 5: Restructure App.vue Layout

Transform `App.vue` from vertical stack to horizontal sidebar layout.

**Template Structure:**
```html
<div class="app">
  <SidebarNav
    @show-profile-details="showProfileDetails = true"
    @show-tasks="showTasks = true"
  />
  <div class="main-area">
    <TopBar />
    <main class="main-content">
      <router-view />
    </main>
  </div>
  <!-- Modals stay at root level -->
</div>
```

**CSS Changes in App.vue:**

Remove:
- `.top-nav`, `.nav-container`, `.nav-tabs` and all related styles
- Sticky nav positioning

Add/Update:
```css
.app {
  display: flex;
  min-height: 100vh;
}

.main-area {
  flex: 1;
  margin-left: 240px;  /* sidebar width */
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  background: #f8fafc;
}

.main-content {
  flex: 1;
  padding: 1.5rem;
  max-width: 1400px;
  width: 100%;
}
```

### Step 6: Refine Global Styles

Update the global styles in `App.vue` for the new layout context:

**Spacing System (use consistently):**
- `xs`: 0.25rem (4px)
- `sm`: 0.5rem (8px)
- `md`: 1rem (16px)
- `lg`: 1.5rem (24px)
- `xl`: 2rem (32px)

**Card Refinements:**
```css
.card {
  background: #ffffff;
  border: 1px solid #e2e8f0;
  border-radius: 12px;       /* slightly more rounded */
  padding: 1.5rem;           /* consistent with spacing system */
  margin-bottom: 1.5rem;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.04);  /* subtle depth */
}
```

**Stat Cards:**
```css
.stat-card {
  padding: 1.5rem;
  border-radius: 12px;
  border: 1px solid #e2e8f0;
  background: #ffffff;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.04);
}

.stat-label {
  font-size: 0.8125rem;
  font-weight: 500;          /* less aggressive than 600 */
  text-transform: none;      /* stop shouting */
  letter-spacing: normal;
  color: #64748b;
}

.stat-value {
  font-size: 1.875rem;       /* slightly smaller, less overwhelming */
  font-weight: 700;
  margin-top: 0.25rem;
}
```

**Table Refinements:**
```css
th {
  font-size: 0.75rem;
  font-weight: 600;
  color: #64748b;
  text-transform: uppercase;
  letter-spacing: 0.04em;
  padding: 0.75rem 1rem;
}

td {
  padding: 0.75rem 1rem;     /* match th padding */
  font-size: 0.875rem;
}
```

**Page Header:**
```css
.page-header h2 {
  font-size: 1.5rem;         /* smaller, sidebar gives context */
  font-weight: 700;
  color: #0f172a;
}
```

## Design Token Reference

Keep these values consistent across all components:

```
Colors:
  --sidebar-bg:       #0f172a
  --sidebar-text:     #94a3b8
  --sidebar-active:   #3b82f6
  --page-bg:          #f8fafc
  --card-bg:          #ffffff
  --border:           #e2e8f0
  --text-primary:     #0f172a
  --text-secondary:   #64748b
  --text-muted:       #94a3b8

Radius:
  --radius-sm:        6px
  --radius-md:        8px
  --radius-lg:        12px

Shadows:
  --shadow-sm:        0 1px 2px rgba(0, 0, 0, 0.04)
  --shadow-md:        0 4px 12px rgba(0, 0, 0, 0.06)

Typography:
  --font-sans:        'Inter', -apple-system, system-ui, sans-serif
  --text-xs:          0.75rem
  --text-sm:          0.8125rem
  --text-base:        0.875rem
  --text-lg:          1rem
  --text-xl:          1.25rem
  --text-2xl:         1.5rem
```

## Constraints

- **Do NOT change** any backend code, API contracts, or data models
- **Do NOT change** the routing paths or composable logic (useFilters, useAuth, useI18n)
- **Do NOT remove** any existing functionality (filters, modals, language switching, profile menu)
- **Do NOT add** external CSS libraries (Tailwind, Bootstrap, etc.) — use plain CSS
- **Do NOT change** the `<script>` logic in view components (Dashboard, Inventory, etc.) unless strictly necessary for layout
- **Preserve** all existing i18n translation keys
- **Preserve** the FilterBar's 4-filter functionality and reset button
- **Preserve** all modal triggers and behavior (ProfileDetailsModal, TasksModal)

## Quality Checklist

Before considering the redesign complete:

- [ ] Sidebar renders with all 6 navigation links
- [ ] Active route is visually highlighted in sidebar
- [ ] Clicking sidebar links navigates correctly
- [ ] FilterBar works in the top bar (all 4 filters + reset)
- [ ] Profile menu and language switcher are accessible
- [ ] All modals still open and close correctly
- [ ] Page content has consistent padding and spacing
- [ ] Cards have uniform border-radius and shadow treatment
- [ ] Tables are readable with proper column alignment
- [ ] Loading and error states still display correctly
- [ ] No horizontal scrollbar on standard viewport sizes
- [ ] The app works at 1280px+ viewport width

## Testing

After implementation, verify by running the frontend:
```bash
cd client && npm run dev
```

Then use Playwright MCP tools to verify:
1. Navigate to `http://localhost:3000`
2. Confirm sidebar is visible with all nav links
3. Click each nav link and verify page loads
4. Confirm filters work in the new layout
5. Check that profile/tasks modals still trigger correctly
