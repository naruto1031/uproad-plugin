---
description: Turn a client brief, meeting notes, or a rough idea into a spec plus a working HTML prototype, push both to Uproad, and return a link the client can open without an account. Use when someone wants to show a client a proposal, mock-up, prototype, or 提案・たたき台.
---

# Take a brief to a client-ready link

The point of this skill is the **link at the end**. A prototype nobody outside the team can open is unfinished work. Do not stop at generating HTML.

## 1. Understand what is being proposed

Work from whatever the user has: meeting notes, a Slack paste, a one-line idea. Read files they point at rather than asking them to paste.

Ask only about things you cannot infer and that would change the output — typically: who the client is, what they will decide from this, and whether there is an existing look to match. **One round of questions, not an interview.** If the brief is thin, make defensible choices, build, and list them as open questions in the spec — a concrete prototype gets better feedback than a questionnaire.

## 2. Write the spec first

Write it before the HTML. Deciding what the screens must do is what makes the prototype coherent, and the client reads this to check you understood them.

Keep it to what a reader needs, in Japanese, as Markdown:

- **背景・目的** — what problem, and how this gets judged
- **対象ユーザー** — who uses it, in what situation
- **画面と導線** — each screen and how they connect
- **機能要件** — what it must do, in a numbered list an engineer can implement against
- **確認事項** — everything you assumed or guessed

**確認事項 is the section that earns the review.** Anything you decided on the client's behalf goes here, phrased as a question. It converts your guesses into their decisions.

If the work has an API, write it as a separate `docs/api.md` (or `openapi.yaml` — Uproad accepts `.md` `.html` `.txt` `.json` `.yaml`).

## 3. Build one self-contained HTML file

- **One file.** Inline the CSS and JS. No build step, no local assets — Uproad serves this file on its own, so anything it references from disk is gone.
- Avoid external CDNs where you reasonably can. The client may open this on a train.
- Make it look finished. Real copy, real-looking data, no `Lorem ipsum` and no `TODO`. A prototype that looks unfinished gets feedback about being unfinished.
- Responsive unless told otherwise — clients open links on their phone.
- Static is fine, but wire up the interactions the client is being asked to judge (tabs, modals, form states). Fake the data.

## 4. Push it and get the link

Use the `uproad` MCP tools in this order:

1. `list_projects` — if there is more than one, pick the one matching the client and say which you picked. One project, or no obvious match: leave `project` off and it lands in the default.
2. `push_design` — pass a stable `external_key` (for example `acme-lp:index.html`). **This is what makes re-running safe**: the same key adds a version to the same design instead of creating a duplicate, so the link you already sent the client keeps working.
3. `push_doc` — the spec, at `docs/spec.md`. Same design, so the client and the next engineer reach both from one place. Repeat for any other document.
4. `get_share_link` — this publishes the design and returns the URL. Safe to call repeatedly; it returns the same URL.

## 5. Hand it over

Report, in Japanese:

- **The share link**, on its own line, unmissable
- What you built, in two or three lines
- **確認事項** — repeat them here. This is what you want the client to respond to
- Tell them the client needs no account, and can click anywhere on the prototype to leave a pinned comment

Then offer `/uproad:review` for when the comments come back.

## When things fail

- **401** — the API token is wrong or expired. It is set in the plugin's configuration (`/plugin` → uproad → configure), not in a file.
- **`title is required`** — you passed neither `design_id` nor a matching `external_key` and no title. Give a title on first push.
- **`project not found`** — projects are never created implicitly. Run `list_projects` and use an existing name, or create it in the app.
- **quota exceeded** — the workspace hit its storage limit. Free is 200MB.
