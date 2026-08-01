---
description: Interview the user about a client brief until the requirements are settled, write a spec, build a working HTML prototype, review it locally with them, then push to Uproad and return a link the client can open without an account. Use when someone wants to build a proposal, mock-up, prototype, or 提案・たたき台 for a client.
---

# Take a brief to a client-ready link

Four phases with **two hard stops**. Do not run them together. The stops exist because a spec the user has not corrected produces a prototype built on the wrong assumptions, and a prototype they have not seen should not reach a client.

```
1. 要件を詰める (interview)
2. Spec を書く ────────────── STOP: 人間が直す
3. プロトタイプを作る → ローカルで見せる ── STOP: 人間がレビューする
4. Uproad に push → 共有リンク
```

Work in Japanese with the user throughout.

---

## Phase 1 — Interview until the requirements are settled

**Interview relentlessly. Walk down each branch of the decision tree, resolving dependencies between decisions one by one.** Do not accept a thin brief and start building.

- **One question at a time.** Wait for the answer before asking the next.
- **Always give your recommended answer**, with the reason. A question without a recommendation makes the user do your thinking.
- **If a question can be answered by looking, look instead of asking.** Read the notes they pointed at, existing designs in the repo, `list_designs`, the current site. Never ask for something you can find.
- **Order questions by what they unlock.** Ask the decision that changes the most downstream choices first — who the client is and what they will decide from this, before colour and copy.

Stop when the answers no longer change what you would build. That is the shared understanding you are after — not an exhausted user. If a branch genuinely does not matter yet, say so and move on rather than asking about it.

Typical branches worth walking: 誰が見て何を判断するのか / 対象ユーザーと利用場面 / 必要な画面と遷移 / 各画面で判断に必要な情報 / 既存デザインやトンマナの制約 / 今回スコープに入れないもの.

## Phase 2 — Write the spec, then STOP

Create the design directory and write the spec:

```
designs/<slug>/spec.md
```

`<slug>` is short, lowercase, hyphenated, and names the thing — `acme-lp`, `billing-dashboard`. **One directory per design**, so several designs sit side by side without colliding.

Sections, in Japanese:

- **背景・目的** — the problem, and how this gets judged
- **対象ユーザー** — who uses it, in what situation
- **画面と導線** — each screen and how they connect
- **機能要件** — numbered, so an engineer can implement against them
- **確認事項** — everything still assumed or undecided

**確認事項 is what earns the review.** Anything you decided on the client's behalf goes here as a question.

Then **stop and ask the user to read and correct it.** Say plainly that you will not start building until they reply. Apply their corrections and confirm before moving on.

## Phase 3 — Build, serve locally, then STOP

Write the prototype to:

```
designs/<slug>/index.html
```

- **One self-contained file.** Inline CSS and JS. No build step and no local asset files — Uproad serves this file on its own later, so anything it references from disk is gone.
- Avoid external CDNs where you reasonably can. The client may open this on a train.
- Make it look finished. Real copy, plausible data, no `Lorem ipsum` and no `TODO`.
- Responsive unless told otherwise.
- Wire up the interactions the client is being asked to judge (tabs, modals, form states). Fake the data behind them.

Then start a static server **from the `designs/` root** so every design is browsable, and **leave it running in the background**:

```bash
python3 -m http.server 4173 --directory designs
```

Pick another port if 4173 is taken. Give the user the direct URL (`http://localhost:4173/<slug>/`) and **stop, asking them to look at it.**

Apply what they come back with and show them again. **Stay in this loop until they say it is good.** Do not push a prototype the user has not approved — the next step puts it in front of their client.

## Phase 4 — Push and hand over

Only after approval:

1. `list_projects` — if there is more than one, pick the one matching the client and say which you picked. One project, or no obvious match: leave `project` off.
2. `push_design` — the contents of `designs/<slug>/index.html`, with `external_key` set to that same path. **This is what makes re-running safe**: the same key adds a version to the same design instead of creating a duplicate, so a link already sent to the client keeps working.
3. `push_doc` — the spec, at `spec.md`.
4. `get_share_link` — publishes the design and returns the URL. Safe to call repeatedly.

Report, in Japanese:

- **The share link**, on its own line
- What you built, in two or three lines
- **確認事項** — repeat them; this is what you want the client to respond to
- The client needs no account, and can click anywhere on the prototype to leave a pinned comment

Then offer `/uproad:review` for when the comments come back.

## When things fail

- **401** — the API token is wrong or expired. It is set in the plugin's configuration (`/plugin` → uproad → configure), not in a file.
- **`title is required`** — you passed neither `design_id` nor a matching `external_key` and no title. Give a title on first push.
- **`project not found`** — projects are never created implicitly. Run `list_projects` and use an existing name, or create it in the app.
- **quota exceeded** — the workspace hit its storage limit. Free is 200MB.
- **Port already in use** — another design's server is probably still running. Reuse it if it is serving the same `designs/` root; otherwise pick a free port.
