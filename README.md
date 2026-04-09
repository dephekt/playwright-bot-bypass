# playwright-bot-bypass v2.0

> Claude Code skill to bypass bot detection using `rebrowser-playwright` with 8 stealth patches.

## Installation

```
npx skills add greekr4/playwright-bot-bypass
```

## Features

- Pass bot.sannysoft.com all tests
- Google search without CAPTCHA
- Twitter/X scraping without login
- Real GPU fingerprint (Apple M2, NVIDIA, etc.)
- 8 fingerprint patches (webdriver, plugins, languages, permissions, canvas, etc.)
- Human-like behavior simulation (mouse movement, typing delays)
- Cookie persistence & proxy support
- Works with Node.js and Python

## A/B Test: bluer.co.kr (Real-World Bot Detection)

Tested against [Blue Ribbon Survey](https://www.bluer.co.kr) — a site with active bot protection:

| Metric | Stealth (this skill) | Normal Playwright |
|--------|---------------------|-------------------|
| **HTTP Status** | **200 OK** | **403 Forbidden** |
| `navigator.webdriver` | `undefined` | `true` |
| `navigator.plugins` | 3 (patched) | 0 (detected) |
| `navigator.languages` | `[ko-KR, ko, en-US, en]` | `[en-US]` |
| `outerWidth - innerWidth` | 16 (real chrome) | 0 (headless) |
| `chrome.runtime` | Present | Missing |
| WebGL Renderer | Apple M2 (real GPU) | SwiftShader (software) |
| User-Agent | Clean Chrome | HeadlessChrome |

### bot.sannysoft.com A/B

| Standard Playwright (Detected) | rebrowser-playwright (Bypassed) |
|:---:|:---:|
| ![Detected](ab-test-detected.png) | ![Stealth](ab-test-stealth.png) |

## Stealth Patches (8 vectors)

| # | Patch | Bypasses |
|---|-------|----------|
| 1 | `navigator.webdriver` removal | All bot detectors |
| 2 | `chrome.runtime` object | Cloudflare, sannysoft |
| 3 | `navigator.plugins` (3 plugins) | Cloudflare Bot Management |
| 4 | `navigator.languages` (ko-KR, en) | Akamai (cross-checks HTTP header) |
| 5 | Permissions API normalization | PerimeterX |
| 6 | `hardwareConcurrency` / `deviceMemory` | Advanced fingerprinters |
| 7 | `outerWidth` / `outerHeight` offset | Headless detection |
| 8 | Canvas fingerprint noise | Cloudflare Turnstile |

Plus: `--disable-blink-features=AutomationControlled`, `--no-sandbox`, real Chrome via `channel: 'chrome'`

## Quick Start

### Node.js (Recommended)

```bash
npm init -y && npm install rebrowser-playwright
```

#### Using the template (recommended)

```javascript
import { createStealthBrowser, humanDelay, humanType, simulateMouseMovement } from './scripts/stealth-template.mjs';

const { browser, page } = await createStealthBrowser();

try {
  await page.goto('https://example.com');
  await simulateMouseMovement(page);  // Natural mouse movement
  await humanType(page, 'input', 'query');  // Human-like typing
  await humanDelay(300, 800);
} finally {
  await browser.close();
}
```

#### Template options

```javascript
createStealthBrowser({
  headless: false,              // Required for stealth (default)
  viewport: { width: 1280, height: 800 },
  locale: 'ko-KR',             // Browser locale
  storageState: './session.json',  // Cookie persistence
  proxy: { server: 'http://proxy:8080' }  // Proxy support
});
```

#### Manual setup

```javascript
import { chromium } from 'rebrowser-playwright';

const browser = await chromium.launch({
  headless: false,
  channel: 'chrome',
  args: ['--disable-blink-features=AutomationControlled', '--no-sandbox']
});

const context = await browser.newContext({
  locale: 'ko-KR',
  extraHTTPHeaders: { 'Accept-Language': 'ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7' }
});

await context.addInitScript(() => {
  delete Object.getPrototypeOf(navigator).webdriver;
  // ... see SKILL.md for full patch list
});

const page = await context.newPage();
try {
  await page.goto('https://google.com');
} finally {
  await browser.close();
}
```

### Python

```bash
pip install undetected-chromedriver
```

```python
import undetected_chromedriver as uc

driver = uc.Chrome()  # auto-detects Chrome version
driver.get('https://google.com')
```

> Python `playwright-stealth` only patches at JS level — WebGL still shows SwiftShader. Use `undetected-chromedriver` instead.

## Test Results

| Environment | bot.sannysoft.com | Google Search | bluer.co.kr |
|-------------|-------------------|---------------|-------------|
| Standard Playwright | Detected | CAPTCHA | **403** |
| **rebrowser-playwright (this)** | **Pass** | **Works** | **200** |
| playwright-stealth (Python) | Pass | CAPTCHA | - |
| **undetected-chromedriver** | **Pass** | **Works** | - |

## Scripts Included

```
skills/playwright-bot-bypass/
  scripts/
    stealth-template.mjs      # Reusable stealth factory (all examples import this)
    bot-detection-test.mjs     # Verify bypass at bot.sannysoft.com
  examples/
    stealth-google-search.mjs  # Google search without CAPTCHA
    ab-test.mjs                # Side-by-side detected vs stealth
    stealth-twitter-scrape.mjs # Twitter/X profile scraping
  package.json                 # Dependencies (type: module)
  marketplace.json
  SKILL.md                     # Full documentation for Claude Code agents
```

## Requirements

- Node.js 18+ (ESM / `.mjs`)
- Google Chrome installed (not just Chromium)
- Headed mode (display required — `headless: false`)

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `ERR_MODULE_NOT_FOUND` | Run `npm install rebrowser-playwright` in your script directory |
| Browser not opening | Verify Chrome: `/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --version` |
| WebGL shows SwiftShader | Confirm import is from `rebrowser-playwright`, not `playwright` |
| Still getting detected | Add `simulateMouseMovement()` and `humanDelay()` between actions |
| Process hangs | Ensure `browser.close()` is in a `finally` block |

## License

MIT
