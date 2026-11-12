# PortalBuddy - Claude handoff folder

Written at the end of work session 2 (2026-11-06) so that a brand new Claude session, on a freshly
logged-in university PC, can pick up exactly where we stopped.

The user (Ismail) speaks French and English. The project, the course and the in-game text are in French.
Answer in the language the user is currently writing in.

## What is in this folder

| File | What it is |
|---|---|
| `00_READ_ME_FIRST.md` | This file: the 5-minute restore procedure |
| `01_PROJECT_HANDOFF.md` | Everything about the project: state, design, decisions, gotchas, next steps |
| `02_MCP_SETUP_GUIDE.md` | How to install and connect the Unreal Engine MCP (the part that was painful) |
| `03_PROMPT_TO_PASTE.txt` | The first message to paste into the new Claude session |
| `Setup-McpJson.ps1` | Script that writes the project's `.mcp.json` from the plugin token (never prints the token) |

## 5-minute restore procedure

1. **Get the project back.** The project lives on the network drive at
   `H:\Home\Bureau\UnrealProjects\Projet-unreal-engine\PortalBuddy`. If it is gone, clone it:
   `git clone https://github.com/ismailAchibat/PortalBuddy.git` and then
   `git checkout feature/puzzle-rooms` (the latest work is on that branch, pushed to GitHub).
   On the network drive, always run git as `git -c safe.directory=* <command>`.
2. **Install Claude Code** if it is missing (see section 1 of `02_MCP_SETUP_GUIDE.md`).
3. **Check the Unreal side**: UE 5.6.1 installed, the EngineWire plugin present in the engine
   (see section 2 of the MCP guide). The project's `.uproject` and `Config/DefaultGame.ini`
   already contain the required settings, so nothing to edit after a clone.
4. **Open the project in Unreal**, wait for `MCP :3000` to appear in the editor status bar.
5. **Run `Setup-McpJson.ps1`** from the project folder to recreate `.mcp.json` with the fresh token.
6. **Start Claude Code in the project folder** and paste `03_PROMPT_TO_PASTE.txt`.
   Claude will read `CLAUDE.md` (auto-loaded, it is in the repo) and then `01_PROJECT_HANDOFF.md`.

## Hard rules to give the new Claude (the user stated these)

- Never show or commit the MCP token. `.mcp.json` and `Saved/` are git-ignored; keep it that way.
- No commit and no push unless the user asks for it in that message. Branch first if on `main`.
- Read before write. Ask the user to save the project before any big modification.
- Do not retry `create_material` (the user's permission prompt refused it once). Only if they ask.
- End commit messages with the `Co-Authored-By` line given by the session's own system reminder.
