# Uproad plugin for Claude Code

Take a client brief to a link the client can actually open — without leaving Claude Code.

[Uproad](https://uproad.design) hosts HTML prototypes behind private links that need no account to view, and lets the person reviewing click anywhere on the page to leave a pinned comment. This plugin wires that into your session: two skills that cover the round trip, and the MCP connection set up for you.

## Install

```
/plugin marketplace add naruto1031/uproad-plugin
/plugin install uproad@uproad
```

Nothing to paste. The first time a skill needs Uproad, Claude Code asks you to sign in: run `/mcp`, pick **uproad**, choose **Authenticate**, and a browser tab opens Uproad's consent screen. Log in, pick which workspaces the connection may reach (all of them by default — `/uproad:new` asks which workspace a new design goes to only when there is more than one), and you are back in the session. There is no `claude mcp add` step and no token to copy.

The connection shows up at [uproad.design/settings/personal-tokens](https://uproad.design/settings/personal-tokens) as **Claude Code（claude.ai）**. Revoke it there to disconnect; Claude Code will ask you to sign in again the next time.

**Updating from 0.5 or earlier:** the stored API token is no longer used. Run `/mcp` → uproad → Authenticate once after updating. (Tokens still work for CI through the [CLI](https://www.npmjs.com/package/uproad) and the GitHub Action.)

## What it does

### `/uproad:new`

From meeting notes, a Slack paste, or a one-line idea — with **two stops for you** along the way:

1. **Interviews you** one question at a time, each with a recommended answer, until the requirements stop moving. It reads what it can find instead of asking about it.
2. Writes a spec in Japanese — 背景・目的 / 対象ユーザー / 画面と導線 / 機能要件 / **確認事項** — then **stops so you can correct it**.
3. Builds a self-contained prototype, serves it locally, and **stops so you can look at it**. Stays in that loop until you approve.
4. Asks whether this goes out **公開** (a share link the client opens without an account) or stays **非公開**
   (visible only to signed-in members of your workspace), then pushes both to Uproad.

The stops are the point. A spec you have not corrected produces a prototype built on the wrong assumptions, and a prototype you have not seen should not reach a client.

The visibility question is asked before the push, not after: publishing through the MCP tools only goes one way, and taking a design back down is a toggle in the app.

The 確認事項 section is what earns a reply: every assumption made on the client's behalf is written back as a question the client can answer.

The prototype and the spec hang off the same design, so one link reaches both — the client sees the screens, and whoever implements it later sees what it was supposed to do.

### Where files land

One directory per design, so several sit side by side:

```
designs/
  acme-lp/
    index.html
    spec.md
  billing-dashboard/
    index.html
    spec.md
```

The local server runs from `designs/`, so every design is browsable at once.

### `/uproad:review`

When the comments come back:

1. Reads the unresolved comments, including where on the page they were pinned
2. Applies the change requests, and updates the spec if the behaviour changed
3. Serves the fixes locally and **stops so you can check them** — the client's link is already in their inbox, so a wrong fix would reach them at once
4. Pushes a new version — **the share link does not change**
5. Marks resolved only what was actually done

Questions and out-of-scope requests are handed back to you instead of being silently closed. The MCP tools cannot reply to a comment, so answering happens in the app.

## Requirements

- Claude Code with plugin support
- An Uproad workspace (the free tier is enough to start)

Guest pinned comments are a Team-plan feature; on Free and Personal the client still opens the link, just without commenting on the page.

## Underneath

The plugin talks to Uproad's MCP server at `https://uproad.design/mcp` (Streamable HTTP, stateless). Sign-in is OAuth 2.1: Uproad is its own authorization server, identifies Claude Code by its Client ID Metadata Document, and hands back a personal token as the access token — so nothing about permissions or workspaces differs from a token you would have pasted by hand. The same server can be added to claude.ai as a custom connector, and the same tools are available to any MCP client — see the [MCP docs](https://uproad.design/docs/mcp), or [`npx uproad`](https://www.npmjs.com/package/uproad) for the command line.

## License

MIT
