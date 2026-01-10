# Sidepanel Design Patterns

Design patterns for collapsible sidepanels in desktop and web applications.

## Design A: Title Bar Toggle (Claude Code Style)

**Best for**: Desktop/windowed applications where you control the window chrome.

```
+------------------------------------------+
| [App Name]           [<] [_] [X]         |  <- Toggle button ON title bar
+----+-------------------------------------+
|    |                                     |
|    |                                     |
| ≡  |     Main Content                    |
|    |                                     |
|    |                                     |
+----+-------------------------------------+
```

### Behavior
- Toggle button (`<` / `>`) in window title bar
- Explicit click to expand/collapse
- Sidebar stays expanded until explicitly collapsed
- Good for frequent sidebar interactions

### Implementation
- Requires custom window chrome (Tauri, Electron)
- Button position: top-right of title bar area
- Animation: slide in/out with 300ms transition

### Pros
- Clear affordance - user knows sidebar can toggle
- Stays expanded during drag operations
- Works well with keyboard navigation

### Cons
- Cannot implement in web apps (no title bar control)
- Requires custom window decoration

---

## Design B: Expand on Hover with Pin (Web App Style)

**Best for**: Web applications where title bar is not accessible.

```
Collapsed (64px):                    Expanded on hover (300px):
+----+---------------------------+   +-------------+-------------------+
|    |                           |   | [Logo] APP [📌] |               |
| ≡  |     Main Content          |   +-------------+-------------------+
|    |                           |   | ≡  LABEL        |               |
+----+---------------------------+   |    Description  | Main Content  |
                                     |    wraps 2 lines|               |
                                     +-------------+-------------------+
```

### Behavior
- Sidebar auto-expands when mouse enters
- Sidebar auto-collapses when mouse leaves (unless pinned)
- Pin button appears in header when expanded
- Pinned state persists to localStorage
- Smooth 300ms transition

### Implementation (React + Tailwind)

```tsx
const [isExpanded, setIsExpanded] = useState(false);
const [isPinned, setIsPinned] = useState(() => {
  if (typeof window !== 'undefined') {
    return localStorage.getItem('sidebar-pinned') === 'true';
  }
  return false;
});

const handleMouseLeave = () => {
  if (!isPinned) setIsExpanded(false);  // Only collapse if not pinned
};

<aside
  className="fixed left-0 top-0 h-full transition-all duration-300"
  style={{ width: isExpanded ? '300px' : '64px' }}
  onMouseEnter={() => setIsExpanded(true)}
  onMouseLeave={handleMouseLeave}
>
  {/* Header with logo and pin button */}
  <div className="h-16 flex items-center gap-3 px-4">
    <img src="/logo.svg" className="w-10 h-10" />
    {isExpanded && (
      <>
        <span className="flex-1">APP NAME</span>
        <button
          onClick={() => {
            const newPinned = !isPinned;
            setIsPinned(newPinned);
            localStorage.setItem('sidebar-pinned', String(newPinned));
          }}
          aria-pressed={isPinned}
          className={\`p-1.5 transition-all duration-200 \${
            isPinned ? 'text-cyan-400 rotate-45' : 'text-gray-500'
          }\`}
          style={isPinned ? { textShadow: '0 0 8px #22d3ee' } : undefined}
        >
          <ThumbtackIcon filled={isPinned} />
        </button>
      </>
    )}
  </div>

  {/* Nav items */}
  {items.map(item => (
    <button className="flex items-center gap-3 px-4 py-3">
      {item.icon}
      {isExpanded && (
        <div className="flex flex-col items-start">
          <span className="text-xs tracking-wider">{item.label}</span>
          <span className="text-xs text-gray-700 leading-tight text-left">
            {item.description}
          </span>
        </div>
      )}
    </button>
  ))}
</aside>
```

### Thumbtack Pin Icon

```tsx
function ThumbtackIcon({ filled }: { filled: boolean }) {
  return (
    <svg
      className={\`w-4 h-4 transition-transform duration-200 \${filled ? 'rotate-45' : ''}\`}
      fill={filled ? 'currentColor' : 'none'}
      stroke="currentColor"
      viewBox="0 0 24 24"
    >
      <path
        strokeLinecap="round"
        strokeLinejoin="round"
        strokeWidth={1.5}
        d="M5 10l1.5-1.5 3 3L12 9V4l4 4h-5l-2.5 2.5 3 3L10 15l-5-5z"
      />
      <path
        strokeLinecap="round"
        strokeLinejoin="round"
        strokeWidth={1.5}
        d="M5 10l-2 2m7 3l-1 7"
      />
    </svg>
  );
}
```

**Pin icon states:**
- Unpinned: Gray outline, upright
- Pinned: Cyan filled, rotated 45°, with glow effect

### Pros
- Works in any web app (no special privileges)
- Intuitive hover-to-reveal pattern
- Clean, minimal collapsed state
- Pin button solves "keep it open" need
- State persists across sessions

### Cons
- May be annoying if frequently hovering accidentally
- Touch devices need alternative (tap to expand)

### Touch Device Adaptation

```tsx
const isTouchDevice = 'ontouchstart' in window;

{isTouchDevice && !isExpanded && (
  <button
    className="absolute right-0 top-1/2 -translate-y-1/2"
    onClick={() => setIsExpanded(true)}
  >
    <ChevronRight />
  </button>
)}
```

---

## Recommendation by Platform

| Platform | Recommended Design |
|----------|-------------------|
| Tauri / Electron | Design A (Title Bar Toggle) |
| Web App (React, Next.js) | Design B (Expand on Hover + Pin) |
| Mobile Web | Design B + tap fallback |
| Progressive Web App | Design B (Expand on Hover + Pin) |

---

## CSS Variables for Theming

```css
:root {
  --sidebar-collapsed-width: 64px;
  --sidebar-expanded-width: 300px;
  --sidebar-transition: 300ms ease-out;
  --sidebar-bg: #000000;
  --sidebar-border: rgba(34, 211, 238, 0.2);
  --sidebar-active-accent: #22d3ee;
  --sidebar-description-color: #374151; /* gray-700 */
}
```

---

## Design Decisions

### Logo
- Transparent background (no bg color)
- Fixed size (w-10 h-10)

### Navigation Items
- Label: uppercase, tracking-wider, mono font
- Description: gray-700 (dimmed), wraps to 2 lines, left-aligned
- Active indicator: left border bar with cyan glow

### Pin Button
- Position: top-right of header (next to app name)
- Only visible when sidebar is expanded
- Thumbtack icon (not bookmark) for clear "pin" metaphor
- Fills when active for visual feedback
- localStorage key: \`{app}-sidepanel-pinned\`

---

## Reference Implementation

See Pathalyzer's CaseSidepanel component:
- File: \`components/case/CaseSidepanel.tsx\`
- Tests: \`components/case/CaseSidepanel.test.tsx\` (19 tests)
- Features: 3-panel nav, pin button, localStorage persistence
