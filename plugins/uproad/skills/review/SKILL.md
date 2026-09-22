---
description: Read the unresolved client comments on an Uproad design, apply the changes, show them to you locally before anything reaches the client, then push the new version and mark the ones you actually handled as resolved. Use when a client has left feedback or 赤入れ on a shared prototype and it needs folding back in.
---

# Close one review round

The client left comments on a link you sent. This skill turns that into a new version they can look at, and leaves an honest record of what was and was not done.

**There is one hard stop**: the fixes are shown to you locally before they replace what the client sees. The link is already in their inbox, so a wrong fix reaches them immediately.

## 1. Read what came back

1. `list_designs` — find the design if the user did not name one. Match on title or `external_key`. Every row carries its `workspace`; the list spans every workspace the connection can reach, so if the same title or key turns up in two workspaces, **ask which one** rather than picking. Everything after this is keyed by `design_id`, which already pins the workspace.
2. `list_comments` with `unresolved_only: true`.

If nothing is unresolved, say so and stop. Do not invent work.

Read all of them before touching anything. Clients contradict themselves across comments, and the second one usually wins — better to notice that now than to implement both.

Sort each comment into one of three piles, and say which pile you put things in:

- **Change requests** — do them
- **Questions** — the client is asking, not instructing. **You cannot answer these**: the MCP tools have no reply. Collect them and hand them to the user to answer in the app
- **Out of scope or contradictory** — anything that needs a decision you should not make alone (budget, scope, something that breaks a stated requirement). Surface it, do not quietly implement or quietly drop it

Comments carry `pin_x`/`pin_y` when the client pinned them to a spot on the prototype. Use that to locate what they meant, and say which element you think a pin refers to when it is ambiguous — a misread pin produces a confidently wrong fix.

## 2. Make the changes

1. `get_design_html` — always start from what is actually deployed, not from whatever is on disk. Someone else may have pushed since. Write it into `designs/<slug>/index.html`, which is where designs live (one directory each, matching `/uproad:new`). Derive `<slug>` from the design's `external_key` when it has one.
2. Apply the changes. Keep the edits tight; unrelated refactoring makes the next round harder to review.
3. If the feedback changes what the thing *does* rather than how it looks, the spec is now wrong too. Fetch it with `get_doc` (`spec.md`) and update `designs/<slug>/spec.md`. **A spec that silently drifts from the prototype is worse than no spec** — the next engineer trusts it. Do not push it yet; it goes out with the prototype in step 4.

## 3. Show it locally, then STOP

Serve the designs directory and **leave the server running in the background**:

```bash
python3 -m http.server 4173 --directory designs
```

If that port is already serving the same `designs/` root, reuse it rather than starting a second one. Otherwise pick a free port.

Give the user the direct URL (`http://localhost:4173/<slug>/`) and **stop, asking them to check the fixes.** Point out specifically what changed, so they know where to look.

**Do not push until they approve.** The next step replaces what the client is currently looking at behind a link that is already in their inbox — a wrong fix arrives faster than a right one. Apply what comes back and show them again, staying in this loop until they say it is good.

## 4. Push and mark resolved

1. `push_design` with the **same `design_id`**. This adds a version; the share link the client already has keeps working and now shows the new one.
2. `push_doc` for any document you changed (the spec is at `spec.md`).
3. `resolve_comment` — **only for comments you actually addressed.** Leave questions and out-of-scope items unresolved. Resolving something you did not do erases the client's request without them knowing.

## 5. Report

In Japanese:

- **対応した** — comment → what changed, one line each
- **確認したい** — the questions, quoted, so the user can paste them back to the client. Say plainly that these are still open in Uproad and need a reply in the app
- **対応していない** — with the reason. Say it out loud rather than leaving it to be discovered
- The share link is unchanged — say so. Clients worry that a new version means a new URL

If nothing needs the client's input, say the round is closed.

## When things fail

- **404 on the design** — the connection cannot reach the workspace it lives in (not a member, or the reach was narrowed on the consent screen when signing in). `list_workspaces` shows what it can see; to widen it, revoke the connection under Settings → Personal tokens and sign in again with `/mcp`.
- **A comment will not resolve** — it may already be resolved, or belong to another design. Re-run `list_comments` and check.
- **Port already in use** — another design's server may already be serving the same `designs/` root; reuse it. Otherwise pick a free port.
- **The prototype has moved on since the comment** — a comment pinned to an element that no longer exists is still real feedback. Say which version it was left against and ask rather than guessing.
