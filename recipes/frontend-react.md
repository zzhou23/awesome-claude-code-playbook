# Claude Code Recipe: Frontend (React / Next.js)

> Turn Claude Code into a senior React developer that writes accessible, tested, and performant components from day one.

## Quick Start

### CLAUDE.md

Drop this into your project root. Adjust paths and libraries to match your stack.

```markdown
# Project: [Your App Name]

## Tech Stack
- Next.js 14+ (App Router)
- React 18+
- TypeScript (strict mode)
- Tailwind CSS
- Vitest + React Testing Library

## Project Structure
```
src/
  app/           # Next.js App Router pages and layouts
  components/
    ui/          # Primitive UI components (Button, Input, Modal)
    features/    # Feature-specific components (UserProfile, Dashboard)
  hooks/         # Custom React hooks
  lib/           # Utilities, API clients, constants
  types/         # Shared TypeScript types and interfaces
  styles/        # Global styles, Tailwind config extensions
```

## Component Conventions
- Functional components only, no class components
- Named exports (not default exports): `export function Button() {}`
- Props defined as TypeScript interface above the component
- One component per file, max 200 lines
- Colocate component, test, and styles: `Button.tsx`, `Button.test.tsx`
- Custom hooks start with `use` prefix and live in `hooks/`

## Styling Rules
- Use Tailwind CSS utility classes
- Extract repeated patterns into components, not @apply
- Responsive: mobile-first (`sm:`, `md:`, `lg:`, `xl:`)
- Breakpoints: sm=640px, md=768px, lg=1024px, xl=1280px
- Dark mode via `dark:` variant (class strategy)

## TypeScript Rules
- No `any` type — use `unknown` and narrow, or define a proper type
- Prefer `interface` for object shapes, `type` for unions/intersections
- All event handlers typed explicitly (e.g., `React.MouseEvent<HTMLButtonElement>`)
- API responses must have defined types, never trust raw JSON

## Testing Rules
- Test framework: Vitest + @testing-library/react
- Test file naming: `ComponentName.test.tsx`
- Test behavior, not implementation details
- Use `screen.getByRole()` over `getByTestId()` whenever possible
- Every component must have at least: render test, interaction test, edge case test

## Key Rules
- NEVER use `useEffect` for data fetching — use server components or React Query
- NEVER mutate state directly — always return new objects/arrays
- Prefer server components by default, add "use client" only when needed
- Images must use `next/image` with width/height or fill
- Links must use `next/link`
- All interactive elements must be keyboard accessible
```

### settings.json

