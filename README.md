# Uproad plugin for Claude Code

Take a client brief to a link the client can actually open — without leaving Claude Code.

[Uproad](https://uproad.design) hosts HTML prototypes behind private links that need no account to view, and lets the person reviewing click anywhere on the page to leave a pinned comment. This plugin wires that into your session: two skills that cover the round trip, and the MCP connection set up for you.

## Install

```
/plugin marketplace add naruto1031/uproad-plugin
/plugin install uproad@uproad
```

You will be asked for an Uproad API token. Create one at [uproad.design/settings/tokens](https://uproad.design/settings/tokens) (workspace admins only) — it starts with `up_`. It is stored in secure storage, not in a settings file, and the plugin uses it to connect to Uproad's MCP server for you. There is no `claude mcp add` step.

## What it does

### `/uproad:new`

From meeting notes, a Slack paste, or a one-line idea — with **two stops for you** along the way:

1. **Interviews you** one question at a time, each with a recommended answer, until the requirements stop moving. It reads what it can find instead of asking about it.
2. Writes a spec in Japanese — 背景・目的 / 対象ユーザー / 画面と導線 / 機能要件 / **確認事項** — then **stops so you can correct it**.
3. Builds a self-contained prototype, serves it locally, and **stops so you can look at it**. Stays in that loop until you approve.
4. Only then pushes both to Uproad and returns a share link.

The stops are the point. A spec you have not corrected produces a prototype built on the wrong assumptions, and a prototype you have not seen should not reach a client.

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

The plugin talks to Uproad's MCP server at `https://uproad.design/mcp` (Streamable HTTP, stateless, bearer auth). The same tools are available to any MCP client — see the [MCP docs](https://uproad.design/docs/mcp) if you want to use them directly, or [`npx uproad`](https://www.npmjs.com/package/uproad) for the command line.

## License

MIT
