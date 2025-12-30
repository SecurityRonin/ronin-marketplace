---
name: browser-automation
description: Use when automating browsers with Playwright/Puppeteer, testing Chrome extensions, using Chrome DevTools Protocol (CDP), handling dynamic content/SPAs, or debugging automation issues
---

# Browser Automation

## When to Use This Skill

- Setting up browser automation (Playwright, Puppeteer, Selenium)
- Testing Chrome extensions (Manifest V3)
- Cloud browser testing (LambdaTest, BrowserStack)
- Using Chrome DevTools Protocol (CDP)
- Handling dynamic/lazy-loaded content
- Debugging automation issues

## Tool Selection

### Playwright (Recommended)

```bash
npm install -D @playwright/test
```

**Pros:**
- Multi-browser support (Chrome, Firefox, Safari, Edge)
- Built-in test runner with great DX
- Auto-wait mechanisms reduce flakiness
- Excellent debugging tools (trace viewer, inspector)
- Strong TypeScript support

**Use when:**
- Cross-browser testing needed
- Writing end-to-end tests
- TypeScript project

---

### Puppeteer

```bash
npm install puppeteer
```

**Pros:**
- Simpler API, easier to learn
- Smaller footprint
- Direct Chrome/Chromium control
- Official Chrome team project

**Use when:**
- Only need Chrome
- Simple automation tasks
- Quick scripts/prototypes

---

### Selenium

```bash
npm install selenium-webdriver
```

**Use when:**
- Legacy projects already using it
- Multi-language team
- Need specific Selenium features

---

## Chrome Extension Testing

### Local Testing (Recommended)

**For Manifest V3 Extensions:**

```javascript
// playwright.config.ts
export default defineConfig({
  use: {
    headless: false,
    args: [
      `--disable-extensions-except=${extensionPath}`,
      `--load-extension=${extensionPath}`,
    ],
  },
})
```

**Find extension ID via CDP:**

```typescript
const client = await context.newCDPSession(page)
const { targetInfos } = await client.send('Target.getTargets')

const extensionTarget = targetInfos.find((target: any) =>
  target.type === 'service_worker' &&
  target.url.startsWith('chrome-extension://')
)

const extensionId = extensionTarget.url.match(/chrome-extension:\/\/([^\/]+)/)?.[1]
```

**Navigate to extension pages:**

```typescript
await page.goto(`chrome-extension://${extensionId}/popup.html`)
await page.goto(`chrome-extension://${extensionId}/options.html`)
await page.goto(`chrome-extension://${extensionId}/sidepanel.html`)
```

---

### Cloud Testing Limitations

**What works:**
- Extension uploads to LambdaTest/BrowserStack
- Extensions load in cloud browsers
- Service workers run
- Can test content scripts on regular sites

**What doesn't work:**
- Cannot navigate to `chrome-extension://` URLs
- All attempts blocked with `net::ERR_BLOCKED_BY_CLIENT`

**Why:** Cloud platforms block extension URLs for security in shared environments.

**Verdict:** Use local testing for extension UI testing. Cloud for content script testing only.

---

## Chrome DevTools Protocol (CDP)

### Get All Browser Targets

```typescript
const client = await context.newCDPSession(page)
const { targetInfos } = await client.send('Target.getTargets')

const extensions = targetInfos.filter(t => t.type === 'service_worker')
const pages = targetInfos.filter(t => t.type === 'page')
```

### Intercept Network Requests

```typescript
await client.send('Network.enable')
await client.send('Network.setRequestInterception', {
  patterns: [{ urlPattern: '*' }],
})

client.on('Network.requestIntercepted', async (event) => {
  await client.send('Network.continueInterceptedRequest', {
    interceptionId: event.interceptionId,
    headers: { ...event.request.headers, 'X-Custom': 'value' },
  })
})
```

### Get Console Messages

```typescript
await client.send('Runtime.enable')
await client.send('Log.enable')

client.on('Runtime.consoleAPICalled', (event) => {
  console.log('Console:', event.args.map(a => a.value))
})

client.on('Runtime.exceptionThrown', (event) => {
  console.error('Exception:', event.exceptionDetails)
})
```

---

## Handling Dynamic Content

### Wait Strategies

```typescript
// Wait for specific content
await page.waitForSelector('.product-price', { timeout: 10000 })

// Wait for network to be idle
await page.goto(url, { waitUntil: 'networkidle' })

// Wait for custom condition
await page.waitForFunction(() => {
  return document.querySelectorAll('.item').length > 10
})
```

### Time-Based vs Scroll-Based Lazy Loading

**Key insight:** Some sites load content based on **time elapsed**, not scroll position.

**Testing approach:**
```javascript
// Test 1: Wait with no scroll
await page.goto(url)
await page.waitForTimeout(3000)
const sectionsNoScroll = await page.$$('.section').length

// Test 2: Scroll immediately
await page.goto(url)
await page.evaluate(() => window.scrollTo(0, 5000))
await page.waitForTimeout(500)
const sectionsWithScroll = await page.$$('.section').length

// If same result: site uses time-based loading
// No scroll automation needed - just wait
```

