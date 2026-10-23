# magpie-agent-craft-agents

[Craft Agents](https://www.craft.do) (Craft's desktop agent app) as a magpie
agent. magpie has no setup of its own for Craft Agents; with this plugin it is
on magpie's Agents page like the agents magpie ships with: pick one of magpie's
models in its row and Craft Agents talks to magpie's gateway.

```sh
magpie plugin add /path/to/magpie-agent-craft-agents
```

or, once it is on npm:

```sh
magpie plugin add @magpie-community/agent-craft-agents
```

Then pick one of magpie's models in Craft Agents' row, or

```sh
magpie craft-agents magpie/deepseek/deepseek-flash
```

magpie writes into `~/.craft-agent/config.json` — the connection Craft Agents
then uses, with magpie's address, the picked model, and magpie's model list:

```json
{
  "llmConnections": [
    {
      "slug": "magpie",
      "name": "Magpie",
      "providerType": "pi_compat",
      "authType": "none",
      "baseUrl": "http://127.0.0.1:3425/v1",
      "defaultModel": "deepseek/deepseek-flash",
      "models": [{ "id": "deepseek/deepseek-flash", "name": "DeepSeek Flash", "contextWindow": 128000, "supportsImages": false }],
      "customEndpoint": { "api": "openai-completions" }
    }
  ],
  "defaultLlmConnection": "magpie"
}
```

Everything else in `config.json` — workspaces, sessions, migrations — is left
alone. Picking Craft Agents' own model again, or switching the plugin off,
puts every connection back as it was, in its original order.

## How it fits together

- `connect()` returns the whole `llmConnections` array, because Craft stores a
  connection as one object; magpie saves the array it replaces first, so the
  restore is exact.
- `authType: "none"` means no stored credential: Craft sends the placeholder
  `not-needed` to a localhost endpoint, which magpie's gateway accepts. Nothing
  in Craft's encrypted `credentials.enc` is touched.
- `customEndpoint.api: "openai-completions"` is what Craft's own "API Key"
  setup writes for an OpenAI-compatible endpoint. `pi_compat` connections get
  no default model list from Craft, so the list magpie writes stays.
- Craft reads the connection when a session starts and applies a model change
  to running sessions without a restart.
- Craft's backend sends pi's generic User-Agent (`pi (darwin …)`), which
  magpie's built-in Pi agent already claims, and `authType: "none"` sends no
  gateway key. So magpie's Usage lists Craft's requests under **pi**, not
  Craft Agents. The model, tokens and cost are right; only the agent name is.
  Declaring `ua: ["pi ("]` here does not fix it — magpie's own Pi agent
  matches first — and declaring an unrelated `ua` would never match at all.

## Develop

```sh
magpie plugin add .                       # install this checkout
magpie craft-agents magpie/<provider>/<model>
magpie craft-agents default               # put Craft's own connections back
```

`agent.config` must match `CONFIG_FILE` in `@craft-agent/shared/config/paths`.
The icon is embedded in `icon.js` so loading needs no network: it is Craft
Agents' own artwork rendered at 256×256 with the card full-bleed and the glyph
at 80% of the tile, so it carries the same visual weight as the built-in
agents' icons at the 16–20px magpie draws them at. (The app's `.icns` sits on
~9% transparent margin, which made it read a size smaller.) `icon-preview.png`
is that rendering next to the icon it replaced.

How agent plugins work: [usemagpie.ai/docs/plugins#agents](https://usemagpie.ai/docs/plugins#agents).
