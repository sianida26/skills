---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets.

If the user passes a ticket reference, fetch it from the issue tracker and state its title before starting. If the reference is ambiguous, ask.

Call the Skill tool with "tdd" where possible, at pre-agreed seams.

Security tests are mandatory, even where the rest of the work is not test-first. Before writing code, list every trust boundary the work touches (routes, procedures, server actions, webhooks, queue consumers, untrusted input) and, for each, write the tests from the `tdd` skill's `security-tests.md` checklist: IDOR, leaky lists, tenant isolation, function-level authz, mass assignment, unauthenticated access, workflow bypass, races, rate limits and resource caps, client retry backoff, injection, data exposure. If the work touches no trust boundary, say so explicitly with the reason. Do not commit while a trust boundary lacks its security tests.

Run typechecking regularly, single test files regularly, and the full test suite once at the end.

Once done, call the Skill tool with "code-review" to review the work. In the final report, list each trust boundary and the security tests that cover it.

Commit your work to the current branch.
