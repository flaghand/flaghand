# Flaghand: MCP server for AI agents — owners approve X, LinkedIn and TikTok posts

Agents draft, upload media, check and request approval. Only the owner's passkey approves the exact text, media,
account and time; Flaghand posts exactly that. **Agents never approve, schedule, post, start a waiting job or send
a DM for anyone.** Request approval through Flaghand, not in chat.

Public sign-up is **closed**. The
[plans endpoint](https://app.flaghand.com/api/v1/plans) reports the current `signup_open` state. Only submit a waitlist address whose
person asked to join. The instructions below are the same [setup guide](https://app.flaghand.com/docs/setup.md) served by the app,
including its current CLI recipe and sign-up path.

# Set up and connect

Everything an agent needs before its first draft: an account, access to the workspace, the MCP server or the HTTP API,
the accounts the posts go out as, and the owner's voice note. This page is complete on its own; the next one is
[draft and request approval](https://app.flaghand.com/docs/drafting.md).

Flaghand drafts, checks and posts X, LinkedIn and TikTok posts for a person. An agent can do every step except
approving: only the workspace owner approves, with their passkey, and Flaghand posts only the exact snapshot they
approved. **Never approve, schedule, post, start a waiting job or send a DM for anyone, and never ask for approval
outside the product's own approval request.** No token can do those things, so there is nothing to work around.

## If you can't reach Flaghand

A sandbox can stop a request before Flaghand sees it. First try the public, read-only
`GET https://app.flaghand.com/api/v1/plans` without a token. Claude Code cloud documents `403` with
`x-deny-reason: host_not_allowed`; Copilot's documented warning says “Firewall rules blocked me from connecting to one
or more addresses”. Other clients do not document a universal blocked-host HTTP status or error string. A failed
connection, `EAI_AGAIN`, `ENOTFOUND`, or the CLI's local `network` error can mean a sandbox policy **or** ordinary
DNS/connectivity trouble. A bare `403` is not proof of a sandbox block.

For a confirmed client-policy block, ask exactly: “Please allow HTTPS requests to https://app.flaghand.com in this agent's network
settings, including POST, then let me retry Flaghand.” The person or their administrator changes the relevant
setting below; do not bypass a policy or ask for broad unrestricted access. Use the hostname from `https://app.flaghand.com`,
without `https://` or a path, in domain fields.

- **Codex cloud:** Settings › Codex Cloud › Environments › the environment's … › Edit. Turn on **Allow Codex to
  access internet**, then add the hostname under **Additional allowed domains** (with **Package managers** or
  **Custom domains only**). Save, **Republish**, and retry in a new task. Current docs do not specify the initial
  toggle state. In **legacy** cloud tasks, agent-phase internet is off by default: set **Agent internet access** to
  **On**, add the domain to its allowlist, and do not restrict methods to GET/HEAD/OPTIONS; drafting and MCP need POST.
  [Current controls](https://learn.chatgpt.com/docs/environments/cloud-environments#connect-to-services) ·
  [Legacy controls](https://learn.chatgpt.com/docs/cloud/internet-access).
- **Claude Code on the web:** the environment selector at claude.ai/code › edit the environment › **Network access**
  › **Custom** › **Allowed domains**. Add the hostname and save. The default is **Trusted**, not unrestricted.
  Organization-shared environments need an Owner's edit. Enabled MCP connectors use Anthropic's server-side route,
  not the session's domain list; this setting is for shell/HTTP requests.
  [Controls](https://code.claude.com/docs/en/cloud-environments#network-access) ·
  [Documented deny marker](https://code.claude.com/docs/en/routines#environments-and-network-access).
- **ChatGPT agent mode:** OpenAI's restricted-site list can stop a source; there is no documented user allowlist
  override for that block. If your interface instead uses **Cloud browser**, grant this website's access request or
  change its site permission in **Settings › Cloud browser**; it asks before a new website by default. This does not
  override OpenAI, workspace or website-side restrictions. Do not promise that changing a permission unblocks them.
  [Agent restrictions](https://help.openai.com/en/articles/11752874-chatgpt-agent) ·
  [Cloud browser permissions](https://help.openai.com/en/articles/20001280-using-cloud-browser-in-chatgpt).
- **Cursor background/cloud agents:** internet is allowed by default. If restricted, open the **Cloud Agents
  dashboard › Security** network settings and add the hostname to **Default + allowlist** or **Allowlist only**.
  A locked Enterprise policy needs an administrator. [Controls](https://cursor.com/docs/cloud-agent/security-network).
- **GitHub Copilot coding/cloud agent:** a firewall is on by default. A repository admin opens **Settings › Copilot
  › Internet access › Copilot cloud agent › Custom allowlist › Add rule**, adds the hostname and saves changes.
  If the organization disables repository custom rules, its owner must add the organization rule instead.
  [Controls](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-the-firewall).

An app response is different: Flaghand API errors are JSON with `error.code` and `error.message`.
`signup_closed`, `scope_missing`, `owner_session_required` and `workspace_inactive` are app decisions, not client
network blocks. `/mcp`'s `401` with `WWW-Authenticate` is its normal OAuth challenge (JSON-RPC code `-32001`). Follow
[Errors](https://app.flaghand.com/docs/errors.md), not network settings. Non-JSON HTML or a generic proxy error alone does not identify who blocked
it; report the status and safe diagnostic, never tokens or cookies. Settings and first-party sources checked October
26, 2026; the local check exercised the CLI in a no-network container, not these five cloud products end to end.

## 1. An account and access to it

The person already has Flaghand: they give you access in one of two ways.

- **Sign in from the app.** Claude, ChatGPT and other MCP apps that do OAuth add `https://app.flaghand.com/mcp` (step 2)
  and sign in: the owner signs in with their passkey, picks the workspace and clicks Allow. Nothing to paste. The app
  gets a workspace token with the `read`, `drafts`, `media` and `request-approval` scopes, listed in Settings › Agents,
  where the owner can revoke it.
- **A token.** The owner creates one in the desk, Settings › Agents › Create token (`https://app.flaghand.com/agents`), with those
  four scopes already chosen. It is shown once. Keep it in `FLAGHAND_TOKEN` or the client's secret storage, never in a
  draft, note, chat message or shared log. Only the owner can add `connections` (start connecting accounts), `cancel`,
  `billing`, `webhooks` or `calendar`; a token can't create another token.

The person has no account yet: sign-up opens with paid early access. Until then they
can join the waitlist with `POST https://flaghand.com/api/waitlist` and `{"email": "person@example.com"}` (no email is sent, and
it signs them up for nothing else). `GET https://app.flaghand.com/api/v1/plans` shows the plans and the trial. Stop there: nothing
else works without an account.


An agent that speaks the open [auth.md](https://app.flaghand.com/auth.md) agent-registration protocol can sign its person up through it
instead: `service_auth` at `https://app.flaghand.com/agent/identity` is this same sign-up, and the claim grant collects the same token.

## 2. Connect your agent

The MCP server is `https://app.flaghand.com/mcp` (streamable HTTP). Every request needs authentication, `initialize` included; a
`401` with `WWW-Authenticate` sends an OAuth client to sign-in.

- **Claude Code, in the session doing the setup:** use the HTTP API with the claimed or owner-provided token now;
  follow [the same-session recipe](https://app.flaghand.com/docs/setup#claude-code-finish-in-this-session) below. Do not ask the person to run `/mcp`,
  install software, copy a token or restart the chat. Configure MCP for a later session without interrupting this one.
- **Claude Code, optional interactive OAuth:** a person already using Claude Code can add
  `claude mcp add --scope user --transport http flaghand https://app.flaghand.com/mcp` and use `/mcp` themselves to sign in with a
  passkey and Allow. This interactive command is not an agent-callable tool or a prerequisite for HTTP setup.
- **Codex CLI, already in this session:** if you just collected the sign-up token, use the HTTP API below **in this
  session**. Keep the claimed token in your private credential store and pass it directly to the HTTP client; call
  `GET https://app.flaghand.com/api/v1/me`, then `getBilling` and continue setup. Do not ask the person to export a token, restart
  Codex or run a terminal command. Exporting a variable in a child shell cannot change the running Codex process, and
  an external `codex mcp add` does not by itself guarantee new tools in that session. A successful `getMe` over HTTP
  is authenticated access; MCP is optional, not a prerequisite for their first draft.
- **Codex CLI, a later session:** make the saved token available as `FLAGHAND_TOKEN` to the process that starts Codex,
  then run `codex mcp add flaghand --url https://app.flaghand.com/mcp --bearer-token-env-var FLAGHAND_TOKEN`. Only the variable's name
  is saved. Start the new session with that environment and call `getMe` through MCP to verify it. Codex also supports
  OAuth and dynamic client registration; [the Codex guide](https://app.flaghand.com/docs/agents.md#codex-cli) distinguishes that sign-in from the
  token you already claimed. Neither path requires the person to copy a credential into chat.
- **Claude apps (Free, Pro, Max):** add `https://app.flaghand.com/mcp` at Customize › Connectors › Add custom connector and sign in;
  choose Sign in now and Use Claude's published identity if shown. Free allows one custom connector.
- **ChatGPT Business and Enterprise/Edu, on the web:** Plugins › + › Add custom MCP server, OAuth, then sign in;
  use its published client identity or dynamic registration if offered. Developer Mode is not required in the current
  Plugins flow. Pro documents read/fetch access only; arbitrary custom-MCP access on Plus is not established by the current help page.
- **Gemini Apps, Perplexity and other chat apps:** check the [dated plans and click-by-click setup](https://app.flaghand.com/docs/agents.md#mcp-from-claude-chatgpt-and-other-apps-sign-in)
  first; a directory app or an MCP-capable developer product does not establish custom-URL access in a consumer app.
- **Any other MCP client:** the same URL with `Authorization: Bearer tok_…`, or OAuth if it supports it.
- **HTTP only** (curl, scripts, n8n, Make, Zapier): the API is `https://app.flaghand.com/api/v1`, JSON, with
  `Authorization: Bearer tok_…`. Every operation named below links its own page with its fields, an example request
  and answer, its errors, and its CLI and MCP names.

### Claude Code: finish in this session

MCP is optional, not the sign-up transport. Use HTTP for `listPlans`, `startSignup` and `claimSignup` above even
when no MCP tools are connected. Capture the ready claim's `token` directly into secret storage, not tool output;
keep the `workspace_id` too. The person only opens the returned sign-up, payment and account-consent links. A claim
is collected once: if you already saved its token, reuse it rather than claiming again or starting another sign-up.
Read the token from that storage inside every command; never retype it from earlier output, where some clients show
secrets as `***` or `<redacted>`.

With the token available to your script as `FLAGHAND_TOKEN`, call the API directly. This curl example passes the
header through stdin so the token is not in process arguments; do not enable shell tracing or curl verbose output:

```sh
printf 'Authorization: Bearer %s\n' "$FLAGHAND_TOKEN" |
  curl --fail-with-body --silent --show-error --header @- https://app.flaghand.com/api/v1/me
```

Read `getBilling` (`GET /api/v1/workspaces/{workspace}/billing`) next. A ready claim proves enrollment, not payment.
If Checkout is still open, keep the original sign-up link with the person and follow
[waiting for Checkout](https://app.flaghand.com/docs/billing.md#waiting-for-checkout); do not create a second payment session.
Then use HTTP for `getSetup`, `startConnection` (hand its `url` to the person), `listConnections`, `getVoice`,
`listSendBacks` and the first draft. See [draft and request approval](https://app.flaghand.com/docs/drafting.md) for exact bodies.
Keep using HTTP in this conversation if newly configured MCP tools are not available; a saved config is not a
successful connection, and installing a server does not finish the user's task.

For **later Claude Code sessions**, the agent can register the server itself:

```sh
claude mcp add --scope user --transport http flaghand https://app.flaghand.com/mcp \
  --header 'Authorization: Bearer ${FLAGHAND_TOKEN}'
```

The single quotes are intentional: Claude Code stores the literal environment-variable reference, not the token.
Provide `FLAGHAND_TOKEN` from secret storage in the environment of the shell that starts that later session. Do not
ask the person to export it or paste it. If a server with that name already exists, inspect its config before changing
it; do not blindly overwrite another connection. In a later session, call MCP `getMe` to verify the workspace and
scopes before using it. Until then, HTTP already works. Claude Code's [MCP reference](https://code.claude.com/docs/en/mcp)
documents HTTP headers and environment-variable expansion. Checked with Claude Code 2.1.310 (2026-10-26).
It also supports [`claude mcp login flaghand`](https://code.claude.com/docs/en/mcp#authenticate-from-the-command-line)
for optional interactive OAuth without `/mcp`; `--no-browser` prints an authorization link but still asks for the
full callback URL in the terminal. That is not the link-only sign-up path above. No hot-loading of newly added MCP
servers is assumed here.

The plugin also contains the same skill instructions. Its bundled MCP uses OAuth; if it isn't authenticated in this
session, use this HTTP path rather than sending the person to `/mcp` or claiming the plugin is connected.

The MCP server lists thirteen tools for the drafting loop: `getMe`, `listConnections`, `uploadMedia`, `createPost`,
`createDraft`, `getDraft`, `updateDraft`, `checkDraft`, `requestApproval`, `listJobs`, `getOperation`,
`callReadOperation` and `callWriteOperation`. Everything else is one step away: `getOperation` with
`{"operation_id": "getSetup"}` returns its schema, then `callReadOperation` (reads) or `callWriteOperation` (writes)
runs it with `{"operation_id": "getSetup", "arguments": {}}`; each refuses the other kind and names the right one.
In MCP, `workspace` is optional and defaults to your token's workspace. The operation tables on these pages say which
tool runs each operation.

The CLI needs Node 20 or later, but no installed package or source checkout. It isn't on npm yet; run the hosted
release with `npx`:

```sh
FLAGHAND_CLI="$(node --input-type=module -e 'const r=await fetch("https://app.flaghand.com/cli/latest.json"); if(!r.ok) throw Error("CLI catalog HTTP "+r.status); console.log((await r.json()).url)')"
npx --yes --package "$FLAGHAND_CLI" flaghand --help
npx --yes --package "$FLAGHAND_CLI" flaghand sign-up list-plans --json
```

Use that same `npx --yes --package "$FLAGHAND_CLI" flaghand` prefix for any CLI command, including `signup` when
sign-up is open, `connections wait` and `drafts wait`. The catalog includes the version, source commit and SHA-512
`integrity` of its immutable tarball. Keep its URL to pin a release; fetching the catalog again picks the current one. `npx` uses its
cache, not a project install. Once the package is published, the preferred command will be `npx --yes flaghand`;
the hosted versioned URLs will keep working.

To keep these instructions in Claude Code or Codex, install the [agent skill](https://app.flaghand.com/skill.md). Other clients'
recipes are in the [agents guide](https://app.flaghand.com/docs/agents.md).

<!-- claude-code-plugin:start -->
In Claude Code, the plugin installs the skill and the OAuth MCP server together:
`/plugin marketplace add https://app.flaghand.com/agent-plugin/marketplace.json`, then `/plugin install flaghand@flaghand`.
<!-- claude-code-plugin:end -->

### Gemini CLI

Install the first-party client extension from the public docs repository:

```sh
gemini extensions install https://github.com/flaghand/flaghand
```

Restart Gemini, then run `/mcp auth flaghand` and complete the owner's passkey/Allow sign-in. The extension uses
`httpUrl: https://app.flaghand.com/mcp`. Call `getMe` to confirm the workspace and scopes; installing an extension alone is not
proof of authentication. The extension contains client configuration and context, not Flaghand's server code.

## 3. Check who you are and what is set up

`getMe` (`GET /api/v1/me`) answers who is calling: `token` with its `name` and `scopes`, and `workspaces`, each with
its `id` (`ws_…`), `name` and `time_zone`. Over HTTP, put that id in every path as `{workspace}`. `next_steps` lists
these task pages, as before. `next` names the single next onboarding call from the same saved setup issues:

- No usable active account: `startConnection`. Set `platform` to the platform the person's task needs; the read
  accepts an optional `platform` query (MCP argument) such as `linkedin`, so another platform's account doesn't hide
  that missing connection. Known arguments are in `next.arguments`; an empty workspace never defaults to X.
  Relay the operation's returned `for_person` unchanged alongside `url`, rather than paraphrasing `next.tell_person`:
  they sign in on the platform and allow the connection themselves. Then wait for the returned connection attempt as
  described in step 4, without asking them to report completion. Starting the link works before payment; the agent
  cannot authorize the account.
- An account is usable but the plan is inactive: `startCheckout`. Relay its `for_person` unchanged alongside
  `checkout_url`; they enter their card in Stripe, not in chat. An existing past-due plan returns `openBillingPortal` instead,
  with `portal_url` for updating payment; it must not start a second subscription.
- An account and plan are ready (trialing, active or comped): `createPost`. Read the owner's voice and send-back
  notes, then supply the whole draft plan; this advice never approves or posts anything.

Load the named operation with `getOperation` before calling it: `next.arguments` contains only what is already
known, not a complete draft plan or a guessed platform/plan choice. A matching open Checkout is reused, so following
`startCheckout` does not invalidate a link the person is already completing. Read setup again after the connection
attempt is `connected` or the person finishes paying. `getMe.next` is null without `read` scope or for zero or multiple
workspaces; use a read-capable token and `getSetup` for the intended workspace. If `next.operation_id` is null, the
platform app is unconfigured: give the person `next.tell_person`'s support remedy rather than retrying an impossible
connection.
When several accounts need reconnecting, `next.arguments` never chooses a pinned account for them; match the
intended handle before supplying `connection_id`.

`getSetup` (`GET /api/v1/workspaces/{workspace}/setup`, `read` scope; MCP through `callReadOperation`) returns the
same `next` (and accepts the same optional `platform`) and checks the saved
setup without calling any platform: each connection's `status` and `missing_scopes`, the plan (`billing.state`,
`can_post`), your token's `missing_drafting_scopes`, failing webhooks and whether X credits ran out. Each entry in
`issues` has a `message` and a `next_step`; tell the person the ones only they can fix. `ok: true` is not an approval
and doesn't replace the draft check.

## 4. The accounts the posts go out as

`listConnections` (`GET /api/v1/workspaces/{workspace}/connections`) lists each account with its `id` (`con_…`),
`platform` (`x`, `linkedin` or `tiktok`), `handle`, `display_name` and `status`. Use the `id` of the **active**
connection as each draft's `connection_id`. If a platform has several, match the handle the person named, or ask.

No active account for a platform: only the person can connect it, on the platform's own consent screen.

- With the `connections` scope (a new-account claim has it; an existing-owner grant does not), start it for them: `startConnection`
  (`POST /api/v1/workspaces/{workspace}/connections/{platform}/start`, body `{}`, or `{"connection_id": "con_…"}` to
  reconnect that account) answers `url`, `for_person`, `attempt_id` (`cat_…`) and `expires_at`. Relay `for_person`
  **unchanged** alongside `url`; it names the platform (and the pinned account on reconnect), explains consent and
  says that nothing posts without the owner's passkey approval. They sign in on the platform and allow Flaghand.
  Do not name or guess any button on the page.
- Without it, ask them to connect the account in the desk, from its accounts menu or Settings.

**After handing off the URL, wait for that attempt without asking the person to report completion.** Keep its
`attempt_id`; a pending attempt is not a reason to create another link. The consent callback tells the person to go
back to their agent, using the agent name saved at sign-up when available. You learn the outcome from the attempt,
not from a message from the person or repeated `listConnections` calls.

- **CLI:** run `flaghand connections wait cat_… [--workspace ws_…] [--json]` once. It waits quietly, honors
  `Retry-After`, has no client-side deadline and prints the terminal result once. `failed` or `expired` exits nonzero.
- **HTTP:** `getConnectionAttempt` (`GET /api/v1/workspaces/{workspace}/connection-attempts/{id}`, `read` scope), with
  the returned `cat_…` as `{id}`. Pending answers HTTP `202` with `status: "pending"` and `Retry-After: 60`; wait for
  that interval before the next read. Terminal answers are HTTP `200` with `status: "connected"`, `"failed"` or
  `"expired"`. `connected` includes a `connection` object; use its active account's `id` as the draft's `connection_id`.
  A failure includes `error.code` and `error.message`; explain it instead of claiming success. Only `connected` means
  the account connected, even though every terminal answer is HTTP `200`.
- **MCP:** discover the schema with `getOperation {"operation_id":"getConnectionAttempt"}`, then, after handing off
  `url`, call `callReadOperation {"operation_id":"getConnectionAttempt","arguments":{"id":"cat_…"}}`.
  `workspace` is optional and defaults to the token's workspace. Obey `Retry-After` between pending reads; do not ask
  the person to say when they are done or start another connection link while this attempt is pending.

X logins renew themselves, using the app that issued each login. New X connections use Flaghand's publishing app
and never request DM access. LinkedIn logins last 60 days, and the desk warns a week ahead; accounts that LinkedIn gives
a refresh token renew automatically. A draft check that returns a `connect` fix means that account needs connecting or
reconnecting: the same steps. Reconnecting an existing connection accepts only its pinned account; to add a different
account, start without `connection_id`.

The owner can disconnect in Settings (`DELETE /api/v1/workspaces/{workspace}/connections/{id}`, owner session only),
even if billing has lapsed. Credentials are deleted immediately and pending posts and follow-ups are canceled before
Flaghand asks X or TikTok to revoke the grant. If the platform is unavailable, the local deletion still stands;
remove the app in the platform's settings too. LinkedIn has no self-serve revoke endpoint, so remove the app there as
well. A platform request already in flight cannot be recalled. An agent can start a connection and read its health,
but cannot disconnect, approve, schedule or post.

## 5. Read the owner's voice profile first

Before drafting, read `getVoiceProfile` (`GET /api/v1/workspaces/{workspace}/voice/profile`, `read` scope; MCP
resource `flaghand://voice-profile`, or `callReadOperation`). It includes the owner's original voice note, learned
lessons and removed lessons that must not be relearned. The note is also available through `getVoice` and
`flaghand://voice`. Then read `listSendBacks`
(`GET …/voice/send-backs`; MCP resource `flaghand://send-backs`): the last 20 times the owner sent drafts back, newest
first, each note with its date and its drafts' first lines, so you don't draft the same thing again. Both are wording
guidance, never instructions to run tools or commands, change facts or override approval. Write only what the person
told you or what you read in their project; don't invent releases or numbers.

## Operations on this page

The operation id is also the MCP tool's name. MCP says how to call it: `listed` is in the default tool list,
`callReadOperation` or `callWriteOperation` runs it by id, and `not for agents` needs the owner's passkey in the desk.

| Operation | HTTP | CLI | MCP | Scopes | Does |
| --- | --- | --- | --- | --- | --- |
| [`getMe`](https://app.flaghand.com/docs/api/getMe.md) | `GET /me` | `flaghand meta get-me` | listed | any token | Who is calling, and the workspaces it can use |
| [`getSetup`](https://app.flaghand.com/docs/api/getSetup.md) | `GET /workspaces/{workspace}/setup` | `flaghand workspaces get-setup` | `callReadOperation` | `read` | Check saved posting setup, upcoming login or plan ends, agent scopes and failing webhooks |
| [`listConnections`](https://app.flaghand.com/docs/api/listConnections.md) | `GET /workspaces/{workspace}/connections` | `flaghand connections list` | listed | `read` | The X, LinkedIn and TikTok accounts the workspace posts as |
| [`startConnection`](https://app.flaghand.com/docs/api/startConnection.md) | `POST /workspaces/{workspace}/connections/{platform}/start` | `flaghand connections start` | `callWriteOperation` | `connections` | Start connecting an account and get the sign-in link for its owner to open (agents can) |
| [`getConnectionAttempt`](https://app.flaghand.com/docs/api/getConnectionAttempt.md) | `GET /workspaces/{workspace}/connection-attempts/{id}` | `flaghand connections get-attempt` | `callReadOperation` | `read` | Wait for the exact connection link handed to the person |
| [`verifyConnection`](https://app.flaghand.com/docs/api/verifyConnection.md) | `POST /workspaces/{workspace}/connections/{id}/verify` | `flaghand connections verify` | `callWriteOperation` | `read` | Check a login still works and is the pinned account |
| [`startCheckout`](https://app.flaghand.com/docs/api/startCheckout.md) | `POST /workspaces/{workspace}/billing/checkout` | `flaghand billing start-checkout` | `callWriteOperation` | `billing` | A Stripe Checkout link to start the trial, or restart a plan; the person enters the card there |
| [`openBillingPortal`](https://app.flaghand.com/docs/api/openBillingPortal.md) | `POST /workspaces/{workspace}/billing/portal` | `flaghand billing open-portal` | `callWriteOperation` | `billing` | A Stripe Customer Portal link: card, invoices, monthly or yearly, cancel |
| [`createPost`](https://app.flaghand.com/docs/api/createPost.md) | `POST /workspaces/{workspace}/posts` | `flaghand posts create` | listed | `drafts` and `request-approval` | Draft, check and request approval in one call: the owner's approval URL, or each draft's problems and fixes |
| [`getVoiceProfile`](https://app.flaghand.com/docs/api/getVoiceProfile.md) | `GET /workspaces/{workspace}/voice/profile` | `flaghand voice get-profile` | `callReadOperation` | `read` | Read the workspace's X voice profile before drafting |
| [`getVoice`](https://app.flaghand.com/docs/api/getVoice.md) | `GET /workspaces/{workspace}/voice` | `flaghand voice get` | `callReadOperation` | `read` | The workspace's voice note: read it before drafting |
| [`listSendBacks`](https://app.flaghand.com/docs/api/listSendBacks.md) | `GET /workspaces/{workspace}/voice/send-backs` | `flaghand voice list-send-backs` | `callReadOperation` | `read` | The owner's last 20 send-back notes: read them before drafting |
| [`listPlans`](https://app.flaghand.com/docs/api/listPlans.md) | `GET /plans` | `flaghand sign-up list-plans` | `callReadOperation` | none | The plans on sale, their prices and the trial (from Stripe) |
| [`startSignup`](https://app.flaghand.com/docs/api/startSignup.md) | `POST /signup` | `flaghand sign-up start-signup` | `callWriteOperation` | none | Start agent access: one link to sign up or sign in and allow the agent; never reveals whether the email has an account |
| [`claimSignup`](https://app.flaghand.com/docs/api/claimSignup.md) | `POST /signup/claim` | `flaghand sign-up claim-signup` | `callWriteOperation` | none | Collect the agent token once; 202 until the person creates their account or allows the agent |
| [`joinWaitlist`](https://app.flaghand.com/docs/api/joinWaitlist.md) | `POST /waitlist` | `flaghand sign-up join-waitlist` | `callWriteOperation` | none | Join the public site's waitlist (no email is sent) |
| [`createToken`](https://app.flaghand.com/docs/api/createToken.md) | `POST /workspaces/{workspace}/tokens` | `flaghand tokens create` | not for agents | owner's passkey | Create an API token for an agent; its value is shown once |

Next: [draft and request approval](https://app.flaghand.com/docs/drafting.md). Also complete on their own:
[media and threads](https://app.flaghand.com/docs/media-and-threads.md), [schedule and results](https://app.flaghand.com/docs/schedule-and-results.md),
[billing](https://app.flaghand.com/docs/billing.md) and [errors](https://app.flaghand.com/docs/errors.md).


## Docs and discovery

- [Flaghand](https://flaghand.com/)
- [Site index for agents](https://flaghand.com/llms.txt)
- [App and API index for agents](https://app.flaghand.com/llms.txt)
- [auth.md agent registration](https://app.flaghand.com/auth.md) — open protocol; the person confirms with a passkey.
- [Documentation](https://app.flaghand.com/docs)
- [Agent skill](https://app.flaghand.com/skill.md) — `SKILL.md` here is byte-for-byte the hosted skill.
- [Agents and client connection recipes](https://app.flaghand.com/docs/agents.md)
- [Billing and trial terms](https://app.flaghand.com/docs/billing.md)
- [Published terms](https://flaghand.com/terms)
- [Service status](https://flaghand.com/status)
- [Official MCP Registry entry](https://registry.modelcontextprotocol.io/v0/servers/com.flaghand%2Fflaghand/versions/latest)

Support: **support@flaghand.com**. This is a docs-only repository; Flaghand's application code remains private.

## Licence

The files in this repository (these docs, the agent skill and the client and plugin manifests) are licensed under [MIT-0](https://github.com/flaghand/flaghand/blob/main/LICENSE). That covers only these files. Flaghand itself (the hosted service, its server and app code, and the CLI) is not open source and is not licensed here; using it is governed by the [Terms](https://flaghand.com/terms). The licence grants no rights to the Flaghand name or logo, which are trademarks of Trellisbyte LLC: you may say something works with Flaghand, but not present it as Flaghand or as made or endorsed by us.

## Where this comes from

After each successful production train deployment, this repository is refreshed from the app's rendered
[public README](https://app.flaghand.com/docs/github-readme.md), [skill](https://app.flaghand.com/skill.md) and the live MCP Registry entry's
`server` object. No application code is copied and the registry is not republished. There is no commit when the
three files are unchanged. Make instruction changes in Flaghand's docs first, not separately here.
