# VektorCode dashboard

Local, dependency-free Python dashboard for model selection, health, sessions, tools, Composio status, receipts, and the kill switch.

Run:

```sh
cd ~/vektorcode
python3 main.py
```

Open http://127.0.0.1:8765. The server binds to `0.0.0.0` and honors Render's `PORT` variable (or `VEKTORCODE_PORT` locally). By default it calls local Ollama at `http://127.0.0.1:11434/v1/chat/completions` with model `llama3.2`.

For a hosted provider, set the provider-specific API key (for example `GROQ_API_KEY` or `OPENAI_API_KEY`) before starting the dashboard. The key stays server-side.

VektorCode reads the shared identity and memory from `../vektor/SOUL.md` and
`~/.vektor/memories/`. Explicitly approved notes can be saved with
`POST /api/memory/save` (`category`: `USER` or `MEMORY`) and tool/skill notes
with `POST /api/skill/save`. These endpoints require the configured tool
password when `VEKTORCODE_TOOL_PASSWORD` is set; nothing is saved automatically
from chat.

## Skills (skill-system-termux)

Skills are instruction packs the chat model follows when Wallace invokes
them. A skill is a directory with a `SKILL.md` (frontmatter `name` /
`description` + Markdown body) and optional `references/` files, or a
loose Markdown note saved via `POST /api/skill/save`.

- Repo skills ship with the code: `skills/<slug>/SKILL.md`. State notes
  live in `$VEKTOR_STATE_DIR/skills/` (same dir `/api/skill/save` writes).
  The loader (`vektor_cloud/skills.py`) reads both; repo wins on a slug
  collision. Only these two roots are ever read — the model can never
  load an arbitrary path.
- Invoke in chat with `/skill <slug>` (e.g. `/skill hypit`) or a `skill`
  field in the `/api/chat` payload. The skill body loads as a system
  message for that turn, and its reference files are readable via the
  `read_skill_reference` tool (read-only, jailed to the skill directory).
- `GET /api/skills` lists installed skills; `/api/state` includes them.
- Installed: `hypit` — Hypit's video-production skill (copied from
  hypit-ai/hypit `skills/hypit/`, 71 files). It guides the model in
  authoring video workflows; rendering still needs the Hypit runtime
  (Node 22 + headless Chromium) — see `docs/hypit-runtime-findings.md`.

Termux note: the cloud approval-gated shell from the render branch was
deliberately NOT ported — Termux already has a native shell, so there is
no `run_shell` tool here. The Hypit runtime script
`scripts/install-node.sh` was also left out: it installs the Linux x86_64
Node tarball for Render, which cannot run on Termux (Android/aarch64).
On Termux, Node comes from `pkg install nodejs` instead.
