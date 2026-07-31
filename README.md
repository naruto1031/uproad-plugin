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

From meeting notes, a Slack paste, or a one-line idea:

1. Writes a spec in Japanese — 背景・目的 / 対象ユーザー / 画面と導線 / 機能要件 / **確認事項**
2. Builds a self-contained HTML prototype
3. Pushes both to Uproad and returns a share link

The 確認事項 section is the part that earns a reply: every assumption made on the client's behalf is written back as a question they can answer.

The prototype and the spec hang off the same design, so one link reaches both — the client sees the screens, and whoever implements it later sees what it was supposed to do.

### `/uproad:review`

When the comments come back:

1. Reads the unresolved comments, including where on the page they were pinned
2. Applies the change requests, and updates the spec if the behaviour changed
3. Pushes a new version — **the share link does not change**
4. Marks resolved only what was actually done

Questions and out-of-scope requests are handed back to you instead of being silently closed. The MCP tools cannot reply to a comment, so answering happens in the app.

## Requirements

- Claude Code with plugin support
- An Uproad workspace (the free tier is enough to start)

Guest pinned comments are a Team-plan feature; on Free and Personal the client still opens the link, just without commenting on the page.

## Underneath

The plugin talks to Uproad's MCP server at `https://uproad.design/mcp` (Streamable HTTP, stateless, bearer auth). The same tools are available to any MCP client — see the [MCP docs](https://uproad.design/docs/mcp) if you want to use them directly, or [`npx uproad`](https://www.npmjs.com/package/uproad) for the command line.

## License

MIT