**Benefits of detecting time-based loading:**
- Simpler automation code
- No visual disruption
- More reliable extraction

---

### Handling Lazy-Loaded Images

```javascript
// Force lazy images to load
await page.evaluate(() => {
  // Handle data-src → src pattern
  document.querySelectorAll('[data-src]').forEach(el => {
    if (!el.src) el.src = el.dataset.src
  })

  // Handle loading="lazy" attribute
  document.querySelectorAll('[loading="lazy"]').forEach(el => {
    el.loading = 'eager'
  })
})
```

---

### Vanity URLs vs Internal IDs

**Problem:** Some sites use vanity URLs that differ from internal identifiers.

```
URL: /user/john-smith
Internal ID: john-smith-a2b3c4d5
```

**Solution:** Match by displayed content, not URL:

```javascript
// Strategy 1: Try URL-based ID
const urlId = location.pathname.split('/').pop()
let profile = findById(urlId)

// Strategy 2: Fall back to displayed name
if (!profile) {
  const displayedName = document.querySelector('h1')?.textContent?.trim()
  profile = findByName(displayedName)
}
```

---

## Cloud Browser Integration

### LambdaTest Setup

```typescript
// playwright.lambdatest.config.ts
const capabilities = {
  'LT:Options': {
    'username': process.env.LT_USERNAME,
    'accessKey': process.env.LT_ACCESS_KEY,
    'platformName': 'Windows 10',
    'browserName': 'Chrome',
    'browserVersion': 'latest',
  }
}

export default defineConfig({
  projects: [{
    name: 'lambdatest',
    use: {
      connectOptions: {
        wsEndpoint: `wss://cdp.lambdatest.com/playwright?capabilities=${encodeURIComponent(JSON.stringify(capabilities))}`,
      },
    },
  }],
})
```

---

## Performance Optimization

### Block Unnecessary Resources

```typescript
await page.route('**/*', route => {
  const type = route.request().resourceType()
  if (['image', 'font', 'media'].includes(type)) {
    route.abort()
  } else {
    route.continue()
  }
})
```

### Reuse Browser Context

```typescript
// Good: Reuse browser, create new contexts
const browser = await chromium.launch()
for (const url of urls) {
  const context = await browser.newContext()
  const page = await context.newPage()
  // ...
  await context.close()
}
await browser.close()
```

### Parallel Execution

```typescript
import pLimit from 'p-limit'
const limit = pLimit(5) // Max 5 concurrent

await Promise.all(
  urls.map(url => limit(() => processUrl(url)))
)
```

---

## Debugging

### Visual Debugging

```typescript
// Screenshots
await page.screenshot({ path: 'debug.png' })

// Video recording
const context = await browser.newContext({
  recordVideo: { dir: 'videos/' }
})
```

### Trace Viewer

```typescript
await context.tracing.start({ screenshots: true, snapshots: true })
// ... run test
await context.tracing.stop({ path: 'trace.zip' })

// View: npx playwright show-trace trace.zip
```

### Slow Motion & Pause

```typescript
const browser = await chromium.launch({
  headless: false,
  slowMo: 1000,
})

await page.pause() // Opens Playwright Inspector
```

---

## Quick Reference

### Common Selectors

```typescript
// CSS
await page.locator('.class')
await page.locator('#id')
await page.locator('[data-testid="value"]')

// Text
await page.locator('text="Exact text"')

// Playwright-specific
await page.getByRole('button', { name: 'Submit' })
await page.getByText('Welcome')
await page.getByLabel('Email')
```

### Data Extraction

```typescript
// Single element
const text = await page.textContent('.element')
const attr = await page.getAttribute('.element', 'href')

// Multiple elements
const texts = await page.$$eval('.item', els => els.map(e => e.textContent))

// Complex extraction
const data = await page.evaluate(() => {
  return Array.from(document.querySelectorAll('.product')).map(el => ({
    title: el.querySelector('.title')?.textContent,
    price: el.querySelector('.price')?.textContent,
  }))
})
```

---

## Common Issues

### Element Not Found

```typescript
// Wait for element
await page.waitForSelector('.element', { state: 'visible' })

// Check if in iframe
const frame = page.frame({ url: /example\.com/ })
if (frame) {
  await frame.waitForSelector('.element')
}
```

### Browser Connection Lost

```typescript
try {
  await page.goto(url)
} catch (error) {
  if (error.message.includes('Browser closed')) {
    browser = await chromium.launch()
    // retry
  }
}
```

---

## Resources

- [Playwright Docs](https://playwright.dev/)
- [Puppeteer Docs](https://pptr.dev/)
- [Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/)
- [LambdaTest Playwright Guide](https://www.lambdatest.com/support/docs/playwright-testing/)
