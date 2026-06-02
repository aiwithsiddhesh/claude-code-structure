---
paths:
  - "src/**/*.{ts,tsx,jsx,js}"
  - "pages/**/*.{ts,tsx}"
  - "app/**/*.{ts,tsx}"
  - "components/**/*.{ts,tsx}"
  - "output/**/src/**/*.{ts,tsx,jsx,js}"
  - "output/**/pages/**/*.{ts,tsx}"
  - "output/**/app/**/*.{ts,tsx}"
  - "output/**/components/**/*.{ts,tsx}"
---

# Frontend Patterns

Loaded when working with frontend source files.

## Component Architecture

**File-per-component rule**: One React component per file. The file name matches the component name.

**Component anatomy** (in order):
```tsx
// 1. Imports
// 2. Types / interfaces
// 3. Constants (if local to component)
// 4. Component function
// 5. Sub-components (if small and tightly coupled)
// 6. Default export
```

**Props pattern**:
```tsx
interface UserCardProps {
  userId: string;
  onSelect?: (id: string) => void; // optional callbacks use ?
}

export function UserCard({ userId, onSelect }: UserCardProps) { ... }
```

- Destructure props in the function signature.
- Define prop types as a named interface directly above the component.
- Export the interface if it's used elsewhere.

## Folder Structure

```
src/
├── components/     # Shared, reusable UI components
│   └── UserCard/
│       ├── UserCard.tsx
│       ├── UserCard.test.tsx
│       └── index.ts          # re-export only
├── features/       # Feature-specific components and logic
│   └── auth/
│       ├── LoginForm.tsx
│       ├── use-auth.ts
│       └── auth.service.ts
├── hooks/          # Shared custom hooks (use-*.ts)
├── lib/            # Utilities, helpers, shared logic
├── types/          # Shared TypeScript types and interfaces
└── api/            # API client functions
```

## State Management

- **Local state first** — `useState` before reaching for external state.
- **Server state** — use React Query (`@tanstack/react-query`) for all API data. Do not store API responses in `useState` or `useContext` manually.
- **Global client state** — Zustand or React Context (for low-frequency updates like theme, auth). Not Redux unless already established.
- **Derived state is not state** — compute it in the render, not in `useEffect` + `useState`.
- `useEffect` only for: subscriptions, DOM manipulation, and syncing with external systems. Not for data transformations.

## API Integration

```tsx
// Use React Query — always handle all three states
const { data, isLoading, error } = useQuery({
  queryKey: ['user', userId],
  queryFn: () => fetchUser(userId),
});

if (isLoading) return <Skeleton />;
if (error) return <ErrorMessage error={error} />;
return <UserCard user={data} />;
```

- Every data-fetching component renders a loading state, error state, and empty state.
- API functions live in `src/api/` — not inline in components.
- API functions return typed responses, never `any`.

## Tailwind CSS Usage

- **Mobile-first**: base styles apply to mobile, `sm:` / `md:` / `lg:` override upward.
- **No inline styles** unless dynamically computed (e.g., chart dimensions).
- **Component variants**: use `cva` (class-variance-authority) for components with multiple visual states.
- **No arbitrary values** (`w-[347px]`) unless genuinely necessary and documented with a comment.
- Extract repeated class combinations into a component, not a CSS class.

## Forms

- Use `react-hook-form` for all forms. Not controlled components with manual state.
- Validation schema with Zod, connected to react-hook-form via `@hookform/resolvers/zod`.
- Display field-level errors adjacent to the field, not in a banner.
- Disable the submit button while submitting (not after — prevents double-submit confusion).
- Reset the form on successful submission, not on mount.

## Accessibility (Baseline)

- All interactive elements are keyboard-reachable.
- All images have `alt` text. Decorative images: `alt=""`.
- Form inputs have associated `<label>` (via `htmlFor` + `id`, or `aria-label`).
- Color is never the only way to convey information.
- Use `<button>` for actions, `<a>` for navigation. Never `<div onClick>`.
