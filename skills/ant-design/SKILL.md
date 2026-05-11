---
name: ant-design
description: Decision guide for antd 6.x, Ant Design Pro 5/ProComponents, Ant Design X v2, and the offline `@ant-design/cli`. Use for component selection, theming/tokens, SSR, a11y, performance, routing/access/CRUD, AI/chat UI patterns, local API lookup, debugging, migration, and usage analysis.
---

# Ant Design

## S - Scope
- Target: `antd@^6` + React 18-19, with `ant-design-pro@^5` / `@ant-design/pro-components` and `@ant-design/x@^2` when needed.
- Tooling: `@ant-design/cli` for offline component metadata, demos, changelogs, migrations, linting, doctor checks, and usage analysis.
- Focus: decision guidance only; no end-user tutorials.
- Source policy: official docs only; no undocumented APIs or internal `.ant-*` coupling.

### Default assumptions
- Language: TypeScript.
- Styling: tokens first, then `classNames`/`styles`; avoid global overrides.
- Provider: one root `ConfigProvider` unless strict isolation is required.

### Mandatory rules
- Before writing or changing antd component code, query the component API first with `antd info <Component> --format json`. Do not rely on memory when the CLI can answer it offline.
- Always use `--format json` with `antd` CLI commands.
- If the project version matters, match it with `--version <x.y.z>` or let the CLI auto-detect from local `node_modules`.
- After changing antd code, run `antd lint <changed-path> --format json`.
- If an `antd` CLI command crashes, returns wrong data, or violates its documented behavior, prepare an `antd bug-cli` preview for user confirmation instead of silently working around it.
- For component questions, first map the component name to the official route slug `{components}` (lowercase kebab-case, e.g. `TreeSelect -> tree-select`, `Button -> button`), then request docs in this order (CN first, EN fallback):
  1. `https://ant.design/components/{components}-cn`
  2. `https://ant.design/components/{components}`
  - Examples: `tree-select-cn -> tree-select`, `button-cn -> button`.
- Use only documented antd/Pro/X APIs.
- Do not invent props/events/component names.
- Do not rely on internal DOM or `.ant-*` selectors.
- Theme priority: global tokens -> component tokens -> alias tokens.

## P - Process
### 1) Classify
- Identify layer: core antd, Pro, or X.
- Confirm version, rendering mode (CSR/SSR/streaming), data scale, and whether `@ant-design/cli` should be the primary lookup path.

### 2) Query authoritative sources
- Prefer local `@ant-design/cli` first for structured lookup:
  - `antd info` for props/API
  - `antd demo` for a working baseline
  - `antd doc` for full docs
  - `antd token` / `antd semantic` for theming and styling hooks
  - `antd doctor`, `antd lint`, `antd usage`, `antd migrate`, `antd changelog` when debugging or upgrading
- Then request the official component docs (`-cn` first, EN fallback) when narrative docs or cross-checking are needed.

### 3) Decide
- Provider baseline: CSR -> `ConfigProvider`; SSR -> `ConfigProvider` + `StyleProvider`.
- Theming baseline: global tokens -> component tokens -> `classNames`/`styles`.
- Output recommendation + risk + verification points (SSR/a11y/perf), citing CLI findings when used.

## O - Output
- Provide short decision rationale (1-3 sentences).
- Include minimal provider/theming strategy.
- Include concrete SSR/a11y/perf checks.
- For Pro: include route/menu/access and CRUD schema direction.
- For X: include message/tool schema and streaming state direction.

## References

| File | Use when |
| --- | --- |
| `references/antd-cli.md` | You need the exact offline CLI workflow for API lookup, demos, linting, doctor checks, migration, changelog review, usage analysis, or bug reporting. |

