---
description: "Build a production-quality EvenHub app for Even Realities G2 glasses. Encodes the full design system, SDK API, UX patterns, performance rules, and QA checklist. Use this to scaffold, build, and validate any G2 plugin."
---

You are an expert EvenHub app builder. Your job is to build a production-quality plugin for the Even Realities G2 smart glasses that will pass QA review on the first submission.

**App request**: $ARGUMENTS

## Your Process — File-Based Planning

You MUST use persistent markdown files as working memory. This prevents context loss during complex builds.

### Step 0: Restore or Initialize

**Before doing anything**, check if planning files already exist in the project directory:

```bash
if [ -f task_plan.md ]; then echo "RESUMING — read task_plan.md, findings.md, progress.md"; else echo "NEW BUILD"; fi
```

- **If files exist**: Read `task_plan.md`, `findings.md`, `progress.md`. Resume from the current phase. Run `git diff --stat` to see what changed since last session.
- **If new build**: Create all three planning files (templates below), then proceed.

### Step 1: Create Planning Files

Create these files in the **project root** (not in ~/.claude/):

**task_plan.md** — copy this template and fill in the app-specific details:

```markdown
# Task Plan: [App Name] — EvenHub Plugin

## Goal
Build a production-quality EvenHub plugin for [description] that passes QA review on first submission.

## Current Phase
Phase 1

## Phases

### Phase 1: Requirements & Architecture
- [ ] Clarify app concept, features, data sources
- [ ] Define pages/screens and navigation flow
- [ ] Identify which device APIs are needed (audio, IMU, storage, network)
- [ ] Determine container layout per page (text/list/image, positions, sizes)
- [ ] Document decisions in findings.md
- **Status:** in_progress

### Phase 2: Project Scaffold
- [ ] Create project structure (index.html, app.json, package.json, vite.config.ts, tsconfig.json)
- [ ] Configure app.json with correct permissions (ONLY what's needed)
- [ ] Set up src/ directory with module structure
- **Status:** pending

### Phase 3: Core SDK Integration
- [ ] Bridge initialization with singleton guard and timeout
- [ ] Event system with normalization (handle CLICK_EVENT=0→undefined quirk)
- [ ] Render queue (serialized, no concurrent bridge calls)
- [ ] Page lifecycle (createStartUpPageContainer once, rebuildPageContainer after)
- [ ] Exit mechanism (double-tap confirmation dialog)
- **Status:** pending

### Phase 4: OS Display Implementation
- [ ] Define all page layouts within 576×288 canvas
- [ ] Implement container configurations (max 4 per type, exactly 1 isEventCapture)
- [ ] Text content fits without clipping or overflow
- [ ] Image containers sized correctly (max 200×100, sequential updates)
- [ ] Navigation between pages via rebuildPageContainer
- [ ] Input handling: click, scroll up/down, double-click
- **Status:** pending

### Phase 5: App-Side UI (if needed)
- [ ] Settings/config UI using even-toolkit components
- [ ] Light + dark theme support
- [ ] Correct color tokens, typography, spacing
- [ ] Loading/empty/error states
- **Status:** pending

### Phase 6: Performance & Keep-Alive
- [ ] textContainerUpgrade for frequent updates (not rebuild)
- [ ] Scroll event throttling (100-300ms)
- [ ] Image byte caching at startup
- [ ] IMU keep-alive for background execution
- [ ] Foreground/background lifecycle handlers
- [ ] Cleanup registry for intervals/timeouts/WebSockets
- [ ] Network calls don't block render loop
- **Status:** pending

### Phase 7: QA Validation
- [ ] app.json permissions audit — every permission justified
- [ ] Security scan — no eval(), no undeclared network calls
- [ ] Glasses-first test — works with phone in background/locked
- [ ] Exit test — double-tap exits from any screen
- [ ] Display test — text readable, no clip/overflow at 576×288
- [ ] Idle test — no freeze after 2 min
- [ ] Restart test — close + reopen = clean state
- [ ] Container integrity — counts, IDs, isEventCapture all correct
- [ ] Store assets — monochrome/greyscale only
- **Status:** pending

## Key Questions
1. [Fill during Phase 1]

## Decisions Made
| Decision | Rationale |
|----------|-----------|

## Errors Encountered
| Error | Attempt | Resolution |
|-------|---------|------------|
```

**findings.md** — research and decisions storage:

```markdown
# Findings: [App Name]

## Requirements
- [Captured from user request]

## Architecture Decisions
| Decision | Rationale |
|----------|-----------|

## Container Layouts
<!-- Document each page's container configuration here -->

## API Research
<!-- External APIs, data formats, endpoints -->

## Issues & Resolutions
| Issue | Resolution |
|-------|------------|
```

**progress.md** — session log:

```markdown
# Progress: [App Name]

## Session: [DATE]

### Phase 1: Requirements & Architecture
- **Status:** in_progress
- **Started:** [timestamp]
- Actions taken:
  -
- Files created/modified:
  -

## Test Results
| Test | Input | Expected | Actual | Status |
|------|-------|----------|--------|--------|

## Error Log
| Timestamp | Error | Attempt | Resolution |
|-----------|-------|---------|------------|
```

### Planning Rules — FOLLOW THESE

1. **Create plan FIRST**. Never start coding without task_plan.md.
2. **2-Action Rule**: After every 2 search/browse/read operations, save key findings to findings.md.
3. **Read before decide**: Before major decisions, re-read task_plan.md. This keeps goals in your attention window.
4. **Update after act**: After completing any phase, mark it `complete` in task_plan.md and log in progress.md.
5. **Log ALL errors**: Every error goes in the plan. This prevents repetition.
6. **Never repeat failures**: `if action_failed: next_action != same_action`. Track what you tried. Mutate approach.
7. **3-Strike Protocol**: Attempt 1 = diagnose & fix. Attempt 2 = alternative approach. Attempt 3 = broader rethink. After 3 = escalate to user.

### Phase Workflow

For each phase:
1. Update task_plan.md: set phase status to `in_progress`
2. Do the work
3. Update progress.md with what you did and files modified
4. Update task_plan.md: set phase status to `complete`, move Current Phase forward
5. If errors occurred, log them in both files

---

