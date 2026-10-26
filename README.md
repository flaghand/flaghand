# Flaghand: MCP server for AI agents — owners approve and schedule X (Twitter), LinkedIn and TikTok posts

An MCP server for AI agents to draft X (Twitter), LinkedIn and TikTok posts. The owner uses a passkey to approve the exact text, media, account and time; Flaghand posts exactly that, now or on the approved schedule (TikTok: when the owner approves it).

**Agents draft, upload media, check and request approval. Agents never approve, schedule, post, start a waiting job or send a DM for anyone.** Request approval through Flaghand, not in chat. Only the owner's passkey approval creates a posting job.

## Start with your person

**As of October 26, 2026, public sign-up is closed.** `POST https://app.flaghand.com/api/v1/signup` answers HTTP **403** with `signup_closed`. Its JSON response names the next call: `POST https://app.flaghand.com/api/v1/waitlist` with `{"email":"person@example.com"}` (no token needed), or the same JSON to `POST https://flaghand.com/api/waitlist`. **Only submit an address whose person asked to join.** See the [documented error response](https://app.flaghand.com/docs/errors.md). Check the public [plans endpoint](https://app.flaghand.com/api/v1/plans): `signup_open` is the current state. Your person can also use the [waitlist](https://flaghand.com/); an existing owner can connect their agent below.

When sign-up opens, follow [set up and connect](https://app.flaghand.com/docs/setup.md):

1. Read the plans and tell your person the price before starting.
2. Send `POST https://app.flaghand.com/api/v1/signup` with the JSON below. Keep the returned `claim_code` in secret storage; give your person only `signup_url`.
3. Your person opens that link, adds their passkey and enters their card in Stripe Checkout. You never handle their passkey or card.
4. Collect the token with `claimSignup`: `POST https://app.flaghand.com/api/v1/signup/claim` with `{"claim_code": "…"}`. It answers `202 pending` with `Retry-After: 60` until the passkey is added, then once `200 ready` with the token. Don't loop pending calls into the conversation: stop the turn and call once when your person is back (the CLI's `flaghand signup wait` waits quietly). Save the token in secret storage at once; it is shown once.
5. The claim is ready after the passkey, not after payment. Read `getBilling`: only `can_post: true` means the workspace can post. You can't enter the card or finish payment for your person.

```json
{
  "email": "person@example.com",
  "plan": "monthly",
  "agent_name": "Claude Code"
}
```

Your first subscription starts with a **7-day free trial at checkout; a card is required**. Standard plans are **US $19/month or $190/year**, plus any required tax. The published founding offer, while available, is **$149 for the first year, then $190/year**. Unless canceled before the trial ends, the card is charged when it ends; plans renew automatically until canceled. Hand the Checkout step to your person. Read [billing](https://app.flaghand.com/docs/billing.md), the [current plans](https://app.flaghand.com/api/v1/plans) and the [published terms](https://flaghand.com/terms) before presenting an offer.

## Connect an agent

Remote MCP server: **`https://app.flaghand.com/mcp`** — streamable HTTP, with OAuth or a workspace API token. Every MCP request needs authentication, including `initialize`; a `401` directs OAuth clients to sign-in.

For OAuth, the workspace owner signs in with their passkey, picks their workspace and clicks Allow. The connector receives only `read`, `drafts`, `media` and `request-approval` scopes. The owner can revoke it in Settings › Agents. No API token or OAuth scope lets an agent approve, schedule or post.

### Claude Code

```sh
claude mcp add --scope user --transport http flaghand https://app.flaghand.com/mcp
```

Then use `/mcp` to sign in. To install the skill and OAuth server together instead, use Flaghand's [hosted plugin](https://app.flaghand.com/agent-plugin/marketplace.json):

```text
/plugin marketplace add https://app.flaghand.com/agent-plugin/marketplace.json
/plugin install flaghand@flaghand
```

Choose the standalone MCP setup or the plugin, not both.

### Codex CLI

Use the documented token setup: the owner creates a workspace token in Settings › Agents with `read`, `drafts`, `media` and `request-approval`. Make it available as `FLAGHAND_TOKEN` through secret storage in the shell that starts Codex; never put its value in a command, config file or shared log.

```sh
codex mcp add flaghand --url https://app.flaghand.com/mcp --bearer-token-env-var FLAGHAND_TOKEN
codex mcp get flaghand --json
```

Or merge this equivalent entry into `~/.codex/config.toml` (or `$CODEX_HOME/config.toml`):

```toml
[mcp_servers.flaghand]
url = "https://app.flaghand.com/mcp"
bearer_token_env_var = "FLAGHAND_TOKEN"
```

Only the variable's name is saved. Start a new Codex session with that variable available. `mcp get` checks saved configuration, not a successful sign-in; call `getMe` to confirm the workspace and scopes.

### Gemini CLI

Install the extension from this public repository:

```sh
gemini extensions install https://github.com/flaghand/flaghand
```

Restart Gemini CLI, then use `/mcp auth flaghand` to complete the owner's OAuth sign-in. The extension registers the Streamable HTTP server and loads the short [GEMINI.md](GEMINI.md) context; no token, local server or build is required. Call `getMe` after signing in to verify the workspace and scopes. Install this extension **or** a manual `flaghand` MCP entry, not both.

**October 26, 2026 — extension 1.0.0:** added `gemini-extension.json` and Gemini-specific context drawn from the [canonical skill](https://app.flaghand.com/skill.md). The [official gallery](https://geminicli.com/extensions/browse/) automatically crawls public repositories tagged `gemini-cli-extension` daily and lists those that pass validation. Direct installation does not depend on gallery inclusion; publication here is not proof of a gallery listing or an agent's discovery.

### Cursor

Merge this into `~/.cursor/mcp.json`, or `.cursor/mcp.json` for one project:

```json
{
  "mcpServers": {
    "flaghand": { "url": "https://app.flaghand.com/mcp" }
  }
}
```

In Cursor's MCP settings, connect Flaghand and complete the owner's passkey/Allow sign-in. No authorization header or static client credentials are needed.

### claude.ai

Customize › Connectors › Add › Add custom connector. Name it **Flaghand**, enter **`https://app.flaghand.com/mcp`**, then Continue, Add and Connect. On Team or Enterprise, an Owner adds it under Organization settings › Connectors; each member then connects. Complete the owner's passkey/Allow sign-in.

### ChatGPT

For Business and Enterprise on the web: Plugins › + › Add custom MCP server. Name it **Flaghand**, enter **`https://app.flaghand.com/mcp`** under Connection, choose **OAuth**, review the client's risk notice, then Create as a plugin. Complete the owner's passkey/Allow sign-in; type `@` in a chat to use it. This is a custom connection, not a claim of a public directory listing.

These recipes follow the [agents guide](https://app.flaghand.com/docs/agents.md); client availability and menus can vary by plan. The [sign-in and tokens guide](https://app.flaghand.com/docs/sign-in-and-tokens.md) explains both authentication paths.

### HTTP API and CLI

HTTP base: `https://app.flaghand.com/api/v1`, JSON, with `Authorization: Bearer <token>`. Keep the token in secret storage. The owner creates it in Settings › Agents; the four drafting scopes above are enough for the approval-request loop.

The Flaghand CLI requires Node 20 or later. It **isn't on npm yet**, but the hosted release runs with nothing installed:

```sh
FLAGHAND_CLI="$(node --input-type=module -e 'const r=await fetch("https://app.flaghand.com/cli/latest.json"); if(!r.ok) throw Error("CLI catalog HTTP "+r.status); console.log((await r.json()).url)')"
npx --yes --package "$FLAGHAND_CLI" flaghand --help
npx --yes --package "$FLAGHAND_CLI" flaghand sign-up list-plans --json
```

Use that prefix for every CLI command. The [release catalog](https://app.flaghand.com/cli/latest.json) gives each release's immutable tarball `url` and SHA-512 `integrity`. This repository contains no CLI or server code. The [API reference](https://app.flaghand.com/api/v1/openapi.json) is public.

## Draft, check, request approval, then stop

Call `getMe`, select the intended active account from `listConnections`, and read the owner's voice and send-back notes. Upload media, then call `createPost` for the whole plan. It drafts, checks and requests approval only if every check passes; it does not approve or create a job.

Give the owner the returned `approval_url` and wait for their passkey approval in Flaghand. Afterward, read the job result and report its post links and actual status. X has live read-back; LinkedIn does not. A successful request is not proof a post went live.

For the full instructions, install the [agent skill](https://app.flaghand.com/skill.md). The `SKILL.md` in this repository is the same skill, ready to install from here.

## Docs and discovery

- [Flaghand](https://flaghand.com/)
- [Site index for agents — llms.txt](https://flaghand.com/llms.txt)
- [App and API index for agents — llms.txt](https://app.flaghand.com/llms.txt)
- [Documentation](https://app.flaghand.com/docs)
- [Service status](https://flaghand.com/status)
- [Official MCP Registry entry — com.flaghand/flaghand](https://registry.modelcontextprotocol.io/v0/servers/com.flaghand%2Fflaghand/versions/0.1.1)

Support: **support@flaghand.com**. This is a docs-only repository; Flaghand's application code remains private.

## Where this comes from

Documentation source snapshot: `9abafaf44a54cdbaff0d1379a95c8548f4105fc6` (October 26, 2026).

- **README.md** summarizes Flaghand's docs: `docs/overview.md`, `docs/setup.md`, `docs/agents.md`, `docs/sign-in-and-tokens.md` and `docs/billing.md` at that commit, with the closed-sign-up response and waitlist next call from `docs/errors.md`. The CLI recipe was run against hosted release 0.1.0 (`flaghand help` lists its commands). Prices and trial terms are the published terms and live plans on the date above.
- **SKILL.md** is `docs/skill.md` at that commit with its placeholders filled in (`{{product}}` = Flaghand, `{{cli}}` = flaghand, `{{origin}}` = https://app.flaghand.com, `{{env}}` = FLAGHAND). It is byte-for-byte the hosted skill at https://app.flaghand.com/skill.md as served on October 26, 2026.
- **server.json** copies the live MCP Registry entry's `server` object, version `0.1.1`, retrieved on October 26, 2026. It is a registry snapshot, not an independently authored configuration or a registry republish.

- **GEMINI.md** is a short Gemini CLI context drawn from that `SKILL.md`, with links back to the canonical skill and task guides rather than a second manual. **gemini-extension.json** is client configuration only: extension `1.0.0` registers the hosted MCP endpoint and that context file, without local server code, credentials or hooks.

Make documentation changes in Flaghand's docs first, then refresh this snapshot. Do not maintain a separate set of instructions here.