Save as `.claude/settings.json` in your project root:

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run test:*)",
      "Bash(npm test -- --reporter=verbose)",
      "Bash(npm run lint)",
      "Bash(npm run lint:fix)",
      "Bash(npm run build)",
      "Bash(npx tsc --noEmit)",
      "Bash(npx vitest run *)",
      "Bash(npx vitest run --coverage)",
      "Bash(npx next lint)",
      "Bash(npx playwright test *)"
    ],
    "deny": [
      "Bash(npm install *)",
      "Bash(npm uninstall *)",
      "Bash(npx create-*)",
      "Bash(rm -rf *)"
    ]
  }
}
```

> **Why deny `npm install`?** You want to review every new dependency before it enters your bundle. Claude should propose the dependency; you approve and install it.

### Recommended MCP Servers

- [modelcontextprotocol/server-puppeteer](https://github.com/modelcontextprotocol/servers/tree/main/src/puppeteer) - Browser automation and screenshot capture. **When to use:** Visual regression testing, verifying rendered output matches design specs, debugging layout issues.

- [modelcontextprotocol/server-fetch](https://github.com/modelcontextprotocol/servers/tree/main/src/fetch) - Fetches web content with robots.txt compliance. **When to use:** Reading component library docs (Radix, shadcn/ui, Headless UI), checking MDN references, pulling API specs.

- [AeroNotix/mcp-server-playwright](https://github.com/AeroNotix/mcp-server-playwright) - Cross-browser testing via Playwright. **When to use:** Running E2E tests across Chrome, Firefox, and Safari; verifying responsive behavior at specific viewport sizes.

- [modelcontextprotocol/server-filesystem](https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem) - Secure file access outside the working directory. **When to use:** Reading design tokens from a shared design system package, accessing a monorepo's shared config.

## Recommended Workflow

### 1. Scaffold the component

Describe the component in plain language. Be specific about props, behavior, and edge cases.

```
Create a SearchAutocomplete component that takes an async onSearch prop returning string[],
debounces input by 300ms, shows a dropdown of results, supports keyboard navigation
(arrow keys + enter), and closes on Escape or outside click. Use Tailwind for styling.
```

Claude will generate the component file, TypeScript interface, and basic structure. Review the prop interface before moving on.

### 2. Write tests first, then implement

Ask Claude to write tests before the implementation is complete. This catches design issues early.

```
Write tests for SearchAutocomplete covering: empty state, typing triggers debounced search,
results display, keyboard navigation between items, selecting an item, escape closes dropdown,
clicking outside closes dropdown. Use Testing Library with user-event.
```

Then ask Claude to make all tests pass. This TDD loop produces cleaner APIs than implementation-first.

### 3. Style and make responsive

Once behavior is solid, focus on visual polish. Provide explicit constraints.

```
Style the SearchAutocomplete for all breakpoints. On mobile (<640px), the dropdown should be
full-width. On desktop, max-width 480px. Add loading spinner during search, empty state message
when no results, and dark mode support.
```

### 4. Audit accessibility

Ask Claude to check against WCAG guidelines. This is where many teams skip, but Claude is thorough.

```
Audit SearchAutocomplete for accessibility. Check: ARIA roles and attributes, focus management,
screen reader announcements for results count, color contrast in both light and dark mode,
keyboard-only operability.
```

### 5. Integrate and review

Wire the component into its page, test the integration, and get a final review.

```
Add SearchAutocomplete to the /dashboard page, connected to the products API endpoint.
Then review the full component for performance issues, unnecessary re-renders, and bundle size impact.
```

## Tips and Pitfalls

1. **Declare your design system in CLAUDE.md.** If you use shadcn/ui, Radix, MUI, or Ant Design, say so explicitly. Otherwise Claude will write raw HTML or pick its own library. A single line like `UI library: shadcn/ui (do NOT install alternatives)` saves hours.

2. **Pin your UI library versions.** Claude knows multiple versions of popular libraries. If your CLAUDE.md says `"@radix-ui/react-dialog": "1.0.x"`, Claude will generate code for that API, not the latest breaking version.

3. **Describe visuals with words, not just screenshots.** Claude works best when you describe layout as structure: "Two-column layout, sidebar 280px fixed, main content fluid, sidebar collapses to hamburger below md breakpoint." Combine with screenshots from a Puppeteer MCP for best results.

4. **Block unsupervised dependency installs.** Put `npm install` in the deny list (see settings.json above). Claude will sometimes try to `npm install` a utility package to solve a problem that a 5-line helper function would handle. Review every addition.

5. **Keep component files under 200 lines.** If Claude generates a component exceeding this, ask it to extract sub-components or custom hooks. Large components are a sign of mixed responsibilities.

6. **Specify state management upfront.** Whether you use React Context, Zustand, Jotai, or server state with React Query — state it in CLAUDE.md. Without guidance, Claude will default to `useState` / `useReducer` for everything, including data that belongs in a server cache.

7. **Always ask for loading, error, and empty states.** Claude will build the happy path perfectly and skip edge cases unless prompted. Add to your CLAUDE.md: `Every data-fetching component must handle: loading, error, empty, and success states.`

8. **Use explicit type boundaries for API data.** Tell Claude to define response types and validate at the boundary. Raw `fetch` calls with no type validation are a common source of runtime errors that TypeScript alone cannot catch.

## Starter Prompt Templates

### Create a new component

```
Create a [ComponentName] component in src/components/[ui|features]/.

Props:
- [propName]: [type] — [description]
- [propName]: [type] — [description]

Behavior:
- [Describe interactions, state changes, side effects]

Constraints:
- Use our existing [Button/Input/etc.] components from src/components/ui/
- Mobile-first responsive with Tailwind
- Must be keyboard accessible
- Include loading, error, and empty states

Write the component and its test file.
```

### Refactor a class component or complex component to hooks

```
Refactor src/components/features/[ComponentName].tsx:

1. Extract the [describe logic] into a custom hook `use[HookName]`
2. Move the hook to src/hooks/use[HookName].ts
3. Keep the component under 150 lines after extraction
4. Preserve all existing behavior — do not change any props or rendered output
5. Update tests to cover the hook independently

Run existing tests after refactoring to confirm nothing broke.
```

### Add tests to existing components

```
Add comprehensive tests for src/components/features/[ComponentName].tsx.

Cover these scenarios:
1. Renders correctly with required props only
2. Renders correctly with all optional props
3. [Specific user interaction] works correctly
4. Handles error state when [condition]
5. Shows empty state when [condition]
6. Keyboard navigation works (Tab, Enter, Escape as applicable)
7. ARIA attributes are present and correct

Use @testing-library/react with userEvent. Prefer getByRole over getByTestId.
Run all tests and confirm they pass.
```

### Performance optimization

```
Analyze src/components/features/[ComponentName].tsx for performance issues.

Check for:
1. Unnecessary re-renders (missing React.memo, unstable references in props)
2. Expensive computations that should use useMemo
3. Event handlers recreated on every render (need useCallback)
4. Large component trees that should use lazy loading
5. Images without proper sizing or lazy loading
6. Bundle size impact of imported libraries

Apply fixes, then verify with existing tests. Explain each optimization and its expected impact.
```

### Accessibility audit

```
Audit src/components/features/[ComponentName].tsx for WCAG 2.1 AA compliance.

Check:
1. All interactive elements have accessible names
2. Focus order is logical and visible
3. ARIA roles, states, and properties are correct
4. Color contrast meets 4.5:1 for text, 3:1 for large text
5. Component is fully operable with keyboard only
6. Screen reader announcements for dynamic content
7. No content is conveyed by color alone

Fix any violations found. Add aria-* attributes and role props as needed.
Run tests after changes.
```