# PLATFORM ARCHITECTURE

```
Developer App (HTML/TS/Vite) → WebView in Even App → EvenAppBridge → BLE → G2 Glasses
                                                                      ↑
                                                               R1 Ring events
```

- App logic runs on the PHONE inside a WebView. The glasses only render containers.
- Your HTML/CSS is the settings/config UI on the phone. The glasses display is entirely SDK-driven.
- Two distinct UI surfaces: **App-side** (WebView, full web stack) and **OS-side** (glasses, container model).

---

# PROJECT SCAFFOLD

Always create this exact structure:

```
<app-name>/
├── index.html
├── app.json
├── package.json
├── tsconfig.json
├── vite.config.ts
└── src/
    ├── main.ts          # Bridge init, event routing, page management
    ├── pages.ts          # Page definitions (container layouts)
    ├── state.ts          # App state management
    ├── renderer.ts       # Render queue, text/image update helpers
    ├── events.ts         # Event normalization and input handling
    ├── constants.ts      # Display dimensions, timing, config
    └── settings.html     # (optional) Phone-side settings UI with even-toolkit
```

### index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>App Name</title>
</head>
<body>
  <div id="app"></div>
  <script type="module" src="/src/main.ts"></script>
</body>
</html>
```

### app.json — REQUIRED FIELDS

```json
{
  "package_id": "com.<author>.<appname>",
  "edition": "202601",
  "name": "<Display Name>",
  "version": "1.0.0",
  "min_app_version": "2.0.0",
  "min_sdk_version": "0.0.9",
  "tagline": "<One-liner, under 80 chars>",
  "description": "<Store description>",
  "author": "<developer-name>",
  "entrypoint": "index.html",
  "permissions": [
    {
      "name": "network",
      "desc": "Describe why this app needs network access (max 300 chars)",
      "whitelist": [
        "https://api.example.com"
      ]
    }
  ],
  "supported_languages": ["en"]
}
```

**Permission names**: `network`, `g2-microphone`, `phone-microphone`, `album`, `location`, `camera`
**Supported languages**: `en`, `de`, `fr`, `es`, `it`, `zh`, `ja`, `ko`

Rules:
- `package_id`: lowercase reverse-domain. Must be unique on Even Hub.
- `permissions`: array of objects. Each needs `name`, `desc`, and `whitelist` (for network). Include ONLY permissions the app actually uses.
- `desc`: explain WHY the permission is needed (max 300 chars). QA reads this.
- `whitelist`: list exact domains. QA will grep your source for undeclared fetch targets.
- If the app makes NO network calls, omit the network permission entirely.

### package.json

```json
{
  "name": "<app-name>",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "qr": "evenhub qr --url http://YOUR_LAN_IP:5173",
    "pack": "npm run build && evenhub pack app.json dist -o out.ehpk"
  },
  "dependencies": {
    "@evenrealities/even_hub_sdk": "^0.0.9"
  },
  "devDependencies": {
    "@evenrealities/evenhub-cli": "^0.1.11",
    "typescript": "^5.7.3",
    "vite": "^6.0.0"
  }
}
```

### vite.config.ts

```typescript
import { defineConfig } from 'vite'

export default defineConfig({
  server: {
    host: true,
    port: 5173,
    // Add proxy entries here if your app calls external APIs that lack CORS headers.
    // See "Network Calls & CORS" section for details.
    // proxy: { '/api': { target: 'https://api.example.com', changeOrigin: true, rewrite: p => p.replace(/^\/api/, '') } },
  },
  build: { target: 'es2020', outDir: 'dist' },
})
```

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "dist",
    "declaration": true
  },
  "include": ["src"]
}
```

---

# SDK API REFERENCE (@evenrealities/even_hub_sdk v0.0.9)

## Bridge Initialization — SINGLETON PATTERN

```typescript
import { waitForEvenAppBridge, type EvenAppBridge } from '@evenrealities/even_hub_sdk'

let bridge: EvenAppBridge | null = null

async function initBridge(): Promise<EvenAppBridge> {
  if (bridge) return bridge
  bridge = await Promise.race([
    waitForEvenAppBridge(),
    new Promise<never>((_, reject) =>
      setTimeout(() => reject(new Error('Bridge timeout')), 6000)
    ),
  ])
  return bridge
}
```

CRITICAL: `waitForEvenAppBridge()` must NOT be called concurrently (React StrictMode will double-invoke). Always guard with a module-level singleton.

## Container Types

### TextContainerProperty

```typescript
new TextContainerProperty({
  containerID: number,        // unique int (1-based)
  containerName: string,      // max 16 chars
  content: string,            // display text (max ~2000 chars with upgrade)
  xPosition: number,          // 0–576
  yPosition: number,          // 0–288
  width: number,              // px
  height: number,             // px
  isEventCapture: 0 | 1,      // EXACTLY ONE per page must be 1
  paddingLength?: number,      // 0–32, uniform all sides
  borderWidth?: number,        // 0–5
  borderColor?: number,        // 0–15 (greyscale intensity)
  borderRadius?: number,       // 0–10
})
```

### ListContainerProperty

```typescript
new ListContainerProperty({
  containerID: number,
  containerName: string,
  xPosition: number,
  yPosition: number,
  width: number,
  height: number,
  isEventCapture: 0 | 1,
  itemContainer: new ListItemContainerProperty({
    itemCount: number,          // 1–20
    itemWidth: number,          // 0 = auto
    isItemSelectBorderEn: 0 | 1,
    itemName: string[],         // one per item, max 64 chars each
  }),
})
```

### ImageContainerProperty

```typescript
new ImageContainerProperty({
  containerID: number,
  containerName: string,
  xPosition: number,
  yPosition: number,
  width: number,               // max 200
  height: number,              // max 100
})
// Image data sent SEPARATELY after container creation:
await bridge.updateImageRawData(new ImageRawDataUpdate({
  containerID: number,
  containerName: string,
  imageData: number[] | Uint8Array,  // raw image bytes
}))
```

## Page Lifecycle Methods

