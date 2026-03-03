# X (Twitter) Tips

X/Twitter has aggressive anti-bot detection. The following patterns achieve high success rates when combined with AgentGo cloud browsers.

## Required: iPhone Context

X heavily fingerprints browser sessions. Always use a mobile context:

```typescript
const context = await browser.newContext({
  userAgent:
    "Mozilla/5.0 (iPhone; CPU iPhone OS 17_4 like Mac OS X) " +
    "AppleWebKit/605.1.15 (KHTML, like Gecko) " +
    "Version/17.4 Mobile/15E148 Safari/604.1",
  viewport: { width: 390, height: 844 },
  deviceScaleFactor: 3,
  isMobile: true,
  hasTouch: true,
});
```

## Authentication via Cookies

X requires two cookies for authenticated sessions. Obtain them from a logged-in browser session (DevTools > Application > Cookies):

| Cookie | Purpose |
|--------|---------|
| `auth_token` | Session authentication |
| `ct0` | CSRF token |

```typescript
await context.addCookies([
  { name: "auth_token", value: "<your_auth_token>", domain: ".x.com", path: "/" },
  { name: "ct0", value: "<your_ct0>", domain: ".x.com", path: "/" },
]);
```

## Navigate to the Tweet Page First

**Never** use intent links (e.g. `https://x.com/intent/tweet?...`). Always navigate to the actual tweet URL and interact from there. X flags sessions that skip natural navigation.

```typescript
await page.goto("https://x.com/user/status/123456789");
await page.waitForSelector('[data-testid="tweet"]');
```

## Simulate Reading Before Interacting

Scroll the page to mimic a real user reading the tweet and replies:

```typescript
await page.evaluate(async () => {
  for (let i = 0; i < 2; i++) {
    window.scrollBy(0, 400);
    await new Promise(r => setTimeout(r, 1500));
  }
});
```

## Posting a Reply

### 1. Open the reply composer

```typescript
const replyBtn = await page.waitForSelector('[data-testid="reply"]');
await replyBtn.click();
await page.waitForSelector('[data-testid="tweetTextarea_0"]');
```

### 2. Safety check — confirm you are in reply mode

The submit button should say "Reply", not "Post". This prevents accidentally creating a standalone tweet:

```typescript
const buttonText = await page.innerText('[data-testid="tweetButton"]');
if (!buttonText.toLowerCase().includes("reply")) {
  throw new Error("Not in reply mode — aborting to prevent standalone post");
}
```

### 3. Type with human-like delays

Use `keyboard.type` with a per-character delay. `page.fill()` triggers X's "Silent Drop" detection (the post appears to send but is silently discarded).

```typescript
await page.type('[data-testid="tweetTextarea_0"]', replyContent, {
  delay: 80 + Math.floor(Math.random() * 70),
});
```

### 4. Submit and verify

```typescript
await page.click('[data-testid="tweetButton"]');
await page.waitForTimeout(3000);

// Reload and confirm the reply exists
await page.reload({ waitUntil: "networkidle" });
```

## Key X `data-testid` Selectors

| Selector | Element |
|----------|---------|
| `[data-testid="tweet"]` | A tweet in the timeline |
| `[data-testid="reply"]` | Reply button on a tweet |
| `[data-testid="tweetTextarea_0"]` | Compose text area |
| `[data-testid="tweetButton"]` | Submit button (Post / Reply) |
| `[data-testid="like"]` | Like button |
| `[data-testid="retweet"]` | Retweet button |
| `[data-testid="UserAvatar-Container-*"]` | User avatar |

## Complete Example: Reply to a Tweet

```typescript
import { chromium } from "playwright";

const apiKey = process.env.AGENTGO_API_KEY;
const opts = encodeURIComponent(JSON.stringify({ _apikey: apiKey }));
const serverUrl = `wss://app.browsers.live?launch-options=${opts}`;

const browser = await chromium.connect(serverUrl);
try {
  const context = await browser.newContext({
    userAgent:
      "Mozilla/5.0 (iPhone; CPU iPhone OS 17_4 like Mac OS X) " +
      "AppleWebKit/605.1.15 (KHTML, like Gecko) " +
      "Version/17.4 Mobile/15E148 Safari/604.1",
    viewport: { width: 390, height: 844 },
    deviceScaleFactor: 3,
    isMobile: true,
    hasTouch: true,
  });

  await context.addCookies([
    { name: "auth_token", value: "<your_auth_token>", domain: ".x.com", path: "/" },
    { name: "ct0", value: "<your_ct0>", domain: ".x.com", path: "/" },
  ]);

  const page = await context.newPage();
  await page.goto("https://x.com/user/status/123456789");

  // Simulate reading
  await page.evaluate(async () => {
    for (let i = 0; i < 2; i++) {
      window.scrollBy(0, 400);
      await new Promise(r => setTimeout(r, 1500));
    }
  });

  // Open reply composer
  const replyBtn = await page.waitForSelector('[data-testid="reply"]');
  await replyBtn.click();
  await page.waitForSelector('[data-testid="tweetTextarea_0"]');

  // Safety check
  const btnText = await page.innerText('[data-testid="tweetButton"]');
  if (!btnText.toLowerCase().includes("reply")) {
    throw new Error("Not in reply mode");
  }

  // Type and submit
  await page.type('[data-testid="tweetTextarea_0"]', "Great thread!", {
    delay: 100,
  });
  await page.click('[data-testid="tweetButton"]');

  console.log("Reply posted successfully");
} finally {
  await browser.close();
}
```
