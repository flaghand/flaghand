# Flaghand for Cursor

Agents draft X and LinkedIn posts, upload media, check and request approval. Only the workspace owner's passkey
approves the exact post. Agents cannot approve, schedule or post.

This repository contains the MIT-0 plugin files and documentation, not Flaghand's server source. The plugin connects
to Flaghand, a paid hosted service. Installing it does not create an account, start a subscription or grant access to
a workspace. Read the [current plans and sign-up availability](https://app.flaghand.com/api/v1/plans) before setup;
`signup_open: false` means new accounts cannot be created yet. Never ask someone to pay in chat.

## Plugin contents

- `.cursor-plugin/plugin.json`: Cursor plugin manifest, version 1.0.0.
- `mcp.json`: remote Streamable HTTP MCP server at `https://app.flaghand.com/mcp`, with OAuth and no bundled credential.
- `SKILL.md`: the canonical Flaghand skill. Cursor discovers this root-level file when no `skills/` directory or
  manifest `skills` field exists. Keeping one file avoids a stale copy of the hosted skill.

There is no local server, executable hook, dependency install or background process in this plugin.

## Local plugin installation

Cursor's [local-plugin instructions](https://cursor.com/docs/plugins#test-plugins-locally) load a plugin from
`~/.cursor/plugins/local/`. Clone this public client repository into its own directory there:

```sh
git clone https://github.com/flaghand/flaghand.git ~/.cursor/plugins/local/flaghand
```

Do not overwrite an existing directory. Restart Cursor or run **Developer: Reload Window**, then inspect the
customization panel for the Flaghand skill and MCP server. Enterprise administrators may need to enable local-plugin
imports. A marketplace plugin with the same name takes precedence over a local plugin. This repository being
installable does not mean it has been admitted to Cursor's Marketplace or cursor.directory.

For an existing account, authenticate the Flaghand MCP server through Cursor's normal OAuth flow: the owner signs
in with their passkey, selects the intended workspace and chooses Allow. The grant is limited to `read`, `drafts`,
`media` and `request-approval`, and can be revoked in Flaghand **Settings › Agents**. Call `getMe` to verify the
workspace and scopes before claiming the connection works. Unauthenticated `initialize` returning `401` with
`WWW-Authenticate` and `resource_metadata` is the expected OAuth challenge, not a successful connection.

For a new account or same-session HTTP setup, follow the [setup guide](https://app.flaghand.com/docs/setup.md).
Payment and platform consent stay with the person; tokens stay in secret storage, never in source, prompts or logs.

## MCP-only alternative

The [Cursor setup section](https://app.flaghand.com/docs/agents.md#cursor) also offers a direct MCP install link.
For manual setup, merge the following server entry into the intended project's `.cursor/mcp.json` or user-level
`~/.cursor/mcp.json` without replacing other entries:

```json
{
  "mcpServers": {
    "flaghand": {
      "url": "https://app.flaghand.com/mcp"
    }
  }
}
```

Choose the plugin or a manual same-named MCP entry, not both. The MCP-only option does not install the skill.

## Try it

- “List my Flaghand drafts and tell me which are waiting for approval. Don't edit or publish anything.”
- “Draft a LinkedIn post from these release notes, check it and request my approval. Don't approve, schedule or publish.”

Use the returned Flaghand approval link, not a chat confirmation. Only the owner approves in the desk.

[Agent docs](https://app.flaghand.com/docs/agents.md) · [Privacy](https://flaghand.com/privacy) ·
[Terms](https://flaghand.com/terms) · [Support](mailto:support@flaghand.com)