```typescript
// ONCE at startup — creates initial page
bridge.createStartUpPageContainer(new CreateStartUpPageContainer({
  containerTotalNum: number,     // total count of ALL containers
  textObject?: TextContainerProperty[],
  listObject?: ListContainerProperty[],
  imageObject?: ImageContainerProperty[],
}))

// ALL subsequent page structure changes
bridge.rebuildPageContainer(new RebuildPageContainer({
  containerTotalNum: number,
  textObject?: TextContainerProperty[],
  listObject?: ListContainerProperty[],
  imageObject?: ImageContainerProperty[],
}))

// Fast text-only update (NO flicker, NO layout change)
bridge.textContainerUpgrade(new TextContainerUpgrade({
  containerID: number,
  containerName: string,
  contentOffset: 0,
  contentLength: 2000,
  content: string,
}))

// Exit the app / close canvas on glasses
bridge.shutDownPageContainer(0)  // 0 = immediate exit, no confirmation
bridge.shutDownPageContainer(1)  // 1 = system-level popup confirmation (RECOMMENDED for double-tap)
```

## Event System

```typescript
bridge.onEvenHubEvent((event: EvenHubEvent) => {
  // Normalize event type — SDK quirk: CLICK_EVENT=0 maps to undefined
  const raw =
    event.listEvent?.eventType ??
    event.textEvent?.eventType ??
    event.sysEvent?.eventType

  const type = raw ?? 0  // undefined = CLICK (0)

  switch (type) {
    case 0: /* CLICK — single tap */; break
    case 1: /* SCROLL_TOP — swipe forward */; break
    case 2: /* SCROLL_BOTTOM — swipe backward */; break
    case 3: /* DOUBLE_CLICK — double tap */; break
    case 4: /* FOREGROUND_ENTER */; break
    case 5: /* FOREGROUND_EXIT */; break
    case 6: /* ABNORMAL_EXIT */; break
  }

  // List selection index (when list has isEventCapture)
  if (event.listEvent) {
    const index = event.listEvent.currentSelectItemIndex ?? 0
  }

  // Audio PCM (when audioControl(true))
  if (event.audioEvent?.audioPcm) {
    const pcm: Uint8Array = event.audioEvent.audioPcm
    // 16kHz, S16LE, mono, 100ms chunks = 3200 bytes
  }

  // IMU data (when imuControl(true))
  if (event.sysEvent?.eventType === 8) {
    const { x, y, z } = event.sysEvent.imuData
  }
})
```

## Device APIs

```typescript
// Device info
const device = await bridge.getDeviceInfo()
// device.model, device.sn, device.status.battery, device.status.wearStatus

// User info
const user = await bridge.getUserInfo()
// user.uid, user.name, user.avatar, user.country

// Launch source
bridge.onLaunchSource((source: string) => {
  // 'appMenu' — opened from Even App
  // 'glassesMenu' — opened from glasses Menu
})

// Device status changes
bridge.onDeviceStatusChanged((status) => {
  // status.connectType, status.battery, status.wearStatus, status.isCharging
})

// Persistent storage — SDK localStorage (survives app restarts, stored on phone)
// Use this for ALL user state: position, bookmarks, settings, preferences
await bridge.setLocalStorage('key', 'value')
const val = await bridge.getLocalStorage('key')
// Returns empty string if key doesn't exist

// Audio
await bridge.audioControl(true)   // start mic, PCM in audioEvent
await bridge.audioControl(false)  // stop mic

// IMU
await bridge.imuControl(true, ImuReportPace.P500)  // 500ms intervals
await bridge.imuControl(false)
```

## Return Codes

```typescript
// createStartUpPageContainer returns:
// 'success' | 'invalid' | 'oversize' | 'outOfMemory'

// updateImageRawData returns:
// 'success' | 'imageException' | 'imageSizeInvalid' | 'imageToGray4Failed' | 'sendFailed'
```

---

# DISPLAY CONSTRAINTS (OS-SIDE)

