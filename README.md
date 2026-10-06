# Procedural World Generation Skills

Three short agent skills that let an LLM coding assistant build a procedural
world generator from scratch for a 2D, 2.5D, or 3D game in any engine.
Works with Claude Code, Codex, Opencode, Cursor, and any assistant that reads
`AGENTS.md` and a `skills/` folder (or `.agents/skills` / `.claude/skills`).

The skills follow one rule: tell the model only what it cannot know on its own.
Goal, decisions to lock, build order, definition of done, and the mistakes
that recur in this domain. The model chooses the algorithms and writes the
code in the project's own language.

| Skill | Purpose |
|-------|---------|
| `worldgen` | Entry point: decisions to lock, build order, definition of done |
| `worldgen-pitfalls` | Recurring worldgen mistakes that surface late |
| `worldgen-sites` | Towns, dungeons, and structures that need spacing or connectivity guarantees |

## Installation

Copy `skills/` and `AGENTS.md` into your game project. Put the skills
wherever your assistant looks for them, for example `.agents/skills/` or
`.claude/skills/`.

```bash
mkdir -p /path/to/your-game-project/.agents && cp -r skills /path/to/your-game-project/.agents/skills && cp AGENTS.md /path/to/your-game-project/
```

If your project already has an `AGENTS.md`, append the short worldgen section
to it instead of overwriting.

## Usage

Open the project in your assistant and ask, for example:

> Build a procedural world generation system for my game

> Generate a world: 2D top-down, infinite, with biomes, caves and towns

The assistant writes `worldgen/DECISIONS.md`, then builds the generator stage
by stage (terrain, biomes, carving, sites, scatter, debug map), showing a
result after each stage.

## License

MIT
