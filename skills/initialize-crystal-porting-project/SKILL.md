---
name: initialize-crystal-porting-project
description: Use when starting a new Crystal porting project from Go (or other language) source code. Sets up project structure, submodules, dependencies, and documentation.
---

# Initialize Crystal Porting Project

## Overview

This skill sets up a Crystal project for porting code from another language (especially Go). It establishes the project structure, adds the source repository as a submodule, configures development dependencies (ameba), creates a Makefile, and prepares documentation.

**Core principle:** Establish a clean project structure that mirrors the source organization and facilitates systematic porting using the `porting-to-crystal` skill.

## When to Use

- Starting a new Crystal porting project from Go source
- Setting up vendor submodules for reference code
- Configuring Crystal project with standard development tools (ameba, formatter)
- Creating project documentation that acknowledges the port

**When NOT to use:**
- Adding new functionality to an existing Crystal port
- Porting individual files (use `porting-to-crystal` instead)
- When you don't have access to the source repository

**Decision Flow:**
1. **New Crystal porting project?** → Use this skill
2. **Have source repository URL?** → Required for submodule
3. **Project directory initialized?** → Should be a fresh or existing Crystal project
4. **Need to port code?** → After setup, use `porting-to-crystal` skill

## Checklist

**IMPORTANT:** Use TodoWrite to create todos for each checklist item below.

- [ ] Gather project information (source URL, subdirectory, vendor directory)
- [ ] Add source repository as git submodule
- [ ] Configure Crystal project (shard.yml with ameba dependency)
- [ ] Create Makefile with standard targets
- [ ] Document the port in README.md
- [ ] Create AGENTS.md file with project-specific instructions
- [ ] Initialize porting workflow with `porting-to-crystal` skill

## Prerequisites

Before using this skill, ensure:
- Git is installed and configured
- Crystal is installed (for formatting and testing)
- You have write access to the project directory
- You know the source repository URL

## The Process

### Step 1: Gather Project Information

Ask the user for:
1. **Source repository URL** - The Go (or other language) source to port
2. **Source subdirectory** - If the source is within a subdirectory (e.g., `x/ansi`)
3. **Project name** - Crystal shard name (default: infer from directory)
4. **Vendor directory name** - Where to place submodule (default: `vendor`)

**Example questions:**
- "What is the URL of the source repository you're porting from?"
- "Is the source code in a subdirectory (e.g., 'x/ansi') within that repository?"
- "What should we name the vendor submodule directory? (default: 'vendor')"

### Step 2: Create Vendor Submodule

Add the source repository as a git submodule:

```bash
git submodule add <source_url> <vendor_directory>
```

If the source is in a subdirectory (e.g., `x/ansi`), note this for reference but keep the submodule at the repository root. The subdirectory will be used as the source path for porting.

**Important:** Submodules track the entire repository, not subdirectories. Record the subdirectory path for reference during porting.

### Step 3: Configure Crystal Project

Ensure `shard.yml` exists (create if needed). Add development dependencies:

```yaml
development_dependencies:
  ameba:
    github: crystal-ameba/ameba
    version: ~> 1.0
```

Also add any runtime dependencies that correspond to Go dependencies (e.g., `colorful` for `go-colorful`).

### Step 4: Create Makefile

Create a `Makefile` with standard targets:

```makefile
.PHONY: install update format lint test

install:
	BEADS_DIR=$$(pwd)/.beads shards install

update:
	BEADS_DIR=$$(pwd)/.beads shards update

format:
	crystal tool format --check

lint:
	ameba --fix
	ameba

test:
	crystal spec

clean:
	rm -rf ./temp/*
```

Adapt based on existing project patterns. Include any project-specific targets.

### Step 5: Document the Port

Read the source repository's README (especially from the subdirectory). Create or update the project README.md:

1. **Title and description** - Indicate this is a Crystal port
2. **Source attribution** - Include URL and version/submodule hash
3. **Installation** - Standard Crystal shard instructions
4. **Usage** - Examples adapted from source documentation
5. **Development** - Reference the Makefile targets
6. **Contributing** - Link to porting guidelines
7. **Issue tracking** - If the project uses beads (like the ansi port), follow the issue tracking workflow in AGENTS.md

**Template:** "This is a Crystal port of [source repo] (specifically [subdirectory]). See vendor/ directory for the original source code."

### Step 6: Create AGENTS.md File

Create an `AGENTS.md` file with project-specific instructions for agents working on the port. Use this template and replace placeholders:

```markdown
# Agent Instructions

This is a port of golang <SOURCE_REPOSITORY_URL> to
Crystal language. Since it is a port, all logic must match the golang implementation
only differening in Crystal language idioms and libs. If you have
a question, the go code is the source of truth. We want to port all go code and
go tests. The golang src is available at ./<VENDOR_DIRECTORY>/<SOURCE_SUBDIRECTORY>

This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get
started.

## Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --status in_progress  # Claim work
bd close <id>         # Complete work
bd sync               # Sync with git
```

## Issue Tracking Workflow

**DO NOT use internal todo list or task list tools** - Use beads for all issue
tracking and work management:

1. **Strategic, multi-session work**: Track in beads using `bd create`
2. **Dependencies and blockers**: Use `bd dep add` to link issues
3. **Session planning**: Use `bd ready` to find unblocked work
4. **Progress tracking**: Use `bd update <id> --status in_progress` when
   starting, `bd close <id>` when complete

