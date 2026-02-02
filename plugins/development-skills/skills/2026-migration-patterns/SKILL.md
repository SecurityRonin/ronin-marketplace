---
name: 2026-migration-patterns
description: Use when upgrading Zod 3→4, Vitest 2→4, Tailwind CSS 3→4, Pinecone 6→7, or AI SDK. Covers breaking API changes, type errors, mock patterns, and CSS-first config migration.
---

# 2026 Major Library Migration Patterns

Reference for breaking changes discovered during dependency upgrades (Feb 2026). Hard-won fixes from real production migrations.

---

## Zod 3 → 4

### `.errors` renamed to `.issues`

The ZodError property was renamed for clarity.

```typescript
// Before (Zod 3)
result.error.errors.map(e => e.message)

// After (Zod 4)
result.error.issues.map(e => e.message)
```

**Find and fix:**
```bash
rg "\.error\.errors" --type ts
```

### `z.record()` requires two arguments

Records now require explicit key type.

```typescript
// Before (Zod 3)
z.record(z.unknown())

// After (Zod 4)
z.record(z.string(), z.unknown())
```

**Find and fix:**
```bash
rg "z\.record\(z\." --type ts
```

### Enum keys in records require ALL keys present

This is a significant behavioral change. In Zod 3, partial records were allowed with enum keys. Zod 4 enforces completeness.

```typescript
// Zod 3: Partial records worked
const schema = z.record(z.enum(['a', 'b', 'c']), z.string())
schema.parse({ a: 'foo' }) // OK - only 'a' provided

// Zod 4: ALL enum keys must be present
schema.parse({ a: 'foo' }) // ERROR - missing 'b' and 'c'

// Fix: Use z.string() for keys if partial is needed
const schema = z.record(z.string(), z.string())
```

### `SafeParseReturnType` type moved

The type export location changed and may not be directly accessible.

```typescript
// Before (Zod 3)
function validate(): z.SafeParseReturnType<unknown, T> { }

// After (Zod 4) - use ReturnType instead
function validate(): ReturnType<typeof schema.safeParse> { }
```

---

## Vitest 2 → 4

### Mock constructors must use regular functions

**This is the #1 migration issue.** Arrow functions cannot be used with `new`. Vitest 4 enforces this strictly and will throw:

```
TypeError: () => ({...}) is not a constructor
```

```typescript
// Before (Vitest 2) - worked but technically incorrect
vi.mock("module", () => ({
  MyClass: vi.fn().mockImplementation(() => ({
    method: vi.fn()
  }))
}))

// After (Vitest 4) - must use regular function
vi.mock("module", () => ({
  MyClass: vi.fn().mockImplementation(function(this: any) {
    this.method = vi.fn()
  })
}))
```

### Common globals that need this fix

These are often mocked in test setup files (`setup.ts`):

```typescript
// test/setup.ts - Vitest 4 compatible

// ResizeObserver
global.ResizeObserver = vi.fn().mockImplementation(function(
  this: { observe: () => void; unobserve: () => void; disconnect: () => void }
) {
  this.observe = vi.fn();
  this.unobserve = vi.fn();
  this.disconnect = vi.fn();
});

// IntersectionObserver
global.IntersectionObserver = vi.fn().mockImplementation(function(
  this: { observe: () => void; unobserve: () => void; disconnect: () => void }
) {
  this.observe = vi.fn();
  this.unobserve = vi.fn();
  this.disconnect = vi.fn();
});

// MutationObserver (same pattern)
```

### Mock type casting for strict assignments

Vitest 4 has stricter mock types. When assigning mocks to native APIs:

```typescript
// Before - type error in Vitest 4
console.log = mockConsole.log

// After - explicit cast
console.log = mockConsole.log as typeof console.log
```

---

## Tailwind CSS 3 → 4

### CSS-first configuration

The JavaScript config file (`tailwind.config.ts`) is **removed** and replaced with CSS directives.

```css
/* globals.css - Tailwind v4 */
@import 'tailwindcss';

@custom-variant dark (&:is(.dark *));

@theme {
  --font-sans: var(--font-inter), sans-serif;
  --color-background: hsl(var(--background));
  --color-foreground: hsl(var(--foreground));
  --color-primary: hsl(var(--primary));
  --color-primary-foreground: hsl(var(--primary-foreground));
  /* ... all theme variables */
}
```

### PostCSS config update

```javascript
// postcss.config.mjs
const config = {
  plugins: {
    '@tailwindcss/postcss': {},  // NOT 'tailwindcss'
  },
};
export default config;
```

### Migration tool

**Requires clean git state** (no uncommitted changes):

```bash
npx @tailwindcss/upgrade
```

The tool:
1. Converts `tailwind.config.ts` to CSS `@theme` blocks
2. Updates PostCSS config
3. Migrates utility class names
4. Deletes the old config file

### Class changes to watch for

- `focus-visible:outline-none` → `focus-visible:outline-hidden`
- Custom variants may need `@custom-variant` declarations
- Some utility classes may have changed names

---

## Pinecone SDK 6 → 7

### Upsert API change

The upsert method now takes an options object instead of an array directly.

```typescript
// Before (v6)
await index.namespace("ns").upsert(vectors)

// After (v7)
await index.namespace("ns").upsert({ records: vectors })
```

**Find and fix:**
```bash
rg "\.upsert\(" --type ts
```

---

## AI SDK (Vercel) 3.x

### Transport-based chat

The useChat hook now uses a transport layer for communication.

```typescript
import { DefaultChatTransport } from "ai"
import { useChat } from "@ai-sdk/react"

const transport = new DefaultChatTransport({
  api: "/api/chat"
})

const { messages, sendMessage } = useChat({ transport })
```

### Parts-based messages

Messages now use a parts array instead of a content string.

```typescript
// Old format
{ role: "user", content: "Hello" }

// New format (v3.x)
{
  role: "user",
  parts: [{ type: "text", text: "Hello" }]
}
```

This enables multimodal messages with different part types (text, image, tool calls, etc.).

---

## Migration Checklist

Use this checklist when upgrading packages:

### Zod 3 → 4
- [ ] `rg "\.error\.errors"` → change to `.issues`
- [ ] `rg "z\.record\(z\."` → ensure 2 args (add `z.string()` as first)
- [ ] Check for `z.record(z.enum(...))` patterns → may need redesign
- [ ] `rg "SafeParseReturnType"` → use `ReturnType<typeof schema.safeParse>`

### Vitest 2 → 4
- [ ] `rg "mockImplementation\(\(\)"` → use `function()` for constructors
- [ ] Check `test/setup.ts` for observer mocks
- [ ] Add type casts for mock assignments to native APIs

### Tailwind CSS 3 → 4
- [ ] Commit all changes (clean git required)
- [ ] Run `npx @tailwindcss/upgrade`
- [ ] Review generated `@theme` block in globals.css
- [ ] Verify PostCSS config uses `@tailwindcss/postcss`
- [ ] Delete `tailwind.config.ts` if not auto-deleted

### Pinecone 6 → 7
- [ ] `rg "\.upsert\("` → wrap arrays in `{ records: }`

### After all migrations
- [ ] Run full test suite
- [ ] Run production build
- [ ] Deploy and verify

---

## Debugging Tips

### Zod validation errors
```typescript
const result = schema.safeParse(data);
if (!result.success) {
  console.log(JSON.stringify(result.error.issues, null, 2));
}
```

### Vitest mock issues
If you see `is not a constructor`, the mock is using an arrow function. The error message includes the arrow function source.

### Tailwind class not working
Check if the class was renamed in v4. Use browser devtools to see what CSS is generated.
