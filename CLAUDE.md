# BetterCallClaude — Plugin Conversion Project

## Project Goal

Convert the existing BetterCallClaude Claude Code framework (`.claude/`) into a **Cowork/Claude Code Plugin** following Anthropic's official plugin format from `anthropics/knowledge-work-plugins` and `anthropics/claude-plugins-official`.

## Current State

This repo is a Swiss Legal Intelligence Framework (v1.3.0) that currently works as a Claude Code configuration (`.claude/` directory). It needs to be repackaged as a plugin that can be installed via:
- `claude plugin marketplace add fedec65/BetterCallClaude`
- Submitted as PR to `anthropics/claude-plugins-official/external_plugins/`
- Uploaded as .zip to Cowork

## Source Material (DO NOT MODIFY these — read them as inputs)

- `.claude/BETTERASK.md` — Main framework entry point
- `.claude/LEGAL_PRINCIPLES.md` — Swiss legal reasoning standards
- `.claude/LEGAL_SYMBOLS.md` — Citation formats DE/FR/IT/EN
- `.claude/SWISS_LAW_CONFIG.md` — Jurisdiction routing, all 26 cantons
- `.claude/personas/` — Legal Researcher, Case Strategist, Legal Drafter
- `.claude/modes/` — Federal law, cantonal law, multilingual modes
- `.claude/commands/` — Slash commands (legal-research, legal-strategy, legal-draft, agent-researcher)
- `.claude/mcp/` — MCP server documentation
- `src/agents/` — Python agent implementations (researcher.py, strategist.py, drafter.py, orchestrator.py)
- `src/integrations/ollama/privacy_mode.py` — Privacy routing with Anwaltsgeheimnis detection patterns
- `mcp-servers/` — TypeScript MCP servers (entscheidsuche, bge-search, legal-citations, shared)

## Target Structure

Create `bettercallclaude-plugin/` directory at repo root:

```
bettercallclaude-plugin/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest (required)
├── .mcp.json                    # MCP server connections
├── commands/                    # Slash commands (at root, NOT inside .claude-plugin/)
│   ├── research.md              # /bettercallclaude:research
│   ├── strategy.md              # /bettercallclaude:strategy
│   ├── draft.md                 # /bettercallclaude:draft
│   ├── cite.md                  # /bettercallclaude:cite
│   ├── federal.md               # /bettercallclaude:federal
│   └── cantonal.md              # /bettercallclaude:cantonal
├── skills/                      # Auto-activated domain knowledge
│   ├── swiss-legal-research/SKILL.md
│   ├── swiss-legal-drafting/SKILL.md
│   ├── swiss-legal-strategy/SKILL.md
│   ├── swiss-citation-formats/SKILL.md
│   ├── swiss-jurisdictions/SKILL.md
│   └── privacy-routing/SKILL.md
├── agents/                      # Subagents (markdown format)
│   ├── researcher.md
│   ├── strategist.md
│   └── drafter.md
├── hooks/
│   └── hooks.json               # Privacy detection hook
├── scripts/
│   └── privacy-check.sh         # Anwaltsgeheimnis pattern detection
├── mcp-servers/                 # Pre-compiled MCP servers (dist/ only)
│   ├── entscheidsuche/dist/
│   ├── bge-search/dist/
│   └── legal-citations/dist/
├── CONNECTORS.md                # MCP server integration docs
└── README.md                    # Plugin-specific README
```

Also create at repo root:
```
.claude-plugin/
└── marketplace.json             # Makes this repo a self-hosted marketplace
```

## Plugin Format Rules (CRITICAL)

1. **`plugin.json`** goes in `.claude-plugin/` — it is the ONLY file in that directory
2. **`commands/`, `skills/`, `agents/`, `hooks/`** go at the PLUGIN ROOT, never inside `.claude-plugin/`
3. **Skills** use `SKILL.md` filename inside a named folder: `skills/topic-name/SKILL.md`
4. **Skills** have YAML frontmatter with `description:` field — this is how Claude decides when to activate them
5. **Commands** are `.md` files with YAML frontmatter containing `description:` — use `$ARGUMENTS` for user input
6. **Agents** are `.md` files with YAML frontmatter containing `name:`, `description:`, `tools:` list
7. **MCP paths** should use `${CLAUDE_PLUGIN_ROOT}` variable for portability
8. **No build steps** for end users — MCP servers must be pre-compiled (include `dist/` folders)

## Conversion Guidelines

### Skills (from .claude/ content)
- Merge `.claude/LEGAL_PRINCIPLES.md` + relevant persona into each skill
- Each SKILL.md should be self-contained with all the knowledge Claude needs
- Include citation format tables, jurisdiction detection rules, quality standards
- Write in imperative voice: "You are a Swiss legal research specialist..."

### Commands (from .claude/commands/)
- Adapt existing `.claude/commands/*.md` to use YAML frontmatter
- Add `$ARGUMENTS` placeholder for user input
- Include output format specifications
- Reference MCP server tools by name

### Agents (from src/agents/*.py)
- Convert Python logic to markdown agent definitions
- List available tools in frontmatter
- Describe workflow steps in markdown
- Include quality standards and constraints

### Hooks
- Privacy detection hook triggers on Write/Edit/Bash tool use
- Scans for Anwaltsgeheimnis/privilege patterns in DE/FR/IT
- Returns `{"decision":"ask","reason":"..."}` when detected

## Key Differentiators (maintain these in all content)

- **All 26 Swiss cantons** with multilingual court systems
- **BGE/ATF/DTF** precedent search via MCP servers
- **Citation verification** across DE/FR/IT/EN
- **Anwaltsgeheimnis** compliance (Art. 321 StGB)
- **Privacy routing** with local LLM support via Ollama
- **Complementary** to Anthropic's generic legal plugin (zero overlap)

## Quality Checks

After creating the plugin:
1. Verify `plugin.json` has all required fields (name, description, version, author)
2. Every skill has YAML frontmatter with `description:`
3. Every command has YAML frontmatter with `description:`
4. Every agent has YAML frontmatter with `name:`, `description:`, `tools:`
5. `.mcp.json` paths use `${CLAUDE_PLUGIN_ROOT}`
6. `hooks.json` matches the PreToolUse format
7. `privacy-check.sh` is executable and handles all three languages
8. No references to `.claude/` directory in the plugin files
9. README.md includes install instructions for all three distribution channels

## Do NOT

- Modify files in `.claude/` — that's the existing Claude Code framework
- Modify files in `src/` — those are standalone Python implementations
- Include node_modules or uncompiled TypeScript in the plugin
- Hardcode file paths — always use `${CLAUDE_PLUGIN_ROOT}`
- Include development artifacts (sprint docs, test reports) in the plugin
