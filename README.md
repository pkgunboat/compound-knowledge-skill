# Compound Knowledge Skill

A global knowledge compounding system for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Capture, search, and maintain reusable problem-solving experience across all your projects.

Inspired by [Compound Engineering Plugin](https://github.com/EveryInc/compound-engineering-plugin)'s knowledge compounding philosophy — lightweight, dependency-free, and designed for individual developers.

## What It Does

| Command | Purpose | When to Use |
|---------|---------|-------------|
| `/compound` | Extract and store knowledge from current conversation | After fixing a bug, finishing debugging, or learning something new |
| `/compound-search` | Search the knowledge base for relevant experience | Before debugging, when encountering errors, or planning a solution |
| `/compound-refresh` | Audit and maintain the knowledge base | Periodically, or after major refactors/migrations |

## How It Works

```
  Fix a bug ──► /compound ──► Knowledge stored
                                    │
                                    ▼
                          ~/.claude/knowledge/
                          └── solutions/
                              ├── build-errors/
                              ├── runtime-errors/
                              ├── patterns/
                              └── ...
                                    │
  Hit a similar bug ──► /compound-search ──► Found! Apply previous solution
```

### Knowledge Format

Each entry is a structured Markdown file with YAML frontmatter, stored in one of two tracks:

**Bug Track** — for problem fixes:
- Problem description → Symptoms → Failed attempts → Solution → Root cause → Prevention

**Knowledge Track** — for patterns and best practices:
- Context → Key points → Why it matters → When to apply → Real examples

### Deduplication

Before creating a new entry, `/compound` searches existing documents to avoid duplicates:
- **High overlap** → Updates existing doc
- **Moderate overlap** → Creates new doc with cross-references
- **No overlap** → Creates new doc

## Installation

### Quick Install (recommended)

```bash
# Clone the repo
git clone https://github.com/pkgunboat/compound-knowledge-skill.git /tmp/compound-knowledge-skill

# Copy skills to Claude Code
cp -r /tmp/compound-knowledge-skill/skills/compound ~/.claude/skills/
cp -r /tmp/compound-knowledge-skill/skills/compound-refresh ~/.claude/skills/
cp -r /tmp/compound-knowledge-skill/skills/compound-search ~/.claude/skills/

# Create the knowledge store directory
mkdir -p ~/.claude/knowledge/solutions

# Clean up
rm -rf /tmp/compound-knowledge-skill
```

### Manual Install

1. Download the three `SKILL.md` files from the [`skills/`](./skills/) directory
2. Place them into your Claude Code skills directory:
   ```
   ~/.claude/skills/
   ├── compound/SKILL.md
   ├── compound-refresh/SKILL.md
   └── compound-search/SKILL.md
   ```
3. Create the knowledge store:
   ```bash
   mkdir -p ~/.claude/knowledge/solutions
   ```

### Verify Installation

Start a new Claude Code session and check that the three commands appear in your skill list:
```
/compound
/compound-search
/compound-refresh
```

## Multi-Device Sync

The knowledge base supports automatic GitHub sync. Every time `/compound` or `/compound-refresh` modifies the knowledge store, changes are automatically committed and pushed to a private GitHub repo.

### Setup (once per device)

**First device (initialize):**

```bash
# Initialize knowledge store as a git repo
cd ~/.claude/knowledge
git init
git add -A
git commit -m "init: knowledge base"

# Create a private GitHub repo and push
gh repo create knowledge-base --private --description "Personal knowledge base" --source . --push
```

**Additional devices:**

```bash
# Clone your knowledge base
git clone git@github.com:<YOUR_USERNAME>/knowledge-base.git ~/.claude/knowledge
```

### How Sync Works

```
Device A                          GitHub                         Device B
   │                                │                               │
   │  /compound                     │                               │
   │  ──► write doc                 │                               │
   │  ──► git commit + push ──────► │                               │
   │                                │                               │
   │                                │   /compound-search            │
   │                                │ ◄────── git pull --rebase ──  │
   │                                │         search locally        │
   │                                │                               │
```

- `/compound` — after writing, auto runs: `git add → commit → pull --rebase → push`
- `/compound-refresh` — after all edits, auto runs the same sync flow
- `/compound-search` — read-only, no sync needed (but you can manually `git pull` to get latest)
- If push fails (e.g. network issue), the skill continues normally and reminds you to push later

## Knowledge Categories

Documents are auto-classified into these categories:

| Category | Description |
|----------|-------------|
| `build-errors/` | Build, compile, dependency errors |
| `test-failures/` | Test failures, assertion errors |
| `runtime-errors/` | Runtime exceptions, crashes |
| `performance-issues/` | Performance bottlenecks, memory leaks |
| `database-issues/` | Database related problems |
| `security-issues/` | Security vulnerabilities |
| `ui-bugs/` | Frontend / UI issues |
| `integration-issues/` | API, third-party service integration |
| `logic-errors/` | Business logic defects |
| `config-issues/` | Configuration, environment issues |
| `patterns/` | General patterns, best practices |
| `tools/` | Tool tips, CLI commands, dev environment |

## Usage Examples

### After fixing a tricky bug

```
You: /compound
Claude: Analyzes the conversation, extracts the fix, and stores it:

已沉淀知识:
  文件: ~/.claude/knowledge/solutions/runtime-errors/pytorch-cuda-oom-with-gradient-accumulation.md
  类型: bug-track
  标签: [pytorch, CUDA, OOM, gradient-accumulation]
  摘要: PyTorch 梯度累积时未正确使用 no_grad 导致显存溢出
```

### When encountering a familiar-looking error

```
You: /compound-search
Claude: Searches for relevant entries and shows matches:

找到 2 份相关经验:

1. PyTorch CUDA OOM with Gradient Accumulation ⟵ 高度相关
   文件: ~/.claude/knowledge/solutions/runtime-errors/pytorch-cuda-oom-with-gradient-accumulation.md
   标签: [pytorch, CUDA, OOM]
   匹配原因: 错误信息包含 CUDA out of memory
```

## Design Philosophy

- **Global, not per-project** — Knowledge lives in `~/.claude/knowledge/` and is accessible from any project
- **Lightweight** — No dependencies, no parallel subagents, single-pass execution
- **Failed attempts are valuable** — The "what didn't work" section is often more useful than the solution itself
- **Quality over quantity** — Only record insights with reuse value; not every bug is worth documenting
- **Cross-project abstraction** — Documents should be generalized enough to help in different projects

## Uninstall

```bash
# Remove skills
rm -rf ~/.claude/skills/compound
rm -rf ~/.claude/skills/compound-refresh
rm -rf ~/.claude/skills/compound-search

# Optionally remove the knowledge store (⚠️ this deletes all your saved knowledge)
# rm -rf ~/.claude/knowledge
```

## License

MIT
