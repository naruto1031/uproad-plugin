# Publishing

Two independent distribution surfaces. Neither blocks the other, and the marketplace works the moment this repository is public — no review needed for that path.

## 1. Claude Code community marketplace

Users can already install from this repository directly:

```
/plugin marketplace add naruto1031/uproad-plugin
/plugin install uproad@uproad
```

To also get listed in Anthropic's reviewed `claude-community` catalog:

1. Validate first — the review pipeline runs this same check, plus automated safety screening:
   ```bash
   claude plugin validate ./plugins/uproad --strict
   ```
2. Submit at **[platform.claude.com/plugins/submit](https://platform.claude.com/plugins/submit)**.
   The claude.ai form (`claude.ai/admin-settings/directory/submissions/plugins/new`) needs a Team or Enterprise organisation; the Console form is the one for individual authors.
3. Approved plugins are pinned to a commit SHA in [`anthropics/claude-plugins-community`](https://github.com/anthropics/claude-plugins-community), and CI bumps that pin as this repository moves. The public catalog syncs nightly, so listing lags approval.

`claude-plugins-official` is curated by Anthropic at their discretion. There is no application process and the submission form does not feed into it.

## 2. MCP registry

`server.json` describes the remote server. Uproad is a hosted Streamable HTTP endpoint, so **no npm package is involved** — the npm step in the registry quickstart only applies to servers distributed as packages.

The server name must match the authentication method:

| Method | Required name | Setup |
| --- | --- | --- |
| Domain (current `server.json`) | `design.uproad/*` | DNS TXT record |
| GitHub | `io.github.naruto1031/*` | none |

Domain-based is what `server.json` currently uses, because it ties the server to the product rather than to a GitHub account. Switching to GitHub auth means changing the `name` field and running `mcp-publisher login github` instead.

```bash
brew install mcp-publisher

# Domain auth. Two traps, both of which produce unhelpful errors:
#  - the TXT record goes on the APEX (uproad.design), not under a selector
#    such as _mcp-auth.uproad.design
#  - macOS ships LibreSSL, which cannot do Ed25519. Use OpenSSL 3 explicitly,
#    or take the ECDSA P-384 path in the registry docs.
/opt/homebrew/opt/openssl@3/bin/openssl genpkey -algorithm Ed25519 -out key.pem
PUBLIC_KEY="$(/opt/homebrew/opt/openssl@3/bin/openssl pkey -in key.pem -pubout -outform DER | tail -c 32 | base64)"
echo "uproad.design. IN TXT \"v=MCPv1; k=ed25519; p=${PUBLIC_KEY}\""
# add that record, wait for propagation, then:
mcp-publisher login dns --domain uproad.design --private-key ./key.pem
mcp-publisher publish
```

Keep `key.pem` out of this repository — it is the proof of domain ownership. If you rotate keys, delete the old TXT record; a stale one is tried first and fails verification.

Moderation is deliberately permissive: the registry removes illegal content, malware, spam, and servers that do not function. It is not a quality gate. The registry is still in preview and warns that data resets are possible.

## 3. Smithery

Needs a GitHub repository and a `smithery.yaml`, submitted through [smithery.ai/new](https://smithery.ai/new). Remote HTTP endpoints are supported. Lowest priority of the three, and the exact requirements were not confirmed against first-party documentation — check the site before spending time on it.
