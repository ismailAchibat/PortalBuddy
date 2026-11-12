# Unreal Engine MCP: install and connect (EngineWire plugin + Claude Code)

This is the setup that worked in session 2 on the university PC (UE 5.6.1, Windows 11, PowerShell).
Statements marked **(not verified on this PC)** come from general knowledge, not from something we did.

## How it works in one paragraph

A free Unreal plugin called **EngineWire** (Fab, by Chirag Panwar, MIT licence, source:
https://github.com/ChiR24/Unreal_mcp) runs an MCP server **inside the editor** at
`http://127.0.0.1:3000/mcp`. It exposes a single tool named `unreal` with four operations:
`search`, `describe`, `execute`, `configure`. Claude Code connects to it through a project file
`.mcp.json` that holds a secret token (`X-MCP-Capability-Token`) which the plugin generates in
`Saved/MCP/capability-token`. The editor must be open for the connection to exist.

Do **not** use Epic's own `ModelContextProtocol` / `AllToolsets` plugins: they do not exist in UE 5.6.1.

## 1. Claude Code

`claude` was **not** on the PATH of the PowerShell used by the tools in session 2, so we do not know which
install route the user used. Any of these works **(not verified on this PC)**:

- Native installer, in PowerShell: `irm https://claude.ai/install.ps1 | iex`
- Or with Node.js: `npm install -g @anthropic-ai/claude-code`
- Or the desktop app / VS Code extension if the user prefers them.

Behind the university proxy, if the download fails, set `HTTPS_PROXY` and `HTTP_PROXY` to
`http://proxy.univ-lemans.fr:3128` for that terminal. Then log in with `claude` (browser sign-in).
Always start Claude Code **from the project folder** so it loads `CLAUDE.md` and `.mcp.json`.

## 2. The plugin in the engine

It was installed from Fab into the **engine** (not the project). In session 2 it lives at
`D:\PROGRAMMES\Epic Games\UE_5.6\Engine\Plugins\Marketplace\MCPAutom3e341f585dc4V3\EngineWire.uplugin`
(version 0.6.0). To check on a new session:

```powershell
Get-ChildItem 'D:\PROGRAMMES\Epic Games\UE_5.6\Engine\Plugins' -Recurse -Filter EngineWire.uplugin | Select-Object -First 1 -ExpandProperty FullName
```

If nothing is found: open the Epic Games Launcher, go to **Fab**, search **EngineWire**, add it to the
library (free), then in the launcher's **Library > Fab Library** choose **Install to Engine** for UE 5.6.
Restart the launcher/editor if it does not appear.
If the engine itself is missing on that PC, UE 5.6.1 must be reinstalled first (needs a PC where it is allowed).

## 3. Project settings (already in the repo, check them after a fresh clone)

`PortalBuddy.uproject` must contain **all of these plugins** (this was the cause of the first big problem:
without the two MovieRender ones the plugin fails to load with missing-DLL errors and **the editor closes**):

```json
{ "Name": "MovieRenderPipeline", "Enabled": true },
{ "Name": "MoviePipelineMaskRenderPass", "Enabled": true },
{ "Name": "EngineWire", "Enabled": true, "SupportedTargetPlatforms": ["Win64", "Mac", "Linux"] }
```

`Config/DefaultGame.ini` must contain (it is committed, so the server starts automatically):

```ini
[/Script/EngineWire.EngineWireSettings]
bEnableNativeMCP=True
```

The same switch is in the editor at **Project Settings > Plug-ins > EngineWire > Native MCP >
Enable Native MCP Server** (French UI: Parametres du projet > Plug-ins).
When the server runs, the editor status bar (bottom right) shows `MCP :3000 (0)`; the number in
parentheses becomes `(1)` when a client (Claude Code) is connected.

## 4. Create `.mcp.json` (git-ignored, contains the secret)

The plugin writes the token to `Saved/MCP/capability-token` when the server starts (so open the editor
first). Then, from the project folder, run the script in this handoff folder:

```powershell
cd 'H:\Home\Bureau\UnrealProjects\Projet-unreal-engine\PortalBuddy'
powershell -ExecutionPolicy Bypass -File 'H:\Home\Bureau\PortalBuddy_Claude_Handoff\Setup-McpJson.ps1'
```

It produces this file (the token is filled in, never printed):

```json
{
  "mcpServers": {
    "unreal-engine": {
      "type": "http",
      "url": "http://127.0.0.1:3000/mcp",
      "headers": { "X-MCP-Capability-Token": "<token from Saved/MCP/capability-token>" }
    }
  }
}
```

Never commit `.mcp.json` and never paste the token anywhere. It is in `.gitignore`. If the token ever
leaks, delete `Saved/MCP/capability-token`, restart the editor so the plugin makes a new one, and re-run the script.
If the connection starts failing with an authentication error after an editor restart, re-run the script:
the token may have changed.

## 5. The university proxy (this was the second big problem)

The Squid proxy answers **403** to requests for `127.0.0.1` if they go through it. Make sure loopback
bypasses the proxy, in the terminal that starts Claude Code:

```powershell
$env:NO_PROXY = '127.0.0.1,localhost,::1'
$env:no_proxy = $env:NO_PROXY
```

Direct test of the server (prints only the HTTP status, not the token). `200` or `400` here means the
server is reachable and the proxy is not in the way; `403` means the proxy is still intercepting:

```powershell
$t = (Get-Content 'Saved\MCP\capability-token' -Raw).Trim()
curl.exe --noproxy '*' -s -o NUL -w '%{http_code}' -H "X-MCP-Capability-Token: $t" http://127.0.0.1:3000/mcp
```

In session 2 the direct authenticated test returned 200.

## 6. Connect from Claude Code

1. Editor open on the project, status bar shows `MCP :3000 (0)`.
2. In the project folder run `claude`. The first time it asks whether to trust the project's `.mcp.json`
   server `unreal-engine`: accept. (This writes `.claude/settings.local.json` with
   `"enabledMcpjsonServers": ["unreal-engine"]` and `"enableAllProjectMcpServers": true`; that file is
   git-ignored, so it must be recreated on a new machine by accepting once.)
3. Type `/mcp` and check that `unreal-engine` is **connected**. The editor status bar shows `(1)`.
4. First call must be read-only, for example: list the actors of the level (`control_actor` `list`).

### How to use the tool (rule of the server itself)

1. `search` with 2-4 plain words (verb + object), e.g. "list level actors".
2. `describe` with the row's `nextCall` (gives the exact parameters, examples and any consent grant).
3. `execute` with only the parameter names `describe` listed.
4. Read the reply's `warnings[]` (placement, z-fighting, default materials...) and fix before moving on.
Asset paths start with `/Game`, class paths like `/Script/Engine.Actor`, vectors are `{x,y,z}` objects
(some calls take `[x,y,z]` arrays: follow `describe`).

## 7. Troubleshooting

| Symptom | Cause / fix |
|---|---|
| Editor closes right after opening the project | `MovieRenderPipeline` / `MoviePipelineMaskRenderPass` missing in the `.uproject` (section 3) |
| `/mcp` shows failed, `403` in tests | proxy intercepts loopback: set `NO_PROXY` (section 5), restart Claude Code from that terminal |
| `NOT_CONNECTED` from the tool | editor not running, or server disabled (`bEnableNativeMCP`), status bar must say `MCP :3000` |
| Authentication error / 401 | token changed: re-run `Setup-McpJson.ps1`, restart Claude Code |
| Plugin not found / project asks to rebuild | EngineWire not installed in this engine (section 2) |
| `PIE_ACTIVE` | stop Play In Editor first (`control_editor` `play`, control `stop`) |
| `DIRTY_PACKAGES` when creating/opening a level | save first or pass `saveDirtyPackages: true` |
| `CONSENT_REQUIRED` / "the user did not allow" | the user's permission prompt refused it: do not retry unless the user asks |
| Tool returns huge output | spawn batches echo every actor; expect it, do not re-run just to shorten it |

## 8. Safety rules for the session

Read before write; commit before big edits; save the project before big edits; never show or commit the
token; no commit or push unless the user asks.
