---
name: frontend-component
description: Split frontend UI into logical, composable components with clean separation of concerns. Use when refactoring React/Vue/Svelte/HTML UI files, extracting tabs/sections/panels/forms from large components, organizing component folders by product information architecture, or splitting CSS into component-owned style files without over-fragmenting.
---

# Frontend Component Refactoring

Use this skill when a frontend file has grown into mixed concerns: page orchestration, tabs, provider-specific panels, forms, cards, async state, and CSS all in one place.

## Core principle

Split by product structure and separation of concerns, not by arbitrary size targets.

Prefer nested ownership that mirrors where UI appears:

```txt
settings/
  SettingsPage.tsx              # orchestration/state/routing
  SettingsSidebar.tsx
  SettingsPanelNotice.tsx
  connections/
    ConnectionsPanel.tsx        # tab shell / section composition
    GoogleConnectionPanel.tsx   # Google only
    OutlookConnectionPanel.tsx  # Outlook only
    NotionConnectionPanel.tsx   # Notion only
  mcp/
    McpConnectionsPanel.tsx     # one component if cohesive
```

Avoid dumping unrelated components into a flat `components/` directory unless they are truly reusable across domains.

## Refactor workflow

1. Read the current component and identify product regions: page shell, navigation, tab shell, each tab/section, repeated cards/forms, notices.
2. Decide the minimum useful split. Do not extract every small JSX fragment.
3. Keep page-level files responsible for orchestration:
   - fetching/loading state
   - active route/tab state
   - mutation handlers
   - composing panels
4. Move provider/domain markup into provider/domain components:
   - Google UI should not contain Outlook logic
   - Outlook UI should not contain Notion logic
   - Workspace UI should not mix account UI
5. Keep a cohesive complex feature as one component unless it has natural internal seams. A self-contained MCP form/list can remain one `McpConnectionsPanel`.
6. Run the narrowest build/typecheck after each meaningful extraction.

## What to extract

Extract when one of these is true:

- The markup belongs to a named product concept: `GoogleConnectionPanel`, `WorkspacePanel`, `AccountPanel`.
- The component mixes several mutually exclusive branches, such as tabs.
- The same UI primitive repeats with stable behavior.
- A section has its own form fields, status summary, actions, and copy.
- The parent file becomes hard to scan because provider-specific details dominate orchestration.

Do not extract when:

- The new component would only hide two or three lines.
- Props become more complex than the JSX being moved.
- The extracted component has no product name beyond `Section`, `Block`, or `Content`.
- You are splitting only to reduce line count.

## Preferred component boundaries

### Page/controller component

Keep in the page:

- data loading effects
- route/query param handling
- top-level active tab selection
- API calls and mutation handlers
- derived summaries passed into panels

The page should read like a table of contents for the screen.

### Tab shell component

A tab shell owns:

- tab list rendering
- keyboard tab navigation
- selecting which panel to show

It should not include all provider-specific markup inline. Compose provider panels instead.

### Provider/section panel

A provider panel owns only that provider's visual UI:

- status header
- status fields
- provider-specific actions
- provider-specific empty/config forms

Example: `GoogleConnectionPanel` gets `googleStatus`, `googleScopeSummary`, `connectGoogle`, `disconnectGoogle`. It should not know about Outlook or Notion.

### Cohesive feature panel

Keep as one component when its list, form, and helper rows are one workflow. Example: an MCP panel can include saved-server cards, server form, and key/value rows if they are not reused elsewhere.

## Props guidance

Pass explicit props first. Do not prematurely create context.

Good:

```tsx
<GoogleConnectionPanel
  googleStatus={googleStatus}
  googleScopeSummary={googleScopeSummary}
  submitting={submitting}
  connectGoogle={connectGoogle}
  disconnectGoogle={disconnectGoogle}
/>
```

Bad:

```tsx
<GoogleConnectionPanel settingsState={settingsState} />
```

If props become excessive, look for a real domain object or hook boundary; do not hide everything in a generic bag.

## CSS organization

Split CSS by ownership and import order.

Use a global entry file:

```css
@import './styles/base.css';
@import './styles/components/shell.css';
@import './styles/components/settings/settings.css';
@import './styles/components/settings/connections.css';
@import './styles/components/settings/mcp.css';
@import './styles/responsive.css';
```

Guidelines:

- `base.css`: variables, resets, body, default button/input typography.
- `components/<domain>.css`: shared app regions such as shell, auth, chat, messages.
- `components/<screen>/<section>.css`: screen-specific sections when a screen grows.
- Keep responsive overrides last unless using colocated CSS modules.
- Preserve cascade order when splitting existing CSS.
- Do not create a CSS file for every component unless the styling is truly separate.

## Naming

Use product-language names:

- `ConnectionsPanel`
- `GoogleConnectionPanel`
- `PersonalNotionConnectionPanel`
- `WorkspacePanel`
- `McpConnectionsPanel`

Avoid generic names:

- `MainContent`
- `PanelContent`
- `SectionView`
- `TabOne`

## Quality checks

After refactoring, verify:

- The parent page is shorter and easier to scan.
- Each extracted component has one obvious reason to change.
- Provider-specific UI is isolated by provider.
- Components are nested according to where they appear in the product.
- CSS imports preserve previous cascade behavior.
- Build/typecheck passes.
