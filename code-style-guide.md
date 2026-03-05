---
trigger: always_on
---

# Next.js Project — Best Practices & Conventions

## Related Architecture Rules

This rule covers coding standards, conventions, and tooling. For architectural decisions, refer to the dedicated rules:

- **`fsd-folder-structure`** — FSD layer responsibilities and folder structure
- **`server-client-components`** — When and how to use Server vs Client Components
- **`data-fetching-and-mutations`** — Data fetching, Server Actions, and Route Handlers
- **`caching-public-api-dependencies`** — Cache tags, public APIs, and dependency direction

Do not duplicate or override rules defined in those files.

---

## Development Philosophy

- Write clean, maintainable, and scalable code
- Follow SOLID principles
- Prefer functional and declarative programming patterns over imperative
- Emphasize type safety and static analysis
- Practice component-driven development

---

## Planning Phase

- Begin with step-by-step planning
- Write detailed pseudocode before implementation
- Document component architecture and data flow
- Consider edge cases and error scenarios upfront

---

## Code Style

- Use tabs for indentation
- Use single quotes for strings (except to avoid escaping)
- Omit semicolons (unless required for disambiguation)
- Eliminate unused variables
- Add space after keywords and before function declaration parentheses
- Always use strict equality (`===`) instead of loose equality (`==`)
- Space infix operators; add space after commas
- Keep `else` on the same line as the closing curly brace
- Use curly braces for multi-line `if` statements
- Always handle error parameters in callbacks
- Limit line length to 80 characters
- Use trailing commas in multiline object/array literals

---

## Naming Conventions

### Casing rules

| Pattern    | Use for                                          |
| ---------- | ------------------------------------------------ |
| PascalCase | Components, type definitions, interfaces         |
| kebab-case | Directory names, file names (`user-profile.tsx`) |
| camelCase  | Variables, functions, methods, hooks, props      |
| UPPERCASE  | Environment variables, constants, global config  |

### Specific patterns

- Prefix event handlers with `handle`: `handleClick`, `handleSubmit`
- Prefix booleans with verbs: `isLoading`, `hasError`, `canSubmit`
- Prefix custom hooks with `use`: `useAuth`, `useForm`
- Use complete words over abbreviations, except: `err`, `req`, `res`, `props`, `ref`

---

## React Best Practices

- Use functional components with TypeScript interfaces
- Define components using the `function` keyword
- Extract reusable logic into custom hooks
- Use `React.memo()` strategically, not by default
- Always clean up side effects in `useEffect`
- Use `useCallback` for memoizing callbacks passed as props
- Use `useMemo` for expensive computations
- Avoid inline function definitions in JSX
- Use dynamic imports for code splitting
- Use stable, meaningful `key` props in lists — never array index

---

## Next.js Conventions

- Use the built-in `<Image>` component for all images
- Use the built-in `<Link>` component for internal navigation
- Use the `<Script>` component for external scripts
- Implement proper `metadata` exports for SEO
- Use `loading.tsx` and `error.tsx` boundaries intentionally

> For Server vs Client Component decisions, see `server-client-components`.
> For data fetching patterns, see `data-fetching-and-mutations`.

---

## TypeScript

- Enable strict mode in `tsconfig.json`
- Define clear interfaces for component props, state, and API responses
- Use type guards to handle `undefined` or `null` safely
- Apply generics where type flexibility is needed
- Use TypeScript utility types (`Partial`, `Pick`, `Omit`) for cleaner reuse
- Prefer `interface` over `type` for object shapes, especially when extending
- Use mapped types for dynamic type variations

---

## UI and Styling

- Use **Tailwind CSS** for all styling — utility-first approach
- Use **Shadcn UI** for consistent, accessible component design
- Use **Radix UI** primitives for customizable, accessible elements
- Design mobile-first; ensure responsiveness across breakpoints
- Implement dark mode using CSS variables or Tailwind's dark mode
- Ensure color contrast meets WCAG accessibility standards
- Define CSS variables for theme colors and spacing
- Maintain consistent spacing values throughout the UI

---

## State Management

### Local state

- `useState` for simple component-level state
- `useReducer` for complex or multi-step state
- `useContext` for shared state within a subtree

### Global state (Redux Toolkit)

- Use `createSlice` to define state, reducers, and actions together
- Avoid `createReducer` and `createAction` unless necessary
- Normalize state to avoid deeply nested data
- Use selectors to encapsulate all state access
- Separate slices by feature — avoid large all-encompassing slices

---

## Error Handling and Validation

- Use **Zod** for all schema validation
- Use **React Hook Form** for form state and submission
- Use error boundaries to catch and handle errors gracefully
- Log caught errors to an external service (e.g., Sentry)
- Design user-friendly fallback UIs for error states

---

## Testing

- Write unit tests for all individual functions and components
- Use **Jest** and **React Testing Library**
- Follow the Arrange-Act-Assert pattern
- Mock external dependencies and API calls to isolate tests
- Focus integration tests on user workflows
- Set up and tear down test environments properly
- Use snapshot testing selectively — not as a default

---

## Accessibility (a11y)

- Use semantic HTML for meaningful structure
- Apply accurate ARIA attributes where needed
- Ensure full keyboard navigation support
- Manage focus order and visibility effectively
- Follow a logical heading hierarchy
- Make all interactive elements accessible
- Provide clear, accessible error feedback

---

## Security

- Sanitize all user input to prevent XSS attacks
- Use DOMPurify for sanitizing HTML content
- Use proper authentication methods (never roll your own)

---

## Internationalization (i18n)

- Use `next-i18next` or `next-intl` for translations
- Implement proper locale detection
- Use proper number, date, and currency formatting per locale
- Implement RTL support where needed
- Dont add string directly, create transalation message, and namespace if it is not declared.

---

## Documentation

- Use JSDoc for all public functions, classes, methods, and interfaces
- Add usage examples when behavior is non-obvious
- Write complete sentences with proper punctuation
- Keep descriptions clear and concise