**When creating multiple tasks**: Use parallel subagents for efficiency with
`bd create` commands.

**Example workflow**:

```bash
# Create issues for test porting
bd create --title="Port [module] tests from Go to Crystal spec" --type=task --priority=2
bd create --title="Port [module] tests from Go to Crystal spec" --type=task --priority=2
bd create --title="Port [module] tests from Go to Crystal spec" --type=task --priority=2

# Claim work
bd update beads-xxx --status=in_progress

# Complete work
bd close beads-xxx beads-yyy beads-zzz
```

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT
complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs
   follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
   - **Crystal-specific quality gates**:

      ```bash
      crystal tool format --check
      ameba --fix
      ameba
      crystal spec
      ```

      Ensure no formatting issues remain, all ameba errors are fixed, and all
      tests pass before committing.
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:

   ```bash
   git pull --rebase
   bd sync
   git push
   git status  # MUST show "up to date with origin"
   ```

5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**

- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds

## Crystal Development Guidelines

This is a Crystal port of the Go code from `./<VENDOR_DIRECTORY>/<SOURCE_SUBDIRECTORY>`. Follow Crystal idioms and
best practices:

- Use Crystal's built-in formatter: `crystal tool format`
- Use ameba for linting: `ameba --fix` then `ameba` to verify
- Prefer Crystal's standard library over custom implementations
- Use Crystal's type system effectively (avoid unnecessary `as` casts)
- Follow Crystal naming conventions (snake_case for methods/variables, CamelCase
  for classes)
- Write specs for new functionality using Crystal's built-in spec framework

### Test Porting Guidelines

When porting Go tests to Crystal specs:

1. **Port test logic exactly** - Don't adjust test assertions or expected values
2. **Use Crystal idioms for structure** - Convert Go test tables to Crystal `it`
   blocks
3. **Mark missing functionality as pending** - Use `pending` for tests that
   can't run yet
4. **Follow Go test patterns** - Maintain the same test coverage and edge cases
5. **Verify against Go implementation** - Ensure Crystal behavior matches Go
   exactly

Example: If a Go test expects compressed data of specific size, replicate that
check in Crystal even if Crystal's compression produces slightly different
output.

## File System Guidelines

- Use `./temp` directory for temporary files created during testing or
  development
- Never commit temporary files to git (they are already in `.gitignore`)
- Clean up temporary files after use (the `make clean` rule removes `./temp`
  contents)
```

**Replace placeholders:**
- `SOURCE_REPOSITORY_URL`: The Go repository URL (e.g., `https://github.com/charmbracelet/x/tree/main/ansi`)
- `VENDOR_DIRECTORY`: The vendor submodule directory (default: `vendor`)
- `SOURCE_SUBDIRECTORY`: The subdirectory within the repository containing the source (e.g., `x/ansi`)

### Step 7: Initialize Porting Workflow

After project setup, use the `porting-to-crystal` skill for actual code porting:

1. **Announce:** "Using porting-to-crystal skill for code translation"
2. **Load skill:** Use the Skill tool to load `porting-to-crystal`
3. **Follow skill:** Implement the porting patterns for constants, functions, tests

## Quick Reference

| Task | Command/Pattern |
|------|----------------|
| Add submodule | `git submodule add <url> vendor` |
| Update submodule | `git submodule update --init --recursive` |
| Add ameba dependency | Add to `shard.yml` development_dependencies |
| Format check | `crystal tool format --check` |
| Lint | `ameba --fix && ameba` |
| Run tests | `crystal spec` |

## Common Mistakes

1. **Forgetting to init submodule** - Run `git submodule update --init --recursive` after cloning
2. **Wrong subdirectory path** - Verify source code location within submodule
3. **Missing development dependencies** - Ensure ameba is in `shard.yml`
4. **No source attribution** - README must acknowledge the port source
5. **Skipping documentation** - Porting projects need clear attribution

## Integration with porting-to-crystal

This skill sets up the project structure; `porting-to-crystal` handles the actual code translation. Use them together:

1. **initialize-crystal-porting-project** → Project setup
2. **porting-to-crystal** → Code translation
3. **systematic-debugging** → Fix test failures
4. **verification-before-completion** → Quality gates

**Project-specific guidelines:** Some projects have additional porting guidelines (e.g., AGENTS.md in the ansi project). Review and follow those guidelines alongside these skills.

## Red Flags

| Thought | Reality |
|---------|---------|
| "I'll skip the submodule, I can just copy files" | Submodules preserve source history and enable updates |
| "Documentation can wait" | Attribution is critical for porting projects |
| "I don't need ameba for now" | Linting ensures code quality from the start |
| "Makefile is unnecessary" | Standard targets ensure consistent development workflow |
| "I'll figure out dependencies later" | Missing dependencies block compilation and testing |

## See Also

- [porting-to-crystal](./porting-to-crystal) - Code translation patterns
- [systematic-debugging](../systematic-debugging) - Debugging test failures
- [verification-before-completion](../verification-before-completion) - Quality gates before completion
- [test-driven-development](../test-driven-development) - TDD for feature implementation

## Real-World Impact

Following this skill ensures:
- **Reproducible source reference** - Submodules with exact versions
- **Consistent development** - Standard Makefile and linting
- **Proper attribution** - Clear documentation of port source
- **Systematic porting** - Integration with `porting-to-crystal` skill
- **Quality foundation** - Established patterns for future work