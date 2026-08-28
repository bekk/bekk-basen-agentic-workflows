---
description: Update vulnerable and outdated dependencies; open a PR when changes are needed
on:
  workflow_call:
  workflow_dispatch:
  skip-if-match: 'is:pr is:open in:title "[deps] "'

inlined-imports: true

timeout-minutes: 30

permissions:
  contents: read
  pull-requests: read
  security-events: read
  copilot-requests: write

network:
  allowed:
    - defaults
    - node
    - python
    - go
    - rust
    - java
    - dotnet

tools:
  github:
    mode: gh-proxy
    toolsets: [repos, pull_requests, code_security]
  bash: true

safe-outputs:
  create-pull-request:
    title-prefix: "[deps] "
    labels: [dependencies, automation]
    draft: false
    max: 1
    protected-files: allowed
    if-no-changes: ignore
---

# Update Dependencies

Update dependencies in this repo. Prefer security fixes. Keep changes small.

## Do

1. Detect package manager(s) from lock/manifest files (`package.json`, `go.mod`, `pyproject.toml` / `requirements.txt`, `Cargo.toml`, `pom.xml`, `*.csproj`, etc.).
2. List known vulnerabilities (`gh api` code scanning / Dependabot alerts if available; otherwise ecosystem audit commands).
3. Apply the smallest safe upgrades:
   - **Vulnerable packages first** (patch/minor that fix CVEs).
   - Then other outdated direct deps only if the bump is low-risk (patch, or minor with no breaking signals).
4. Refresh lockfiles. Do not invent new dependencies or major-version jumps unless required to fix a vulnerability and no safer path exists.
   - For transitive vulnerabilities, prefer a plain lockfile-only fix (e.g. `npm update <pkg>` / regenerated 
     lockfile) over adding an `overrides` entry. Check whether the parent dependency's own declared range already permits the patched version — if so, a lockfile refresh alone resolves it. Only add an override when the dependency tree has a genuine conflicting range that prevents natural resolution to the patched version, and note that reasoning explicitly in the PR body.
5. Run the repo’s normal install/build/test commands if they are obvious and cheap. Skip heavy suites.
6. If files changed, open one PR via `create-pull-request`. Title: `[deps] …`. Body: what changed, why (CVE/alert IDs when known), and how you verified.
7. If nothing needs updating, call `noop`.

## Don’t

- Multi-ecosystem kitchen-sink PRs when one focused PR is enough.
- Refactors, formatting-only churn, or doc edits.
- Force major upgrades “because latest”.
- Touch unrelated files.
