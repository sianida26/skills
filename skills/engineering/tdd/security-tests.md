# Security Tests

A happy-path suite proves the right user can do the right thing. It says nothing about the wrong user doing it, and that is the half agents skip. Every seam that crosses a trust boundary (an HTTP route, an RPC or tRPC procedure, a server action, a queue consumer, a webhook, a CLI that takes untrusted input) gets security tests alongside its behaviour tests, in the same red → green loop.

## Rules

- **Use a second real actor.** Seed two users (and two tenants, if the system has tenants) and attack A's resource as B. Never mock the auth or permission layer: a mocked guard is the tautological anti-pattern, it passes by construction.
- **Assert the denial and the unchanged state.** Check the status code (follow the project's convention; prefer 404 over 403 where existence itself is sensitive), then read the resource back as its owner through the interface and confirm nothing changed.
- **Red first, here too.** Write the attack test before the guard. If it passes before the guard exists, either the guard already lives elsewhere (find it and name it) or the test is not attacking anything.
- **One attack per test.** Name it as the threat: "user cannot read another user's invoice", not "getInvoice returns 404".

## Checklist per trust-boundary seam

Walk this list for every seam under test. Write a test for each item that applies; skip an item only with a one-line reason.

| Threat | What the test does |
| --- | --- |
| **IDOR / BOLA** | B reads, updates, deletes, and exports A's resource by ID. Covers every verb, not just GET, and nested routes (`/orders/:id/items/:itemId` where the item belongs to another order). |
| **Leaky lists** | B's list, search, filter, count, and aggregate endpoints return none of A's rows. Includes filters that take an `ownerId`/`tenantId` parameter supplied by the caller. |
| **Tenant isolation** | Same as the two above, across tenants, including admin-of-tenant-X acting on tenant Y. |
| **Function-level authz** | A lower role calls the higher role's action (approve, delete, change role, export all, impersonate). |
| **Mass assignment** | The request body carries fields the caller must not set (`role`, `ownerId`, `tenantId`, `status`, `price`, `isVerified`, `approvedBy`, timestamps). They are ignored or rejected, and the stored value stays the server's. |
| **Unauthenticated** | No token, expired token, malformed token, token of a deleted or disabled user. All denied. |
| **Workflow bypass** | Skipping a state (draft straight to paid), acting on a finalised record, approving one's own request, replaying a one-time action. |
| **Race / double use** | Two concurrent requests redeem the same coupon, spend the same balance, or claim the same slot. Exactly one succeeds. |
| **Rate limit / brute force** | Login, OTP, password reset, signup, invite, and token endpoints: send the limit plus one and get 429 with `Retry-After`. Check the limit is keyed per account and per client (rotating usernames from one client, or one username from many clients, still trips it), and that a success does not reset the attacker's counter. |
| **Expensive actions** | Endpoints that send email/SMS/WA, call paid APIs or an LLM, export, or run heavy queries are rate limited per actor, so one user cannot burn quota or money for everyone. |
| **Resource caps** | Page size, batch size, upload size, request body size, and query depth/complexity have a server-side maximum; asking for `limit=100000` gets the cap, not the table. |
| **Client retry** | A client that calls the API (frontend, worker, SDK) backs off on 401/403/429/5xx and honours `Retry-After`, instead of retrying in a loop that turns one failure into a request storm. |
| **Injection** | Untrusted input reaches a query, a path, a URL fetch, a shell, or a template: SQL/NoSQL operators, `../` path traversal, SSRF to internal addresses (`169.254.169.254`, `localhost`), header injection. |
| **Data exposure** | Responses and error bodies contain no password hashes, tokens, secrets, internal IDs the caller should not see, or stack traces. |

## Example

```typescript
test("user cannot update another user's invoice", async () => {
  const alice = await seedUser();
  const bob = await seedUser();
  const invoice = await createInvoice(alice, { amount: 100 });

  const res = await api(bob).patch(`/invoices/${invoice.id}`, { amount: 1 });

  expect(res.status).toBe(404);
  const reread = await api(alice).get(`/invoices/${invoice.id}`);
  expect(reread.body.amount).toBe(100);
});

test("user cannot set their own role on signup", async () => {
  const res = await api().post("/signup", {
    email: "eve@example.com",
    password: "correct-horse-battery",
    role: "admin",
  });

  const me = await api(res.body.token).get("/me");
  expect(me.body.role).toBe("member");
});
```
