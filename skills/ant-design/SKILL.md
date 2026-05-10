name: ant-design-expert
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
