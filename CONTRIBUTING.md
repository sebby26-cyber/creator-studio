# Contributing to Creator Studio

Thanks for your interest. Bug reports, skill improvements and new stage skills are all welcome.

## Ground rules

- **No personal or project data.** Never commit projects, voices, character ids, media, API keys, `.env` files,
  `~/.creator-studio/config.json` or anything from your projects folder. Use clearly fictional examples (for example
  `my-channel`).
- **Keep choices binding.** A skill must never silently substitute a model, tool or setting the user chose; it keeps
  the choice or explains which capability is unsupported.
- **Paid steps stay explicit.** Anything that spends credits needs a cost preflight and the user's approval.
- **Instructions, not programs.** The repo ships skills and reference files, no runtime. If a stage needs a command,
  put the exact, tested command in the skill or its `references/` folder (see `audio-master` and
  `clip-assemble/references/ffmpeg-recipes.md`). Say which FFmpeg version you tested with.
- **The agent installs dependencies; the user signs in.** Never tell a skill to leave a missing dependency to the user
  if the agent can install it, and never have an agent enter credentials or pay.

## Referencing shared files across plugins

Each plugin is installed into its own folder, so a skill can only reach files inside its own plugin by relative path.
Shared files (models, dependencies, project file, sidecar) live in `studio-core`. From any other plugin, name the
skill instead of writing a path: "the `studio` skill → `references/dependencies.md`". The agent loads `studio`
and reads the file from there. Never write `../` paths or a bare `references/...` for another plugin's file.

## Adding or changing a skill

1. Create or edit `plugins/<domain>/skills/<id>/SKILL.md` with frontmatter (`name`, `description`) and plain
   **Takes / Makes / Tool / Save** lines.
2. Point to the skill from the stage table in `plugins/studio-core/skills/studio/SKILL.md`.
3. If it needs a new outside tool, add a row to `plugins/studio-core/skills/studio/references/dependencies.md` with a
   check command and an install command, using only sources you can name.
4. Third-party material must keep its license file next to it and be credited in the README.
5. **No scripts in plugins.** No executable or script files (`.sh`, `.py`, `.awk`, `.js` and the like) anywhere under
   `plugins/`. A skill gives the spec (inputs, behaviour, thresholds, exit codes) and pointers; short inline command
   lines in a `.md` are fine as documentation, but an inline block that is really a program (over about 25 lines)
   becomes a spec plus a one-liner. Each project builds and keeps its own reusable tools in its own `tools/` folder.

## Adding a model

Add an entry to `plugins/studio-core/skills/studio/references/models/<provider>.json` with capabilities, inputs,
per-route ids, estimated credits, the price source and the date you verified it. New capability words go in
`capability_vocabulary`.

## Adding a project key

Add it to `plugins/studio-core/skills/studio/references/templates/project.json`, add a row to the table in `plugins/studio-core/skills/studio/references/project-file.md`, and make the
skill that uses it read it. A missing key must always mean "no default".

## Checking your change

Creator Studio is LLM-driven chained skills; verification is by reading the skill chain and measuring outputs.

- Validate the plugin manifests: `claude plugin validate .` from the repo root (Claude Code).
- Load the plugins locally with `claude --plugin-dir plugins/studio-core --plugin-dir plugins/narrative …` and try a
  first run in a scratch folder: "Set up Creator Studio".
- Run every command you added or changed on real or synthetic media.

## Pull requests

- One focused change per pull request, with a short description of what and why.
- By contributing, you agree that your contribution is licensed under the GNU General Public License v3.0 or later.

## Visuals

Banners, icons and graphics follow [BRAND.md](BRAND.md): one accent per repo, the shared palette and type, and SVG
with text outlined to paths. PNG and other media stay out of git; upload the social preview in the repo settings.
