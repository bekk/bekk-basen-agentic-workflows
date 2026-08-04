# bekk-basen-agentic-workflows

Org catalog of [GitHub Agentic Workflows](https://github.com/github/gh-aw) for Basen repos.

Workflows live in `.github/workflows/*.md`. Other repos install them from here; they do not run automatically just because this catalog exists.

## Prerequisites

On the **target** repo (or org):

1. [gh-aw](https://github.com/github/gh-aw) CLI: `curl -sL https://raw.githubusercontent.com/github/gh-aw/main/install-gh-aw.sh | bash`
2. AI engine secret, e.g. `COPILOT_GITHUB_TOKEN` (fine-grained PAT with Copilot Requests: Read), or `ANTHROPIC_API_KEY` / `OPENAI_API_KEY` / `GEMINI_API_KEY`

## Install a workflow

From the target repo:

```bash
gh aw add bekk/bekk-basen-agentic-workflows/update-dependencies
gh aw compile
```

Or deploy from this catalog:

```bash
gh aw deploy ./update-dependencies.md --repo bekk/<target-repo>
```

Commit the `.md` and generated `.lock.yml`, then push.

## Run

```bash
gh aw run update-dependencies --repo bekk/<target-repo>
```

Or: Actions → workflow → Run workflow.

## Workflows

### update-dependencies

Manual dependency updates (vulnerable first, then small safe bumps). Opens a `[deps]` PR or no-ops.

- Trigger: `workflow_dispatch` only
- Skips if an open `[deps]` PR already exists
- Does not run in this catalog repo
