---
name: qpbuilder-run
description: Launch and drive Paper Setter (this repo's index.html) in a real headless browser to verify a change actually works — use whenever asked to run, test, or screenshot this app, or to confirm a UI/behavior change before reporting it done.
---

# Running Paper Setter

This is a single static file — `index.html` — with no build step and no
server. "Running" it means opening that file directly in a browser.
There's no `npm run dev`, no port to wait on.

## Why not just eyeball the source

This app is a single 100KB+ HTML file wiring together forms, localStorage,
`html2canvas`/`jsPDF`, and `docx.js`. Bugs in event wiring, `hidden`
attribute toggling, or type-specific branches (`q.type === "..."`) don't
show up from reading the code — they show up when you click through the
actual form. Always drive it in a real browser before calling a change done.

## Driving it

`chromium-cli` is not installed in this environment (checked: `command not
found`). Use **puppeteer-core** against the system's installed Chrome
instead — no npm project needed, install it straight into the scratchpad
directory:

```bash
cd "<scratchpad-dir>"
npm install puppeteer-core --no-save
```

Find the Chrome (or Edge, as fallback) executable — on this Windows box it's:

```
C:\Program Files (x86)\Google\Chrome\Application\chrome.exe
C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe   (fallback)
```

Then a driver script (adjust the two paths per repo location):

```js
const puppeteer = require('puppeteer-core');
const path = require('path');

(async () => {
  const browser = await puppeteer.launch({
    executablePath: 'C:\\Program Files (x86)\\Google\\Chrome\\Application\\chrome.exe',
    headless: 'new',
    args: ['--no-sandbox'],
  });
  const page = await browser.newPage();
  await page.setViewport({ width: 1400, height: 1000 });

  const consoleErrors = [];
  page.on('console', msg => { if (msg.type() === 'error') consoleErrors.push(msg.text()); });
  page.on('pageerror', err => consoleErrors.push('pageerror: ' + err.message));

  await page.goto('file:///E:/My personal Projects/qpbuilder/index.html', { waitUntil: 'networkidle0' });

  // ...drive it: page.click(selector), page.type(selector, text),
  // page.$eval / $$eval to read DOM state, page.waitForFunction / waitForSelector
  // to wait on state changes instead of sleeping...

  await page.screenshot({ path: path.join(__dirname, 'out.png') });
  console.log('Console errors:', JSON.stringify(consoleErrors));
  await browser.close();
})().catch(e => { console.error('TEST FAILED:', e); process.exit(1); });
```

Run it with `node script.js`, then **look at the screenshot** — a passing
console-error check with a blank or broken-looking page is not a pass.

Write throwaway driver scripts and screenshots to the scratchpad
directory, not into the repo, and delete them when done (along with the
`node_modules`/`package.json` puppeteer-core install) so nothing gets
accidentally committed.

## What to actually check

State lives entirely in `localStorage` (key `paperSetter.state.v1`) and
falls back to `sampleState()` when empty/absent — a fresh profile always
starts on the bundled sample paper, which is useful for a quick smoke
test with no setup.

For a typical change, verify:
- The relevant form control appears/hides correctly (many fields toggle
  via the `hidden` attribute based on `formType`/`subFormType` — check
  `page.$eval(selector, el => el.hidden)`).
- Adding/editing a question updates `#questionList` (check `.q-badges .badge`
  text) and the live preview in `#printArea`.
- Both preview modes render correctly — click
  `.view-btn[data-view="paper"]` / `.view-btn[data-view="key"]` and
  re-inspect `#printArea`'s innerHTML.
- No entries in `consoleErrors`.

PDF/.docx export (`#printBtn` / `#downloadDocxBtn`) trigger real file
downloads via `html2canvas`/`jsPDF`/`docx.js` — these are slower and
harder to assert on headlessly; a manual describe-and-defer is fine unless
the change specifically touches export code, in which case set
`page.setDownloadBehavior` (CDP) or check for thrown errors in the click
handler's promise chain instead of the resulting file.