| Spec | Value |
|------|-------|
| Canvas | 576 × 288 px |
| Color | Monochrome green (#3CFA44) on black |
| Greyscale depth | 4-bit (16 shades, 0–15) |
| Coordinate origin | (0,0) top-left |
| Max containers/page | 4 per type (Text, List, Image) |
| Max total containers | 12 per page |
| Text alignment | Left-aligned, top-aligned ONLY |
| Font control | NONE — single fixed-width system font |
| Image max size | 200 × 100 px |
| Event capture | EXACTLY 1 container per page with isEventCapture: 1 |
| containerName max | 16 characters |

### Container Rules — NEVER VIOLATE

1. `createStartUpPageContainer` called EXACTLY ONCE. All subsequent = `rebuildPageContainer`.
2. ONE and ONLY ONE container per page has `isEventCapture: 1`.
3. Image containers: create placeholder first, then `updateImageRawData` SEQUENTIALLY. Never concurrent.
4. Image must match container dimensions exactly or it TILES (visual bug).
5. `textContainerUpgrade` for frequent content updates (no flicker). `rebuildPageContainer` for layout changes.
6. List items: plain text only, 1–20 items, max 64 chars each. No in-place updates — full rebuild required.
7. Text overflow clips (without event capture) or scrolls (with event capture).
8. No background colors, no per-item styling, no font control, no animations on glasses.

### Unicode That Works on LVGL Font

Good: `← ↑ → ↓ ─ ━ │ ┌ ┐ └ ┘ ├ ┤ █ ▁▂▃▄▅▆▇ ▌▍▎▏ ■ □ ● ○ ▲ ▼ ★ ☆`
Bad: emoji, weather symbols (☀☁), dingbats

### UI Patterns for Glasses

**Fake buttons**: Text with `> ` prefix as cursor. Move with scroll events.
**Progress bar**: `━━━━───────` using `━` (filled) and `─` (empty).
**Selection highlight**: `borderWidth: 0` (unselected) vs `borderWidth: 1–3` (selected).
**Full-screen event capture**: Hidden text container with `content: ' '`, `isEventCapture: 1`, covering 576×288.
**Text pagination**: Pre-paginate at ~400 chars/page, track pageIndex, rebuild on scroll boundary events.

---

# UX REQUIREMENTS (CRITICAL FOR QA)

## Glasses-First Principle

**QA Rule**: The app must work even when the phone app is in background or the phone screen is locked. Users must be able to open, use all main features, and exit the plugin entirely from the glasses/ring.

### Exit Mechanism — MANDATORY

Every app MUST implement a way to exit via glasses/ring interaction. **Use the system-level exit confirmation** — do NOT build your own.

**Pattern A (Recommended) — Double-tap calls system popup:**
```typescript
function handleEvent(event: EvenHubEvent) {
  const e = normalizeEvent(event)

  if (e.type === C.EVT_DOUBLE_CLICK) {
    persistState()      // save progress FIRST
    cleanup()           // release intervals/timers
    bridge.shutDownPageContainer(1)  // 1 = system confirmation popup
    return
  }
  // ... rest of event handling
}
```

**How `shutDownPageContainer(1)` works:**
- Calling with `1` triggers the Even App's built-in "Exit?" confirmation dialog on the glasses
- User can confirm or cancel via ring/temple input
- You don't need to draw your own dialog, manage timers, or track dialog state
- This is the canonical pattern — matches Even's native apps

**Exit mode values:**
- `shutDownPageContainer(0)` — immediate exit, no confirmation (use sparingly)
- `shutDownPageContainer(1)` — system popup confirmation (standard double-tap behavior)

**Pattern B — Menu with Exit option:**
```typescript
function showMenu() {
  bridge.rebuildPageContainer(new RebuildPageContainer({
    containerTotalNum: 1,
    listObject: [new ListContainerProperty({
      containerID: 1,
      containerName: 'menu',
      xPosition: 0, yPosition: 0,
      width: 576, height: 288,
      isEventCapture: 1,
      itemContainer: new ListItemContainerProperty({
        itemCount: 3,
        itemWidth: 0,
        isItemSelectBorderEn: 1,
        itemName: ['Resume', 'Settings', 'Exit'],
      }),
    })],
  }))
}
```

### Launch Source Handling

```typescript
bridge.onLaunchSource((source) => {
  if (source === 'glassesMenu') {
    // Launched from glasses — skip phone UI, go straight to glasses experience
    renderMainPage()
  } else {
    // Launched from app — can show phone settings first
    showSettingsUI()
  }
})
```

### Navigation

- Use `rebuildPageContainer` for page transitions (adds flicker but changes layout).
- Track current page state in a variable. Swipe forward/backward = navigate.
- Always provide a way back to the main screen.

### Idle Resilience

- App must not freeze after 2 minutes idle. No infinite loops, no leaked intervals.
- Clean up `setInterval`/`setTimeout` on exit events.
- Handle FOREGROUND_ENTER (event 4) to re-sync state after returning from background.
- Handle FOREGROUND_EXIT (event 5) to pause timers/streams.
- Handle ABNORMAL_EXIT (event 6) to clean up resources.

### Disconnect Recovery

- Monitor `bridge.onDeviceStatusChanged()` for disconnect/reconnect.
- On reconnect: re-render the current page state.
- On disconnect: pause operations, don't crash.

### Restart Cleanliness

- On app restart (new `createStartUpPageContainer`), reset ALL module-level state.
- Don't rely on stale singleton state from a previous session.

---

# PERFORMANCE RULES

## Render Queue — PREVENT CONCURRENT SENDS

```typescript
// renderer.ts — serialized render queue
let renderInFlight = false
let pendingRender: (() => Promise<void>) | null = null

async function enqueueRender(fn: () => Promise<void>) {
  if (renderInFlight) {
    pendingRender = fn  // only keep latest, drop intermediate
    return
  }
  renderInFlight = true
  try {
    await fn()
  } finally {
    renderInFlight = false
    if (pendingRender) {
      const next = pendingRender
      pendingRender = null
      await enqueueRender(next)
    }
  }
}

// Usage:
async function updateDisplay(content: string) {
  await enqueueRender(() =>
    bridge.textContainerUpgrade(new TextContainerUpgrade({
      containerID: 1,
      containerName: 'main',
      contentOffset: 0,
      contentLength: 2000,
      content,
    }))
  )
}
```

## Render Strategy Priority

1. **`textContainerUpgrade`** — fastest, text only, no flicker. Use for live data (clocks, scores, status).
2. **`rebuildPageContainer`** — full layout change. Use for page navigation. Causes brief flicker.
3. **`updateImageRawData`** — slowest. Must be sequential. Cache image bytes at startup.

## Timing

- Text-based animation: ~80ms/frame = ~12 FPS max practical.
- Clock updates: 1000ms intervals via `textContainerUpgrade`.
- Scroll event throttle: 100–300ms debounce to prevent double-fires.
- Bridge timeout: 6000ms for `waitForEvenAppBridge()`.

## Image Optimization

- Load and convert images at startup, not on demand.
- Cache byte arrays in memory.
- Never call `updateImageRawData` concurrently — serialize with the render queue.
- Match image dimensions exactly to container dimensions.

## Network Calls & CORS

The Even App WebView is a real browser engine (Chromium on Android, WKWebView on iOS). **Full CORS enforcement applies.** The `app.json` network whitelist is an Even-level permission layer — it does NOT bypass CORS. You need BOTH: domain whitelisted in app.json AND the remote server returning proper `Access-Control-Allow-Origin` headers.

### During Dev (QR sideload from localhost)

Origin = `http://YOUR_LAN_IP:5173`. External APIs that don't return `Access-Control-Allow-Origin: *` will be blocked. Use a Vite proxy:

```typescript
// vite.config.ts
import { defineConfig } from 'vite'

export default defineConfig({
  server: {
    host: true,
    port: 5173,
    proxy: {
      // Proxy /api/* requests to the real API server-side (no CORS)
      '/api': {
        target: 'https://api.example.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
      },
    },
  },
  build: { target: 'es2020', outDir: 'dist' },
})
```

Then in your app, call `/api/endpoint` instead of `https://api.example.com/endpoint`. Vite fetches it server-side — no CORS headers needed.

**IMPORTANT:** This proxy only works in dev. For production, you need one of:

### In Production (.ehpk in Even App WebView)

| API returns `Access-Control-Allow-Origin: *`? | What to do |
|-----------------------------------------------|------------|
| Yes (e.g. Gutenberg, Open Library, public APIs) | Just `fetch()` directly. It works. |
| No (most private APIs, some REST endpoints) | You need a CORS proxy — use Cloudflare Worker, your own backend, or a proxy service |
| You control the API | Add `Access-Control-Allow-Origin: *` to your server responses |

### CORS proxy pattern (Cloudflare Worker — free tier)

```typescript
// For APIs that don't support CORS, route through a lightweight proxy
const PROXY_URL = 'https://your-worker.your-subdomain.workers.dev'

async function fetchViaProxy(apiUrl: string) {
  const resp = await fetch(`${PROXY_URL}?url=${encodeURIComponent(apiUrl)}`)
  return resp.json()
}
```

### Rules
- **Cache API responses** where appropriate — don't re-fetch on every page turn.
- **Never block the render loop** waiting for network. Fire the fetch, render a loading state, update when data arrives.
- **Handle network errors** — show "No connection" on glasses, don't crash.
- **Declare all domains** in `app.json` `permissions.network` — QA will grep your source for undeclared fetch targets.

---

# PERSISTENCE (localStorage + IndexedDB)

Every app that tracks user state MUST persist it. Users expect to resume where they left off.

## SDK localStorage — Primary persistence layer

```typescript
// ALWAYS namespace keys to avoid collision with other plugins
const PREFIX = 'myapp.'

// Save structured data
async function saveState(bridge: EvenAppBridge, key: string, data: any): Promise<void> {
  await bridge.setLocalStorage(PREFIX + key, JSON.stringify(data))
}

// Load structured data
async function loadState<T>(bridge: EvenAppBridge, key: string, fallback: T): Promise<T> {
  try {
    const raw = await bridge.getLocalStorage(PREFIX + key)
    return raw ? JSON.parse(raw) : fallback
  } catch {
    return fallback
  }
}
```

### What to persist (and when)

| Data | When to save | Example key |
|------|-------------|-------------|
| User position / progress | On every meaningful state change (page turn, level complete, etc.) | `myapp.position` |
| User preferences / settings | On setting change | `myapp.settings` |
| Bookmarks / favorites | On add/remove | `myapp.bookmarks` |
| Last-used item | On item selection | `myapp.lastItem` |
| High scores / stats | On update | `myapp.stats` |

### Restore on startup

```typescript
async function init() {
  const bridge = await initBridge()

  // ALWAYS restore state before first render
  const savedPosition = await loadState(bridge, 'position', 0)
  const savedSettings = await loadState(bridge, 'settings', { density: 'normal' })

  state.position = savedPosition
  state.settings = savedSettings

  await render() // First render uses restored state
}
```

### Save on exit

```typescript
bridge.onEvenHubEvent((event) => {
  const type = event.sysEvent?.eventType ?? event.textEvent?.eventType
  // Save before exit — user expects to resume here
  if (type === 5 || type === 6 || type === 7) { // BACKGROUND, ABNORMAL, SYSTEM EXIT
    saveState(bridge, 'position', state.position)
  }
})
```

## IndexedDB — For large data (>5MB)

SDK localStorage is limited. For large content (books, cached API responses, media), use browser IndexedDB:

```typescript
function openDB(name: string, storeName: string): Promise<IDBDatabase> {
  return new Promise((resolve, reject) => {
    const req = indexedDB.open(name, 1)
    req.onupgradeneeded = () => req.result.createObjectStore(storeName, { keyPath: 'id' })
    req.onsuccess = () => resolve(req.result)
    req.onerror = () => reject(req.error)
  })
}

async function saveBlob(db: IDBDatabase, store: string, id: string, data: any): Promise<void> {
  return new Promise((resolve, reject) => {
    const tx = db.transaction(store, 'readwrite')
    tx.objectStore(store).put({ id, ...data })
    tx.oncomplete = () => resolve()
    tx.onerror = () => reject(tx.error)
  })
}

async function loadBlob(db: IDBDatabase, store: string, id: string): Promise<any> {
  return new Promise((resolve, reject) => {
    const tx = db.transaction(store, 'readonly')
    const req = tx.objectStore(store).get(id)
    req.onsuccess = () => resolve(req.result ?? null)
    req.onerror = () => reject(req.error)
  })
}
```

### Storage strategy summary

| Data size | Storage | API |
|-----------|---------|-----|
| Small (settings, position, metadata) | SDK localStorage | `bridge.setLocalStorage()` / `bridge.getLocalStorage()` |
| Large (files, cached content, images) | Browser IndexedDB | `indexedDB.open()` |
| Temporary (session state) | JS variables | Module-level `let` |

### Rules

1. **ALWAYS restore state on startup** — before the first `createStartUpPageContainer` call.
2. **ALWAYS save state on exit events** (BACKGROUND, ABNORMAL_EXIT, SYSTEM_EXIT).
3. **Namespace all keys** with app prefix to avoid collision.
4. **Wrap loads in try/catch** — storage can be empty or corrupted.
5. **Save incrementally**, not just on exit — exit events aren't guaranteed to fire.

---

# KEEP-ALIVE STRATEGIES

## Problem

The phone OS can suspend the WebView when the app is in the background or the screen is locked. This kills timers, WebSocket connections, and event listeners.

## IMU Keep-Alive Workaround

Enable IMU at low frequency to keep the WebView active:

```typescript
// Start IMU as keep-alive (lowest useful frequency)
async function enableKeepAlive() {
  try {
    await bridge.imuControl(true, ImuReportPace.P1000) // 1Hz
  } catch {
    // IMU not available — fall back to polling
  }
}

// Stop when not needed (save battery)
async function disableKeepAlive() {
  try {
    await bridge.imuControl(false)
  } catch {}
}

// In event handler, use IMU events as heartbeat:
bridge.onEvenHubEvent((event) => {
  if (event.sysEvent?.eventType === 8) {
    lastHeartbeat = Date.now()
    // IMU data available at event.sysEvent.imuData.{x,y,z}
  }
})
```

## Foreground/Background Lifecycle

```typescript
let isInForeground = true
let keepAliveInterval: number | null = null

bridge.onEvenHubEvent((event) => {
  const type = event.sysEvent?.eventType
  if (type === 4) { // FOREGROUND_ENTER
    isInForeground = true
    resumeApp()
  }
  if (type === 5) { // FOREGROUND_EXIT
    isInForeground = false
    pauseApp()
    enableKeepAlive()
  }
})

function resumeApp() {
  disableKeepAlive() // stop IMU keep-alive to save battery
  rerenderCurrentPage()
  restartTimers()
}

function pauseApp() {
  stopTimers()
  // Keep essential background processing only
}
```

## Cleanup on Exit

```typescript
const cleanupFns: Array<() => void> = []

function registerCleanup(fn: () => void) {
  cleanupFns.push(fn)
}

function cleanup() {
  cleanupFns.forEach(fn => fn())
  cleanupFns.length = 0
}

// Register cleanup for intervals, WebSockets, etc.
const timer = setInterval(tick, 1000)
registerCleanup(() => clearInterval(timer))

// On exit events:
bridge.onEvenHubEvent((event) => {
  const type = event.sysEvent?.eventType ?? event.textEvent?.eventType
  if (type === 6 || type === 7) { // ABNORMAL_EXIT or SYSTEM_EXIT
    cleanup()
  }
})
```

---

# APP-SIDE DESIGN SYSTEM (WebView Settings UI)

When the app has phone-side settings/configuration, use even-toolkit for the WebView UI.

## Install

```bash
npm install even-toolkit
```

## Import Pattern

```typescript
import 'even-toolkit/web/theme/tokens-light.css'
import 'even-toolkit/web/theme/tokens-dark.css'
import 'even-toolkit/web/theme/typography.css'
import 'even-toolkit/web/theme/utilities.css'
```

## Color Tokens (CSS Variables)

### Light Theme
| Variable | Hex | Usage |
|----------|-----|-------|
| `--color-text` | #232323 | Primary text |
| `--color-text-dim` | #7B7B7B | Secondary text |
| `--color-bg` | #EEEEEE | Page background |
| `--color-surface` | #FFFFFF | Card surface |
| `--color-surface-light` | #F6F6F6 | Elevated surface |
| `--color-border` | #E4E4E4 | Default border |
| `--color-accent` | #232323 | Primary CTA |
| `--color-accent-warning` | #FEF991 | Brand Yellow — warnings |
| `--color-positive` | #4BB956 | Success |
| `--color-negative` | #FF453A | Error |
| `--color-overlay` | rgba(0,0,0,0.50) | Modal backdrop |

### Dark Theme
| Variable | Hex |
|----------|-----|
| `--color-text` | #f0ebe3 |
| `--color-bg` | #0c0a07 |
| `--color-surface` | #161310 |
| `--color-accent` | #FFFFFF |
| `--color-accent-warning` | #FEF991 |

### Color Rules
- **#FEF991 (Brand Yellow)** = accent only. Sparingly. Never for body text.
- **#3CFA44 (OS Green)** = glasses display ONLY. Never in app UI.
- Dark mode uses warm-tinted neutrals, not neutral gray.
- Status: green = success, red = error, yellow = warning.
- Support BOTH light and dark themes.

## Typography

| Class | Size | Weight | Letter Spacing |
|-------|------|--------|----------------|
| `.text-vlarge-title` | 24px | 400 | -0.72px |
| `.text-large-title` | 20px | 400 | -0.60px |
| `.text-medium-title` | 17px | 400 | -0.17px |
| `.text-medium-body` | 17px | 300 | -0.17px |
| `.text-normal-title` | 15px | 400 | -0.15px |
| `.text-normal-body` | 15px | 300 | -0.15px |
| `.text-subtitle` | 13px | 400 | -0.13px |
| `.text-detail` | 11px | 400 | -0.11px |

- **Font**: FK Grotesk Neue. Fallback: -apple-system, sans-serif.
- **CJK**: Source Han Sans SC/TC/HC/JP/K.
- Weight 400 for titles, 300 for body. Never use Bold (700).
- Negative letter spacing is brand signature — do not override.
- Minimum font size: 11px.

## Spacing

| Variable | Value | Usage |
|----------|-------|-------|
| `--spacing-same` | 6px | Related items |
| `--spacing-margin` | 12px | Standard margin |
| `--spacing-card-margin` | 16px | Card padding |
| `--spacing-cross` | 12px | Between groups |
| `--spacing-section` | 24px | Between sections |
| `--radius-default` | 6px | All cards/inputs/buttons |

## Components Available (even-toolkit)

**Use these. Do NOT build custom equivalents.**

Primitives: Button, Card, Badge, Input, Textarea, Select, MultiSelect, Checkbox, RadioGroup, Slider, Toggle, SegmentedControl, Divider, Skeleton, Progress, StatusDot, Pill
Layout: AppShell, DrawerShell, NavBar, Page, SideDrawer, ScreenHeader, SectionHeader, SettingsGroup, ListItem, SearchBar, Tag
Feedback: Dialog, ConfirmDialog, Toast, EmptyState, Loading, BottomSheet
Data: Sparkline, LineChart, BarChart, PieChart, StatCard

## Icons (even-toolkit)

191 pixel-art icons at 32×32px grid.
```typescript
import { IcSettings, IcSearch, IcTrash } from 'even-toolkit/web/icons'
```
Use at 32px or exact multiples only.

## App-Side Do/Don't

| DO | DON'T |
|----|-------|
| Use even-toolkit components | Roll custom UI components |
| Use `--spacing-*` CSS variables | Hardcode pixel values |
| Use `--radius-default: 6px` | Mix different border radii |
| Use DrawerShell or NavBar+AppShell | Custom navigation |
| Support light + dark themes | Light theme only |
| Show EmptyState, Skeleton, Loading | Ignore empty/loading states |
| Use plain CSS or CSS variables | Use Tailwind CSS |
| FK Grotesk Neue for all text | Custom/system fonts |
| `--color-positive` for success | Ad-hoc green values |
| `--color-negative` for errors | Ad-hoc red values |
| `--color-accent-warning` for warnings | OS Green (#3CFA44) in app |

---

# COMPLETE APP TEMPLATE

Use this as your starting implementation and modify for the specific app:

### src/constants.ts

```typescript
export const CANVAS_W = 576
export const CANVAS_H = 288
export const BRIDGE_TIMEOUT_MS = 6000
export const EXIT_CONFIRM_MS = 3000
export const SCROLL_THROTTLE_MS = 150
export const RENDER_DEBOUNCE_MS = 50

// Event types
export const EVT_CLICK = 0
export const EVT_SCROLL_UP = 1
export const EVT_SCROLL_DOWN = 2
export const EVT_DOUBLE_CLICK = 3
export const EVT_FOREGROUND = 4
export const EVT_BACKGROUND = 5
export const EVT_ABNORMAL_EXIT = 6
export const EVT_SYSTEM_EXIT = 7
export const EVT_IMU = 8
```

### src/renderer.ts

```typescript
import type { EvenAppBridge } from '@evenrealities/even_hub_sdk'
import {
  TextContainerUpgrade,
  RebuildPageContainer,
  CreateStartUpPageContainer,
  ImageRawDataUpdate,
} from '@evenrealities/even_hub_sdk'

let renderInFlight = false
let pendingRender: (() => Promise<void>) | null = null

export async function enqueueRender(fn: () => Promise<void>) {
  if (renderInFlight) {
    pendingRender = fn
    return
  }
  renderInFlight = true
  try {
    await fn()
  } finally {
    renderInFlight = false
    if (pendingRender) {
      const next = pendingRender
      pendingRender = null
      await enqueueRender(next)
    }
  }
}

let startupRendered = false

export async function renderPage(
  bridge: EvenAppBridge,
  config: ConstructorParameters<typeof CreateStartUpPageContainer>[0],
) {
  await enqueueRender(async () => {
    if (!startupRendered) {
      // CRITICAL: Mark startup BEFORE the call, not after based on return value.
      // SDK return type varies (string vs enum) and checking result === 0
      // breaks silently — causing createStartUpPageContainer to be called
      // repeatedly, which floods the bridge and crashes the glasses.
      startupRendered = true
      try {
        await bridge.createStartUpPageContainer(
          new CreateStartUpPageContainer(config)
        )
      } catch (err) {
        startupRendered = false // allow retry on failure
        throw err
      }
    } else {
      await bridge.rebuildPageContainer(new RebuildPageContainer(config))
    }
  })
}

export async function updateText(
  bridge: EvenAppBridge,
  containerID: number,
  containerName: string,
  content: string,
) {
  await enqueueRender(() =>
    bridge.textContainerUpgrade(new TextContainerUpgrade({
      containerID,
      containerName,
      contentOffset: 0,
      contentLength: 2000,
      content,
    }))
  )
}

export async function updateImage(
  bridge: EvenAppBridge,
  containerID: number,
  containerName: string,
  imageData: number[],
) {
  await enqueueRender(() =>
    bridge.updateImageRawData(new ImageRawDataUpdate({
      containerID,
      containerName,
      imageData,
    }))
  )
}

export function resetRenderer() {
  startupRendered = false
  renderInFlight = false
  pendingRender = null
}
```

### src/events.ts

```typescript
import type { EvenHubEvent } from '@evenrealities/even_hub_sdk'
import { SCROLL_THROTTLE_MS } from './constants'

export type NormalizedEvent = {
  type: number      // 0–8
  listIndex?: number
  imuData?: { x: number; y: number; z: number }
  audioPcm?: Uint8Array
  source?: string
}

export function normalizeEvent(event: EvenHubEvent): NormalizedEvent {
  const result: NormalizedEvent = { type: -1 }

  // Extract event type from whichever sub-event is populated
  const raw =
    event.listEvent?.eventType ??
    event.textEvent?.eventType ??
    event.sysEvent?.eventType

  // SDK quirk: CLICK_EVENT=0 normalizes to undefined
  result.type = raw ?? 0

  // List selection index
  if (event.listEvent) {
    result.listIndex = event.listEvent.currentSelectItemIndex ?? 0
  }

  // IMU data
  if (event.sysEvent?.imuData) {
    result.imuData = event.sysEvent.imuData
  }

  // Audio PCM
  if (event.audioEvent?.audioPcm) {
    result.audioPcm = event.audioEvent.audioPcm
  }

  return result
}

// Scroll throttle
let lastScrollTime = 0
export function throttleScroll(handler: () => void): boolean {
  const now = Date.now()
  if (now - lastScrollTime < SCROLL_THROTTLE_MS) return false
  lastScrollTime = now
  handler()
  return true
}
```

### src/main.ts (template)

```typescript
import {
  waitForEvenAppBridge,
  TextContainerProperty,
  type EvenAppBridge,
  type EvenHubEvent,
} from '@evenrealities/even_hub_sdk'
import { renderPage, updateText, resetRenderer } from './renderer'
import { normalizeEvent, throttleScroll } from './events'
import * as C from './constants'

let bridge: EvenAppBridge | null = null

// === CLEANUP REGISTRY ===
const cleanupFns: Array<() => void> = []
function registerCleanup(fn: () => void) { cleanupFns.push(fn) }
function cleanup() { cleanupFns.forEach(fn => fn()); cleanupFns.length = 0 }

// === BRIDGE INIT ===
async function init() {
  if (bridge) return

  bridge = await Promise.race([
    waitForEvenAppBridge(),
    new Promise<never>((_, rej) =>
      setTimeout(() => rej(new Error('Bridge timeout')), C.BRIDGE_TIMEOUT_MS)
    ),
  ])

  // RESTORE PERSISTED STATE — always do this before first render
  // Example: restore last position, settings, etc.
  // const savedPage = await bridge.getLocalStorage('myapp.page')
  // if (savedPage) state.page = parseInt(savedPage)

  bridge.onLaunchSource((source) => {
    console.log('Launched from:', source)
  })

  bridge.onDeviceStatusChanged((status) => {
    console.log('Device status:', status.connectType)
  })

  bridge.onEvenHubEvent(handleEvent)

  await showMainPage()
}

// === EVENT HANDLING ===
function handleEvent(event: EvenHubEvent) {
  const e = normalizeEvent(event)

  // Double-tap = system exit confirmation (no custom dialog needed)
  if (e.type === C.EVT_DOUBLE_CLICK) {
    persistState()
    cleanup()
    bridge!.shutDownPageContainer(1) // 1 = system popup
    return
  }

  switch (e.type) {
    case C.EVT_CLICK:
      handleClick(e.listIndex)
      break
    case C.EVT_SCROLL_UP:
      throttleScroll(() => handleScrollUp())
      break
    case C.EVT_SCROLL_DOWN:
      throttleScroll(() => handleScrollDown())
      break
    case C.EVT_FOREGROUND:
      handleResume()
      break
    case C.EVT_BACKGROUND:
      handlePause()
      persistState() // SAVE STATE when going to background
      break
    case C.EVT_ABNORMAL_EXIT:
    case C.EVT_SYSTEM_EXIT:
      persistState() // SAVE STATE before exit
      cleanup()
      break
  }
}

// === PAGES ===
async function showMainPage() {
  await renderPage(bridge!, {
    containerTotalNum: 1,
    textObject: [
      new TextContainerProperty({
        containerID: 1,
        containerName: 'main',
        content: 'Hello G2!\n\nTap to interact\nDouble-tap to exit',
        xPosition: 8, yPosition: 8,
        width: 560, height: 272,
        isEventCapture: 1,
        paddingLength: 4,
      }),
    ],
  })
}

// === PERSIST STATE — save user progress to SDK localStorage ===
async function persistState() {
  if (!bridge) return
  // Example: save current position so user resumes here
  // await bridge.setLocalStorage('myapp.page', String(state.currentPage))
  // await bridge.setLocalStorage('myapp.settings', JSON.stringify(state.settings))
}

// === APP-SPECIFIC HANDLERS (customize these) ===
function handleClick(listIndex?: number) {
  // TODO: implement click behavior
}

function handleScrollUp() {
  // TODO: implement scroll up behavior
}

function handleScrollDown() {
  // TODO: implement scroll down behavior
}

function handleResume() {
  showMainPage()
}

function handlePause() {
  // Pause timers, enable keep-alive if needed
}

// === START ===
init().catch((err) => {
  console.error('Failed to initialize:', err)
  document.getElementById('app')!.textContent = `Error: ${err.message}`
})
```

---

# QA SELF-CHECK (run before submission)

Before presenting the app as complete, verify EVERY item:

## 6.1 app.json Review
- [ ] `package_id` is lowercase reverse-domain format
- [ ] `name`, `category`, `description`, `version` all present and accurate
- [ ] `permissions.network` lists ONLY domains actually used (grep source for fetch/XMLHttpRequest/WebSocket)
- [ ] No unnecessary permissions requested
- [ ] `entrypoint` points to correct HTML file
- [ ] `min_sdk_version` matches SDK version used

## 6.2 Privacy & Security
- [ ] No `eval()`, `new Function()`, or dynamic script injection in source
- [ ] No network calls to undeclared domains
- [ ] No unauthorized device API calls
- [ ] No data exfiltration patterns (base64-encoded sensor data sent externally)
- [ ] If backend used: domain is declared in permissions

## 6.3 Store Assets
- [ ] Screenshots/visuals are MONOCHROME/GREYSCALE — no color images
- [ ] App name doesn't impersonate existing apps or brands
- [ ] Description is accurate, no keyword stuffing
- [ ] No unauthorized trademarks or logos

## 6.4 UX & Stability (THE CRITICAL SECTION)
- [ ] **Glasses-first**: App works when phone is in background or screen locked
- [ ] **Startup**: Loads within reasonable time, no infinite spinner
- [ ] **Background launch**: Can start from glasses Menu even when app is in background
- [ ] **Core flow**: All main functions work via glasses/ring interaction alone
- [ ] **Display**: Text readable at 576×288, no clipping or overflow
- [ ] **Exit**: Can exit via double-tap confirmation dialog or menu Exit option
- [ ] **Navigation**: Can navigate between pages/states and return to main
- [ ] **Idle**: No freeze or infinite loop after 2 minutes idle
- [ ] **Disconnect recovery**: BT disconnect + reconnect = app recovers or degrades gracefully
- [ ] **Restart**: Close and reopen = no residual state or crashes
- [ ] **State reset**: Module-level variables reset properly on new session

## 6.5 Content & Safety
- [ ] No medical, financial, or emergency routing features (flag for legal if present)
- [ ] No offensive, explicit, or NSFW content
- [ ] If AI-generated content: outputs are filtered for harmful content

## Performance
- [ ] Render queue prevents concurrent bridge calls
- [ ] `textContainerUpgrade` used for frequent updates (not `rebuildPageContainer`)
- [ ] Image byte arrays cached at startup
- [ ] Network calls don't block render loop
- [ ] Scroll events throttled (100–300ms)
- [ ] All intervals/timeouts cleaned up on exit
- [ ] IMU keep-alive enabled during background execution if needed

## Container Integrity
- [ ] `containerTotalNum` matches actual container count
- [ ] Exactly ONE container has `isEventCapture: 1`
- [ ] All containerIDs are unique within a page
- [ ] containerName ≤ 16 characters
- [ ] Image dimensions match container dimensions
- [ ] Image updates are sequential (not concurrent)

---

# CLI COMMANDS REFERENCE

```bash
# Dev server
npm run dev

# Sideload via QR (find LAN IP: ipconfig getifaddr en0)
npx evenhub qr --url http://YOUR_IP:5173

# Simulator
npx @evenrealities/evenhub-simulator http://localhost:5173

# Build
npm run build

# Package
npx evenhub pack app.json dist -o out.ehpk

# Login (for publishing)
npx evenhub login
```

---

# IMPORTANT REMINDERS

1. **Two UIs**: WebView (phone) = settings/config UI with even-toolkit. Glasses = SDK containers only.
2. **Never use #3CFA44 (OS Green) in app UI**. It's for glasses display only.
3. **createStartUpPageContainer once**. Then rebuildPageContainer forever.
4. **One isEventCapture per page**. Always.
5. **Double-tap = `shutDownPageContainer(1)`**. Use the system popup, don't build custom dialogs. QA will reject without an exit path.
6. **Works from glasses Menu**. Even with phone locked. Always.
7. **Sequential image updates**. Never concurrent.
8. **SDK quirk**: CLICK_EVENT (0) may arrive as `undefined`. Handle both.
9. **Test on simulator** during dev: `npx @evenrealities/evenhub-simulator http://localhost:5173`
10. **Test on real device** before submitting. Simulator doesn't catch everything.
