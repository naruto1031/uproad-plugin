# Publishing

Two independent distribution surfaces. Neither blocks the other, and the marketplace works the moment this repository is public — no review needed for that path.

## 1. Claude plugin directory (`claude-plugins-official`)

Users can already install from this repository directly:

```
/plugin marketplace add naruto1031/uproad-plugin
/plugin install uproad@uproad
```

To appear in the plugin directory that every Claude Code and Cowork user sees without adding a marketplace (surfaced in Claude Code as the `claude-plugins-official` marketplace — checked against [claude.com/docs/plugins/submit](https://claude.com/docs/plugins/submit) on 2026-09-22):

1. The repository must be public (closed source is not accepted). Validate first — the review pipeline runs the same check plus automated safety screening:
   ```bash
   claude plugin validate ./plugins/uproad --strict
   ```
2. Submit the GitHub link through one of the in-app forms:
   - **Console** — [platform.claude.com/plugins/submit](https://platform.claude.com/plugins/submit). Needs a Developer, Admin or Owner role on a Console organisation; this is the form for individual authors.
   - **claude.ai** — [claude.ai/admin-settings/directory/submissions/plugins/new](https://claude.ai/admin-settings/directory/submissions/plugins/new). Needs a Team or Enterprise organisation with directory management access (Owners by default).
3. Anthropic runs a basic automated review and lists the plugin as a community plugin. "Anthropic Verified" is a separate, deeper review with no guarantee. Review time varies with the queue.
4. After publication, pushes to this repository are mirrored automatically (CI re-runs the screening on each update) — no re-submission for updates.

Plugins that bundle a connector already in the Connectors Directory are more likely to be verified and show fewer warnings, so listing the MCP server there (below) helps this listing too.

## 1b. Connectors Directory (the MCP server itself)

Listing `https://uproad.design/mcp` in the [Connectors Directory](https://claude.com/docs/connectors/directory) puts Uproad in the connector list of claude.ai, Desktop, mobile, Code and Cowork, and makes it eligible for in-chat "suggested connectors". Requirements checked on 2026-09-22 ([submission guide](https://claude.com/docs/connectors/building/submission), [pre-submission checklist](https://claude.com/docs/connectors/building/review-criteria)):

- Submission happens in the claude.ai admin portal ([claude.ai/admin-settings/directory/submissions/new](https://claude.ai/admin-settings/directory/submissions/new)) and **requires a Team or Enterprise organisation** — there is no Console form for connectors.
- OAuth 2.0 for authentication (the server does CIMD + PKCE; DCR is not implemented and not required).
- Every tool needs a `title` and `readOnlyHint` / `destructiveHint` (done server-side, uproad PR #47).
- Public documentation ([uproad.design/docs/mcp](https://uproad.design/docs/mcp)), privacy policy URL, support contact, icon, and a fully populated test account the reviewer can sign in with.
- Seven policy acknowledgements; the server must call first-party APIs on a domain matching the product (it does).

Submissions are auto-scanned and listed as community connectors; Anthropic escalates useful ones to verified review on its own.

## 2. MCP registry

`server.json` describes the remote server. Uproad is a hosted Streamable HTTP endpoint, so **no npm package is involved** — the npm step in the registry quickstart only applies to servers distributed as packages.

The server name must match the authentication method:

| Method | Required name | Setup |
| --- | --- | --- |
| Domain (current `server.json`) | `design.uproad/*` | DNS TXT record |
| GitHub | `io.github.naruto1031/*` | none |

Domain-based is what `server.json` currently uses, because it ties the server to the product rather than to a GitHub account. Switching to GitHub auth means changing the `name` field and running `mcp-publisher login github` instead.

`key.pem` has already been generated in this directory (gitignored). The TXT
record it produced is:

```
host:  uproad.design        (apex — no subdomain, no selector)
value: v=MCPv1; k=ed25519; p=cg8tOXaw0C+r21oZSRDjT4OGX3fC8y4pHhZHWTl5nsw=
```

Add that at the DNS provider, wait for propagation, then:

```bash
brew install mcp-publisher   # already installed on the author's machine
mcp-publisher validate       # ✅ as of 2026-07-31
mcp-publisher login dns --domain uproad.design --private-key ./key.pem
mcp-publisher publish
```

To regenerate the key (which invalidates the record above):

```bash
# The record goes on the APEX. Under a selector such as _mcp-auth.uproad.design
# the registry never sees it and fails with a generic signature error.
# macOS may ship LibreSSL, which cannot do Ed25519 — use OpenSSL 3 explicitly
# (/opt/homebrew/opt/openssl@3/bin/openssl) or take the ECDSA P-384 path.
openssl genpkey -algorithm Ed25519 -out key.pem
openssl pkey -in key.pem -pubout -outform DER | tail -c 32 | base64
```

Note that `description` in `server.json` is capped at **100 characters** — the
registry rejects longer values with a 422 that only shows up at `validate` time.

Keep `key.pem` out of this repository — it is the proof of domain ownership. If you rotate keys, delete the old TXT record; a stale one is tried first and fails verification.

Moderation is deliberately permissive: the registry removes illegal content, malware, spam, and servers that do not function. It is not a quality gate. The registry is still in preview and warns that data resets are possible.

## 3. Smithery

Needs a GitHub repository and a `smithery.yaml`, submitted through [smithery.ai/new](https://smithery.ai/new). Remote HTTP endpoints are supported. Lowest priority of the three, and the exact requirements were not confirmed against first-party documentation — check the site before spending time on it.
