---
name: overlay-design-patterns
description: Design patterns for dropdowns, modals, and overlay components with backdrop dismiss using Solid.js
---

# Overlay Design Patterns

Reference patterns for implementing dropdowns, modals, and overlays in this codebase.

## 1. Dropdown Menu with Chevron

Model selector dropdown with rotating chevron indicator and backdrop dismiss.

**Implementation** (`ComparisonPane.jsx:33-232`):
```jsx
const [dropdownOpen, setDropdownOpen] = createSignal(false);

<button
  onClick={() => setDropdownOpen(!dropdownOpen())}
  class="flex items-center gap-1.5 px-2.5 py-1 rounded-lg bg-gray-100 dark:bg-gray-700 hover:bg-gray-200 dark:hover:bg-gray-600 transition-colors"
>
  <span class="text-sm font-medium">{selectedModel().name}</span>
  <svg class={`w-3.5 h-3.5 transition-transform ${dropdownOpen() ? 'rotate-180' : ''}`}>
    <!-- chevron-down icon -->
  </svg>
</button>

<Show when={dropdownOpen()}>
  {/* Backdrop to close on outside click */}
  <div class="fixed inset-0 z-40" onClick={() => setDropdownOpen(false)} />

  {/* Dropdown menu */}
  <div class="absolute left-0 top-full mt-1 z-50 min-w-[280px] bg-white dark:bg-gray-800 rounded-xl shadow-lg border animate-dropdown">
    {/* Menu items */}
  </div>
</Show>
```

**How It Works:**
- **Chevron rotation**: `rotate-180` class flips the arrow when open
- **Invisible backdrop**: Full-screen `div` at `z-40` catches outside clicks
- **Menu above backdrop**: Menu at `z-50` stays clickable
- **Animation**: `animate-dropdown` provides subtle entrance effect

**Key Points:**
- Chevron rotates 180deg when open: `${dropdownOpen() ? 'rotate-180' : ''}`
- Invisible backdrop catches outside clicks to close
- Uses `animate-dropdown` CSS animation

---

## 2. Modal Overlay

Full-screen modal with visible backdrop and centered content.

**Implementation** (`App.jsx:874-880`):
```jsx
const [showSettings, setShowSettings] = createSignal(false);

<Show when={showSettings()}>
  <div class="fixed inset-0 z-50 flex items-center justify-center">
    {/* Visible backdrop - dims the page */}
    <div
      class="absolute inset-0 bg-black/50"
      onClick={() => setShowSettings(false)}
    />
    {/* Modal content - relative lifts it above backdrop */}
    <div class="relative bg-white dark:bg-gray-800 rounded-2xl max-w-md w-full mx-4">
      {/* Content */}
    </div>
  </div>
</Show>
```

**How It Works:**
- **Visible backdrop**: `bg-black/50` dims the page (50% opacity black)
- **Centered content**: Flexbox centers the modal
- **Stacking order**: `relative` on modal lifts it above `absolute` backdrop
- **Click to dismiss**: Clicking backdrop (not modal) closes it

---

## CSS Animations

**Dropdown Animation** (`index.css:254-268`):
```css
@keyframes dropdownIn {
  from {
    opacity: 0;
    transform: translateY(-4px) scale(0.98);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.animate-dropdown {
  animation: dropdownIn 0.15s ease-out;
}
```

---

## Pattern Summary

| Pattern | Backdrop | Z-Index | Animation | Close Trigger |
|---------|----------|---------|-----------|---------------|
| Dropdown | Invisible (`fixed inset-0`) | 40/50 | `dropdownIn` keyframe | Outside click |
| Modal | Visible (`bg-black/50`) | 50 | Fade-in | Backdrop click |

## Best Practices

1. **Use backdrop overlays** for outside click detection (simpler than event listeners)
2. **Layer z-index correctly**: backdrop below, content above
3. **Rotate chevron icons** to indicate open/closed state
4. **Keep animations short** (150ms) for responsive feel
5. **Use `relative`** on modal content to lift above `absolute` backdrop

## Related

- [Sidepanel Design Patterns](sidepanel-design-patterns.md) - Collapse/expand buttons