## Regression checklist
- [ ] One root `ConfigProvider`; SSR style order/hydration verified.
- [ ] Tokens first; no broad global `.ant-*` overrides.
- [ ] Table has stable `rowKey`; sort/filter/pagination entry is unified.
- [ ] Select remote mode disables local filter when using remote search.
- [ ] Upload controlled/uncontrolled mode is explicit with failure/retry path.
- [ ] Pro route/menu/access remain consistent with backend enforcement.
- [ ] X streaming supports stop/retry and deterministic tool rendering.
- [ ] If `antd` CLI was used, commands ran with `--format json` and any CLI defect was escalated via `antd bug-cli` preview.
](name: ant-design-expert
description: Decision guide for antd 6.x, Ant Design Pro 5/ProComponents, Ant Design X v2, antd-style, and the offline `@ant-design/cli`. Use for component selection, theming/tokens, SSR, a11y, performance, routing/access/CRUD, AI/chat UI patterns, local API lookup, debugging, migration, and usage analysis.
---

# Ant Design & ProComponents Expert Guide

## S - Scope
- Target: `antd@^6` + React 18-19, coupled with `ant-design-pro@^5`, `@ant-design/pro-components`, `antd-style`, and `@ant-design/x@^2` when needed.
- Tooling: `@ant-design/cli` for offline component metadata, demos, changelogs, migrations, linting, doctor checks, and usage analysis.
- Focus: Architectural decision guidance, precise component/API selection, strictly structured styling, scalable layouts, and robust network strategies.
- Source policy: Official docs only; no undocumented APIs, no hallucinated hooks, and strictly NO internal `.ant-*` coupling.

### Default assumptions
- Language: TypeScript.
- Styling: `antd-style` (`createStyles`) FIRST, falling back to Tokens, then `classNames`/`styles`. 
- Provider: One root `ConfigProvider` or `ProConfigProvider` unless strict micro-frontend isolation is required.

### Mandatory rules
- **Pre-flight Check:** Before writing or changing antd component code, query the component API first with `antd info <Component> --format json`. Do not rely on memory.
- **CLI Commands:** Always use `--format json` with `antd` CLI commands. Run `antd lint <changed-path> --format json` post-modification.
- **Fallback Verification:** If an `antd` CLI command crashes or violates behavior, prepare an `antd bug-cli` preview.
- **Component Docs Mapping:** Map component name to route slug `{components}` (lowercase kebab-case), request docs sequentially: `https://ant.design/components/{components}-cn` -> EN fallback.
- **Strict API Usage:** Use only documented APIs. Do not invent props/events.
- **[CRITICAL] Multi-Theme & Token Strategy:** When customizing styles, future multi-theme adaptation MUST be proactively considered. **Hardcoding hex/rgb color values is STRICTLY FORBIDDEN.** You MUST extract and use Ant Design Design Tokens (e.g., `token.colorPrimary`) via `useToken()` or `antd-style` context for all colors, borders, and shadows.
- **[CRITICAL] ProTable Pagination Schema:** For `ProTable` remote requests, ALWAYS explicitly map and rename the default `current` query parameter to `pageNum` within the `request` function to ensure unified backend alignment.
- **[CRITICAL] Desktop-First Grid Layout:** You MUST use structural grid layouts (`Row`/`Col`, `Space`, or CSS Grid/Flex via `antd-style`) for all complex interfaces. **Assume a Desktop-First environment.** UNLESS the user explicitly requests mobile client adaptation, aggressively IGNORE mobile viewport scaling (`xs`, `sm`) to optimize code density for enterprise dashboard scenarios.
- **[CRITICAL] Performance & Virtualization:** Always assume large datasets. You MUST enable the `virtual` prop for `<Table>`, `<Select>`, and `<Tree>` when dealing with >100 items. Prevent excessive re-renders by memoizing complex custom `render` outputs in columns.
- **[CRITICAL] Form Dependency Handling:** STRICTLY FORBIDDEN to use `useEffect` loops to manually sync `ProForm` states. You MUST use `<ProFormDependency>` or the `dependencies` prop for intra-form field reactions, or handle updates explicitly within `onValuesChange`.
- **[CRITICAL] Mock Data Strict Isolation:** NEVER import static JSON/arrays directly into components for mock data. Components must remain oblivious. All mock data MUST be served via API intercepts (e.g., placing mock files in a dedicated `/mock` directory) and fetched using the standard `request` utility.
- **[CRITICAL] Visibility-Driven Fetching (Prevent Network Storms):** API requests MUST ONLY trigger when the target component is within the user's visible context. You MUST utilize techniques like `useInViewport` (from `ahooks`) for scrolling layouts, verify the active `Tab` key before fetching tab contents, and strictly enforce `destroyOnClose` on `Modal`/`Drawer` components to prevent hidden lifecycle background fetching.

## P - Process
### 1) Classify & Contextualize
- Identify the abstraction layer: Base antd, ProComponents (Form/Table/Layout), `antd-style` (CSS-in-JS), or Ant Design X (AI UI).
- Confirm rendering mode, data scale, network payload strategy, and whether `@ant-design/cli` should dictate the initial payload.

### 2) Query Authoritative Sources
- Prefer local `@ant-design/cli` first: `info`, `demo`, `token`, `semantic`, `doctor`, `lint`, `usage`.
- For CSS-in-JS: Leverage `antd-style` API mindset (`useTheme`, `createStyles`).
- For CRUD/Data: Rely on ProComponents `valueType` dictionary mapping before attempting custom renders.

### 3) Architectural Decisions
- **Provider Baseline:** CSR -> `ConfigProvider` | SSR -> `ConfigProvider` + `StyleProvider` (evaluate `hashPriority="high"` and `extractStaticStyle`).
- **ProComponents Strategy:** 
  - Never manually sync form states if `ProForm` or `EditableProTable` can handle it natively.
  - Rely on `request` API for tables; utilize `postData` only for payload transformation, not side effects.
  - Prefer built-in `valueType` over manual custom renders.
- **Output Validation:** Define risk vectors (SSR hydration mismatch, unhandled `request` promise rejection, missing `rowKey`, hardcoded colors, invisible data fetching).

## O - Output
- Provide short, authoritative decision rationale (1-3 sentences).
- Output minimal, robust code enforcing the Provider/Token-first styling, Desktop-First Grid layout, and lazy-loaded networking strategies.
- Detail concrete SSR/a11y/perf boundary checks.
- **For Pro:** Dictate route/menu configuration in `ProLayout`, and enforce `request` / `valueType` schemas.
- **For X:** Dictate message/tool schema, `useXAgent` state sync, and deterministic tool rendering during streaming.

## References

| File | Use when |
| --- | --- |
| `references/antd-cli.md` | You need the exact offline CLI workflow for API lookup, demos, linting, migration, changelogs, or usage analysis. |

## Regression Checklist
- [ ] Only one root `ConfigProvider` / `ProConfigProvider`; SSR style registry verified.
- [ ] **Styling & Theming:** Styles leverage `createStyles` or inline `token`. Zero hardcoded colors present. Absolutely no broad `.ant-*` global mutation.
- [ ] **Layout:** Employs explicit Grid/Flex structures. Mobile (`xs`, `sm`) breakpoints are intentionally omitted unless explicitly requested.
- [ ] **Data Fetching & Performance:** `ProTable` `current` is mapped to `pageNum`. Mock data is isolated to external API endpoints. Visibility checks (`useInViewport`, `destroyOnClose`, active Tab checks) are implemented to prevent hidden fetching. `virtual` rendering used for massive lists.
- [ ] **Forms:** Remote `Select` properly disables local filtering (`filterOption={false}`). `useEffect` is NOT used for state syncing between fields.
- [ ] **Uploads:** Controlled vs. Uncontrolled state is decisively chosen.
- [ ] **AI (X):** Streaming correctly halts/retries; `<Suggestion />` fallbacks map to disconnected states.
- [ ] CLI defect reporting (`antd bug-cli`) executed if anomalies triggered during local lookups.
](name: ant-design-expert
description: Decision guide for antd 6.x, Ant Design Pro 5/ProComponents, Ant Design X v2, antd-style, and the offline `@ant-design/cli`. Use for component selection, theming/tokens, SSR, a11y, performance, routing/access/CRUD, AI/chat UI patterns, local API lookup, debugging, migration, and usage analysis.
---

# Ant Design & ProComponents Expert Guide

## S - Scope
- Target: `antd@^6` + React 18-19, coupled with `ant-design-pro@^5`, `@ant-design/pro-components`, `antd-style`, and `@ant-design/x@^2` when needed.
- Tooling: `@ant-design/cli` for offline component metadata, demos, changelogs, migrations, linting, doctor checks, and usage analysis.
- Focus: Architectural decision guidance, precise component/API selection, strictly structured styling, scalable layouts, robust networking, and sensible component abstraction.
- Source policy: Official docs only; no undocumented APIs, no hallucinated hooks, and strictly NO internal `.ant-*` coupling.

### Default assumptions
- Language: TypeScript.
- Styling: `antd-style` (`createStyles`) FIRST, falling back to Tokens, then `classNames`/`styles`. 
- Provider: One root `ConfigProvider` or `ProConfigProvider` unless strict micro-frontend isolation is required.

### Mandatory rules
- **Pre-flight Check:** Before writing or changing antd component code, query the component API first with `antd info <Component> --format json`. Do not rely on memory.
- **CLI Commands:** Always use `--format json` with `antd` CLI commands. Run `antd lint <changed-path> --format json` post-modification.
- **Strict API Usage:** Use only documented APIs. Do not invent props/events.
- **[CRITICAL] Multi-Theme & Token Strategy:** When customizing styles, future multi-theme adaptation MUST be proactively considered. **Hardcoding hex/rgb color values is STRICTLY FORBIDDEN.** You MUST extract and use Ant Design Design Tokens via `useToken()` or `antd-style`.
- **[CRITICAL] Component Extraction & Anti-Wrappers:** You MUST extract repeated, stateless UI patterns (e.g., custom status badges, metric cards, specialized layouts) into shared components. However, it is **STRICTLY FORBIDDEN to create "God Wrappers" around complex stateful components like `ProTable` or `ProForm`**. Do not hide `ProTable` behind opaque custom generic list components. `ProTable` itself IS the abstraction for "Search + Table + Pagination". Wrapping it destroys TS inference, ref forwarding, and slot extensibility.
- **[CRITICAL] ProTable Pagination Schema:** For `ProTable` remote requests, ALWAYS explicitly map and rename the default `current` query parameter to `pageNum` within the `request` function.
- **[CRITICAL] Desktop-First Grid Layout:** You MUST use structural grid layouts (`Row`/`Col`, `Space`, or CSS Grid/Flex via `antd-style`). **Assume a Desktop-First environment.** Aggressively IGNORE mobile viewport scaling (`xs`, `sm`) to optimize code density unless explicitly requested.
- **[CRITICAL] Performance & Virtualization:** Assume large datasets. You MUST enable the `virtual` prop for `<Table>`, `<Select>`, and `<Tree>` when >100 items. Memoize complex custom `render` outputs in columns to prevent re-renders.
- **[CRITICAL] Form Dependency Handling:** STRICTLY FORBIDDEN to use `useEffect` loops to manually sync `ProForm` states. You MUST use `<ProFormDependency>` or the `dependencies` prop for intra-form field reactions.
- **[CRITICAL] Mock Data Strict Isolation:** NEVER import static JSON/arrays directly into components for mock data. All mock data MUST be served via API intercepts (e.g., `/mock` directory) and fetched using the standard `request` utility.
- **[CRITICAL] Visibility-Driven Fetching:** API requests MUST ONLY trigger when the target component is visible. Utilize `useInViewport`, verify active `Tab` keys, and strictly enforce `destroyOnClose` on `Modal`/`Drawer` components to prevent hidden background network storms.

## P - Process
### 1) Classify & Contextualize
- Identify the abstraction layer: Base antd, ProComponents, `antd-style`, or Ant Design X.
- Confirm rendering mode, data scale, network strategy, and abstraction necessity.

### 2) Query Authoritative Sources
- Prefer local `@ant-design/cli` first.
- For CSS-in-JS: Leverage `antd-style`.
- For CRUD/Data: Rely on ProComponents `valueType` dictionary mapping before attempting custom renders.

### 3) Architectural Decisions
- **Provider Baseline:** CSR -> `ConfigProvider` | SSR -> `ConfigProvider` + `StyleProvider`.
- **ProComponents Strategy:** 
  - Never manually sync form states if `ProForm` handles it.
  - Rely on `request` API for tables; utilize `postData` only for payload transformation.
  - Expose ProComponents APIs directly; do not bury them in unnecessary wrapper components.
- **Output Validation:** Define risk vectors (unhandled `request` rejection, missing `rowKey`, invisible fetching, over-abstraction destroying types).

## O - Output
- Provide short, authoritative decision rationale (1-3 sentences).
- Output minimal, robust code enforcing Token-first styling, Desktop-First Layout, Visibility-Driven Fetching, and Flat Component Hierarchies.
- Detail concrete SSR/a11y/perf boundary checks.
- **For Pro:** Dictate route/menu configuration, enforce `request` / `valueType` schemas, and validate component abstraction.
- **For X:** Dictate message/tool schema and streaming states.

## References

| File | Use when |
| --- | --- |
| `references/antd-cli.md` | You need the exact offline CLI workflow for API lookup, demos, linting, migration, or changelogs. |

## Regression Checklist
- [ ] Only one root `ConfigProvider` / `ProConfigProvider`; SSR style registry verified.
- [ ] **Styling & Theming:** Styles leverage `createStyles` or inline `token`. Zero hardcoded colors.
- [ ] **Architecture & Abstraction:** Stateless UI is extracted. `ProTable` and `ProForm` are NOT hidden behind custom black-box wrappers; their native APIs remain exposed and typed.
- [ ] **Layout:** Employs explicit Grid/Flex structures. Mobile breakpoints intentionally omitted.
- [ ] **Data Fetching & Performance:** `ProTable` `current` is mapped to `pageNum`. Mock data is isolated to external endpoints. Visibility checks implemented. `virtual` rendering used for massive lists.
- [ ] **Forms:** Remote `Select` disables local filtering. `useEffect` NOT used for state syncing.
- [ ] **AI (X):** Streaming correctly halts/retries.
- [ ] CLI defect reporting (`antd bug-cli`) executed if anomalies triggered.)
](name: ant-design-expert
description: Decision guide for antd 6.x, Ant Design Pro 5/ProComponents, Ant Design X v2, antd-style, and the offline `@ant-design/cli`. Use for component selection, theming/tokens, SSR, a11y, performance, routing/access/CRUD, AI/chat UI patterns, local API lookup, debugging, migration, E2E testability, and usage analysis.
---

# Ant Design & ProComponents Expert Guide

## S - Scope
- Target: `antd@^6` + React 18-19, coupled with `ant-design-pro@^5`, `@ant-design/pro-components`, `antd-style`, and `@ant-design/x@^2` when needed.
- Tooling: `@ant-design/cli` for offline component metadata, demos, changelogs, migrations, linting, doctor checks, and usage analysis.
- Focus: Architectural decision guidance, precise component/API selection, strictly structured styling, scalable layouts, robust networking, sensible component abstraction, and E2E automation readiness.
- Source policy: Official docs only; no undocumented APIs, no hallucinated hooks, and strictly NO internal `.ant-*` coupling.

### Default assumptions
- Language: TypeScript.
- Styling: `antd-style` (`createStyles`) FIRST, falling back to Tokens, then `classNames`/`styles`. 
- Provider: One root `ConfigProvider` or `ProConfigProvider` unless strict micro-frontend isolation is required.

### Mandatory rules
- **Pre-flight Check:** Before writing or changing antd component code, query the component API first with `antd info <Component> --format json`. Do not rely on memory.
- **CLI Commands:** Always use `--format json` with `antd` CLI commands. Run `antd lint <changed-path> --format json` post-modification.
- **Strict API Usage:** Use only documented APIs. Do not invent props/events.
- **[CRITICAL] Multi-Theme & Token Strategy:** When customizing styles, future multi-theme adaptation MUST be proactively considered. **Hardcoding hex/rgb color values is STRICTLY FORBIDDEN.** You MUST extract and use Ant Design Design Tokens via `useToken()` or `antd-style`.
- **[CRITICAL] Component Extraction & Anti-Wrappers:** You MUST extract repeated, stateless UI patterns into shared components. However, it is **STRICTLY FORBIDDEN to create "God Wrappers" around complex stateful components like `ProTable` or `ProForm`**. `ProTable` itself IS the abstraction. Wrapping it destroys TS inference, ref forwarding, and slot extensibility.
- **[CRITICAL] E2E Testability (Playwright):** All interactive elements (buttons, links, tabs, critical form inputs) MUST include deterministic `data-testid` attributes to facilitate stable Playwright/E2E automation. **NEVER rely on fragile `.ant-*` class selectors or DOM hierarchy for testing.** For ProComponents, utilize `fieldProps={{ 'data-testid': '...' }}` to penetrate inner DOM nodes.
- **[CRITICAL] ProTable Pagination Schema:** For `ProTable` remote requests, ALWAYS explicitly map and rename the default `current` query parameter to `pageNum` within the `request` function.
- **[CRITICAL] Desktop-First Grid Layout:** You MUST use structural grid layouts (`Row`/`Col`, `Space`, or CSS Grid/Flex via `antd-style`). **Assume a Desktop-First environment.** Aggressively IGNORE mobile viewport scaling (`xs`, `sm`) to optimize code density unless explicitly requested.
- **[CRITICAL] Performance & Virtualization:** Assume large datasets. You MUST enable the `virtual` prop for `<Table>`, `<Select>`, and `<Tree>` when >100 items. Memoize complex custom `render` outputs in columns to prevent cascading re-renders.
- **[CRITICAL] Form Dependency Handling:** STRICTLY FORBIDDEN to use `useEffect` loops to manually sync `ProForm` states. You MUST use `<ProFormDependency>` or the `dependencies` prop for intra-form field reactions.
- **[CRITICAL] Mock Data Strict Isolation:** NEVER import static JSON/arrays directly into components for mock data. All mock data MUST be served via API intercepts (e.g., `/mock` directory) and fetched using the standard `request` utility.
- **[CRITICAL] Visibility-Driven Fetching:** API requests MUST ONLY trigger when the target component is visible. Utilize `useInViewport`, verify active `Tab` keys, and strictly enforce `destroyOnClose` on `Modal`/`Drawer` components to prevent hidden background network storms.

## P - Process
### 1) Classify & Contextualize
- Identify the abstraction layer: Base antd, ProComponents, `antd-style`, or Ant Design X.
- Confirm rendering mode, data scale, network strategy, and automation test requirements.

### 2) Query Authoritative Sources
- Prefer local `@ant-design/cli` first.
- For CSS-in-JS: Leverage `antd-style`.
- For CRUD/Data: Rely on ProComponents `valueType` dictionary mapping before attempting custom renders.

### 3) Architectural Decisions
- **Provider Baseline:** CSR -> `ConfigProvider` | SSR -> `ConfigProvider` + `StyleProvider`.
- **ProComponents Strategy:** 
  - Never manually sync form states if `ProForm` handles it.
  - Rely on `request` API for tables; utilize `postData` only for payload transformation.
  - Expose ProComponents APIs directly; do not bury them in unnecessary wrapper components.
- **Output Validation:** Define risk vectors (unhandled `request` rejection, missing `rowKey`, invisible fetching, missing test IDs).

## O - Output
- Provide short, authoritative decision rationale (1-3 sentences).
- Output minimal, robust code enforcing Token-first styling, Desktop-First Layout, Visibility-Driven Fetching, and E2E automation readiness.
- Detail concrete SSR/a11y/perf boundary checks.
- **For Pro:** Dictate route/menu configuration, enforce `request` schemas, and validate component abstraction.
- **For X:** Dictate message/tool schema and streaming states.

## References

| File | Use when |
| --- | --- |
| `references/antd-cli.md` | You need the exact offline CLI workflow for API lookup, demos, linting, migration, or changelogs. |

## Regression Checklist
- [ ] Only one root `ConfigProvider` / `ProConfigProvider`; SSR style registry verified.
- [ ] **Styling & Theming:** Styles leverage `createStyles` or inline `token`. Zero hardcoded colors.
- [ ] **Architecture & Abstraction:** Stateless UI is extracted. `ProTable` and `ProForm` native APIs remain exposed and typed.
- [ ] **Automation Readiness (E2E):** `data-testid` is explicitly implemented on all forms, tables, and interactive elements for Playwright stability.
- [ ] **Layout:** Employs explicit Grid/Flex structures. Mobile breakpoints intentionally omitted.
- [ ] **Data Fetching & Performance:** `ProTable` `current` is mapped to `pageNum`. Mock data is isolated to external endpoints. Visibility checks implemented. `virtual` rendering used for massive lists.
- [ ] **Forms:** Remote `Select` disables local filtering. `useEffect` NOT used for state syncing.
- [ ] **AI (X):** Streaming correctly halts/retries.
- [ ] CLI defect reporting (`antd bug-cli`) executed if anomalies triggered.)
](name: ant-design-expert
description: Decision guide for antd 6.x, Ant Design Pro 5/ProComponents, Ant Design X v2, antd-style, and the offline `@ant-design/cli`. Use for component selection, theming/tokens, SSR, a11y, performance, routing/access/CRUD, AI/chat UI patterns, local API lookup, debugging, migration, E2E testability, and usage analysis.
---

# Ant Design & ProComponents Expert Guide

## S - Scope
- Target: `antd@^6` + React 18-19, coupled with `ant-design-pro@^5`, `@ant-design/pro-components`, `antd-style`, and `@ant-design/x@^2` when needed.
- Tooling: `@ant-design/cli` for offline component metadata, demos, changelogs, migrations, linting, doctor checks, and usage analysis.
- Focus: Architectural decision guidance, precise component/API selection, strictly structured styling, scalable layouts, robust networking, sensible component abstraction, and E2E automation readiness.
- Source policy: Official docs only; no undocumented APIs, no hallucinated hooks, and strictly NO internal `.ant-*` coupling.

### Default assumptions
- Language: TypeScript.
- Styling: `antd-style` (`createStyles`) FIRST, falling back to Tokens, then `classNames`/`styles`. 
- Provider: One root `ConfigProvider` or `ProConfigProvider` unless strict micro-frontend isolation is required.

### Mandatory rules
- **Pre-flight Check:** Before writing or changing antd component code, query the component API first with `antd info <Component> --format json`. Do not rely on memory.
- **CLI Commands:** Always use `--format json` with `antd` CLI commands. Run `antd lint <changed-path> --format json` post-modification.
- **Strict API Usage:** Use only documented APIs. Do not invent props/events.
- **[CRITICAL] Multi-Theme & Token Strategy:** When customizing styles, future multi-theme adaptation MUST be proactively considered. **Hardcoding hex/rgb color values is STRICTLY FORBIDDEN.** You MUST extract and use Ant Design Design Tokens via `useToken()` or `antd-style`.
- **[CRITICAL] Component Extraction & Anti-Wrappers:** You MUST extract repeated, stateless UI patterns into shared components. However, it is **STRICTLY FORBIDDEN to create "God Wrappers" around complex stateful components like `ProTable` or `ProForm`**. `ProTable` itself IS the abstraction. Wrapping it destroys TS inference, ref forwarding, and slot extensibility.
- **[CRITICAL] E2E Testability (Playwright):** All interactive elements (buttons, links, tabs, critical form inputs) MUST include deterministic `data-testid` attributes to facilitate stable Playwright/E2E automation. **NEVER rely on fragile `.ant-*` class selectors or DOM hierarchy for testing.** For ProComponents, utilize `fieldProps={{ 'data-testid': '...' }}` to penetrate inner DOM nodes.
- **[CRITICAL] ProTable Pagination Schema:** For `ProTable` remote requests, ALWAYS explicitly map and rename the default `current` query parameter to `pageNum` within the `request` function.
- **[CRITICAL] Desktop-First Grid Layout:** You MUST use structural grid layouts (`Row`/`Col`, `Space`, or CSS Grid/Flex via `antd-style`). **Assume a Desktop-First environment.** Aggressively IGNORE mobile viewport scaling (`xs`, `sm`) to optimize code density unless explicitly requested.
- **[CRITICAL] Performance & Virtualization:** Assume large datasets. You MUST enable the `virtual` prop for `<Table>`, `<Select>`, and `<Tree>` when >100 items. Memoize complex custom `render` outputs in columns to prevent cascading re-renders.
- **[CRITICAL] Form Dependency Handling:** STRICTLY FORBIDDEN to use `useEffect` loops to manually sync `ProForm` states. You MUST use `<ProFormDependency>` or the `dependencies` prop for intra-form field reactions.
- **[CRITICAL] Mock Data Strict Isolation:** NEVER import static JSON/arrays directly into components for mock data. All mock data MUST be served via API intercepts (e.g., `/mock` directory) and fetched using the standard `request` utility.
- **[CRITICAL] Visibility-Driven Fetching:** API requests MUST ONLY trigger when the target component is visible. Utilize `useInViewport`, verify active `Tab` keys, and strictly enforce `destroyOnClose` on `Modal`/`Drawer` components to prevent hidden background network storms.
- **[CRITICAL] Async State Feedback (Loading & Error):** You MUST provide explicit visual feedback for ALL asynchronous operations. Rely on ProComponents' native `loading` states where applicable. For custom layouts or non-Pro components, you MUST implement `<Skeleton active>` or `<Spin>` during data fetching. Implement graceful degradation for failed API calls using `<Result>`, `message.error`, or Error Boundaries. NEVER leave a broken or indefinitely empty UI without actionable feedback.

## P - Process
### 1) Classify & Contextualize
- Identify the abstraction layer: Base antd, ProComponents, `antd-style`, or Ant Design X.
- Confirm rendering mode, data scale, network strategy, and automation test requirements.

### 2) Query Authoritative Sources
- Prefer local `@ant-design/cli` first.
- For CSS-in-JS: Leverage `antd-style`.
- For CRUD/Data: Rely on ProComponents `valueType` dictionary mapping before attempting custom renders.

### 3) Architectural Decisions
- **Provider Baseline:** CSR -> `ConfigProvider` | SSR -> `ConfigProvider` + `StyleProvider`.
- **ProComponents Strategy:** 
  - Never manually sync form states if `ProForm` handles it.
  - Rely on `request` API for tables; utilize `postData` only for payload transformation.
  - Expose ProComponents APIs directly; do not bury them in unnecessary wrapper components.
- **Output Validation:** Define risk vectors (unhandled `request` rejection, missing `rowKey`, invisible fetching, missing test IDs, missing loading states).

## O - Output
- Provide short, authoritative decision rationale (1-3 sentences).
- Output minimal, robust code enforcing Token-first styling, Desktop-First Layout, Visibility-Driven Fetching, graceful error handling, and E2E automation readiness.
- Detail concrete SSR/a11y/perf boundary checks.
- **For Pro:** Dictate route/menu configuration, enforce `request` schemas, and validate component abstraction.
- **For X:** Dictate message/tool schema and streaming states.

## References

| File | Use when |
| --- | --- |
| `references/antd-cli.md` | You need the exact offline CLI workflow for API lookup, demos, linting, migration, or changelogs. |

## Regression Checklist
- [ ] Only one root `ConfigProvider` / `ProConfigProvider`; SSR style registry verified.
- [ ] **Styling & Theming:** Styles leverage `createStyles` or inline `token`. Zero hardcoded colors.
- [ ] **Architecture & Abstraction:** Stateless UI is extracted. `ProTable` and `ProForm` native APIs remain exposed and typed.
- [ ] **Automation Readiness (E2E):** `data-testid` is explicitly implemented on all forms, tables, and interactive elements for Playwright stability.
- [ ] **Layout:** Employs explicit Grid/Flex structures. Mobile breakpoints intentionally omitted.
- [ ] **Data Fetching & Performance:** `ProTable` `current` is mapped to `pageNum`. Mock data is isolated to external endpoints. Visibility checks implemented. `virtual` rendering used for massive lists.
- [ ] **State Feedback:** Async operations have explicit `loading` states (`Skeleton`/`Spin`). API failures are caught and displayed gracefully instead of silently failing.
- [ ] **Forms:** Remote `Select` disables local filtering. `useEffect` NOT used for state syncing.
- [ ] **AI (X):** Streaming correctly halts/retries.
- [ ] CLI defect reporting (`antd bug-cli`) executed if anomalies triggered.)
](name: ant-design-expert
description: Decision guide for antd 6.x, Ant Design Pro 5/ProComponents, Ant Design X v2, antd-style, and the offline `@ant-design/cli`. Use for component selection, theming/tokens, SSR, a11y, performance, routing/access/CRUD, AI/chat UI patterns, local API lookup, debugging, migration, E2E testability, and usage analysis.
---

# Ant Design & ProComponents Expert Guide

## S - Scope
- Target: `antd@^6` + React 18-19, coupled with `ant-design-pro@^5`, `@ant-design/pro-components`, `antd-style`, and `@ant-design/x@^2` when needed.
- Tooling: `@ant-design/cli` for offline component metadata, demos, changelogs, migrations, linting, doctor checks, and usage analysis.
- Focus: Architectural decision guidance, precise component/API selection, strictly structured styling, scalable layouts, robust networking, sensible component abstraction, E2E automation readiness, and enterprise-grade UX.
- Source policy: Official docs only; no undocumented APIs, no hallucinated hooks, and strictly NO internal `.ant-*` coupling.

### Default assumptions
- Language: TypeScript.
- Styling: `antd-style` (`createStyles`) FIRST, falling back to Tokens, then `classNames`/`styles`. 
- Provider: One root `ConfigProvider` or `ProConfigProvider` unless strict micro-frontend isolation is required.

### Mandatory rules
- **Pre-flight Check:** Before writing or changing antd component code, query the component API first with `antd info <Component> --format json`. Do not rely on memory.
- **CLI Commands:** Always use `--format json` with `antd` CLI commands. Run `antd lint <changed-path> --format json` post-modification.
- **[CRITICAL] Multi-Theme & Token Strategy:** When customizing styles, future multi-theme adaptation MUST be proactively considered. **Hardcoding hex/rgb color values is STRICTLY FORBIDDEN.** You MUST extract and use Ant Design Design Tokens via `useToken()` or `antd-style`.
- **[CRITICAL] Component Extraction & Anti-Wrappers:** You MUST extract repeated, stateless UI patterns into shared components. However, it is **STRICTLY FORBIDDEN to create "God Wrappers" around complex stateful components like `ProTable` or `ProForm`**. `ProTable` itself IS the abstraction. Wrapping it destroys TS inference, ref forwarding, and slot extensibility.
- **[CRITICAL] E2E Testability (Playwright):** All interactive elements (buttons, links, tabs, critical form inputs) MUST include deterministic `data-testid` attributes to facilitate stable Playwright/E2E automation. **NEVER rely on fragile `.ant-*` class selectors for testing.**
- **[CRITICAL] ProTable Pagination Schema:** For `ProTable` remote requests, ALWAYS explicitly map and rename the default `current` query parameter to `pageNum` within the `request` function.
- **[CRITICAL] Desktop-First Grid Layout:** You MUST use structural grid layouts (`Row`/`Col`, `Space`, or CSS Grid/Flex via `antd-style`). **Assume a Desktop-First environment.** Aggressively IGNORE mobile viewport scaling (`xs`, `sm`) to optimize code density unless explicitly requested.
- **[CRITICAL] Performance & Virtualization:** Assume large datasets. You MUST enable the `virtual` prop for `<Table>`, `<Select>`, and `<Tree>` when >100 items. Memoize complex custom `render` outputs in columns to prevent cascading re-renders.
- **[CRITICAL] Input Debouncing:** For any input or `<Select>` that triggers remote API calls on change (e.g., remote search, async validation), you MUST wrap the fetch logic in a `debounce` function (e.g., via `lodash` or `ahooks`).
- **[CRITICAL] Form Dependency Handling:** STRICTLY FORBIDDEN to use `useEffect` loops to manually sync `ProForm` states. You MUST use `<ProFormDependency>` or the `dependencies` prop for intra-form field reactions.
- **[CRITICAL] Mock Data Strict Isolation:** NEVER import static JSON/arrays directly into components for mock data. All mock data MUST be served via API intercepts (e.g., `/mock` directory) and fetched using the standard `request` utility.
- **[CRITICAL] Visibility-Driven Fetching:** API requests MUST ONLY trigger when the target component is visible. Utilize `useInViewport`, verify active `Tab` keys, and strictly enforce `destroyOnClose` on `Modal`/`Drawer` components.
- **[CRITICAL] Async State Feedback (Loading & Error):** You MUST provide explicit visual feedback for ALL asynchronous operations. Rely on ProComponents' native `loading` states. For custom layouts, you MUST implement `<Skeleton active>` or `<Spin>` during data fetching. Implement graceful degradation for failed API calls using `<Result>`, `message.error`, or Error Boundaries. 
- **[CRITICAL] Button-Level Access Control:** You MUST wrap all data-mutating interactive elements (e.g., "Delete", "Edit" buttons inside `ProTable` action columns) with explicit access checks. NEVER expose mutating actions to unauthorized roles.
- **[CRITICAL] Accessibility (a11y) Compliance:** All interactive elements MUST be accessible. If you render an icon-only button (e.g., `<Button icon={<DeleteOutlined />} />`), you MUST explicitly provide a descriptive `aria-label` or `tooltip` for screen readers.

## P - Process
### 1) Classify & Contextualize
- Identify the abstraction layer: Base antd, ProComponents, `antd-style`, or Ant Design X.
- Confirm rendering mode, data scale, network strategy, and automation test requirements.

### 2) Query Authoritative Sources
- Prefer local `@ant-design/cli` first.
- For CSS-in-JS: Leverage `antd-style`.
- For CRUD/Data: Rely on ProComponents `valueType` dictionary mapping.

### 3) Architectural Decisions
- **Provider Baseline:** CSR -> `ConfigProvider` | SSR -> `ConfigProvider` + `StyleProvider`.
- **ProComponents Strategy:** 
  - Expose ProComponents APIs directly; do not bury them in unnecessary wrapper components.
- **Output Validation:** Define risk vectors (missing `rowKey`, invisible fetching, missing test IDs, missing loading states, missing debouncing, exposed mutating buttons).

## O - Output
- Provide short, authoritative decision rationale (1-3 sentences).
- Output minimal, robust code enforcing Token-first styling, Desktop-First Layout, Visibility-Driven Fetching, graceful error handling, and E2E automation readiness.
- **For Pro:** Dictate route/menu configuration, enforce `request` schemas, and validate component abstraction.

## References
| File | Use when |
| --- | --- |
| `references/antd-cli.md` | You need the exact offline CLI workflow for API lookup, demos, linting, migration, or changelogs. |

## Regression Checklist
- [ ] Only one root `ConfigProvider` / `ProConfigProvider`; SSR style registry verified.
- [ ] **Styling & Theming:** Styles leverage `createStyles` or inline `token`. Zero hardcoded colors.
- [ ] **Architecture:** Stateless UI is extracted. `ProTable` and `ProForm` native APIs remain exposed and typed.
- [ ] **Automation (E2E) & A11y:** `data-testid` explicitly implemented. Icon-only buttons have `aria-label`/tooltips.
- [ ] **Data Fetching:** `ProTable` `current` mapped to `pageNum`. Mock data isolated. Visibility checks (`useInViewport`) implemented.
- [ ] **State & Performance:** Remote search is **debounced**. Explicit `loading` states (`Skeleton`/`Spin`) and graceful error handling (`Result`) are implemented. `virtual` rendering used for lists > 100.
- [ ] **Security:** Data-mutating buttons are wrapped in access controls.
- [ ] **Forms:** `useEffect` NOT used for state syncing.
- [ ] CLI defect reporting (`antd bug-cli`) executed if anomalies triggered.
