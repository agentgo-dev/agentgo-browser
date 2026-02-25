# Session Management with AgentGo

## Single session

```typescript
const browser = await connectAgentGo();
const page = await browser.newPage();
// ... work ...
await browser.close(); // always close to release cloud resources
```

## Multiple isolated contexts

```typescript
const browser = await connectAgentGo();
const ctxA = await browser.newContext();
const ctxB = await browser.newContext();

const pageA = await ctxA.newPage();
const pageB = await ctxB.newPage();
// pageA and pageB share no cookies or storage

await ctxA.close();
await ctxB.close();
await browser.close();
```

## Save and restore auth state

```typescript
// save after login
await context.storageState({ path: "auth.json" });

// restore in next session
const context = await browser.newContext({ storageState: "auth.json" });
```

## Always close in finally

Cloud browsers consume AgentGo credits while open:

```typescript
const browser = await connectAgentGo();
try {
  const page = await browser.newPage();
  await doWork(page);
} finally {
  await browser.close();
}
```
