---
name: add-deploy-action
description: Add a GitHub Actions workflow that runs `ait deploy` on push to main, registering a new deployment in the Apps In Toss console. Use when the user asks to add 자동 배포 / GitHub Action / CI 배포 / 앱인토스 자동 배포 / ait deploy workflow / deploy on push to an existing Apps In Toss mini-app project.
allowed-tools: Bash(ls *) Bash(test *) Bash(cat *) Bash(grep *) Bash(git *) Bash(mkdir *) Read Write Edit Glob Grep
---

# Add an Apps In Toss deploy GitHub Action

Add a GitHub Actions workflow at `.github/workflows/deploy.yml` that builds the mini-app and runs `ait deploy` to register a new deployment in the 앱인토스 콘솔. The workflow uploads a deployment — it does **not** publish to end users. Public release still goes through the console review flow.

> Note: This skill writes a single workflow file plus a couple of human-readable instructions. It does **not** modify the user's deploy pipeline outside this repo, and it never touches `~/.ait/credentials` on the developer's machine.

## When to use

- The user asks to "add CI for 앱인토스 deploy", "make a GitHub Action that runs `ait deploy`", "main 푸시하면 자동 배포되게 해줘", or similar.
- The repo is an Apps In Toss mini-app (verified in Step 1).

## When NOT to use

- The user wants to publish a release to end users → that's a console review action, not a CI action.
- The user wants per-PR preview deploys, rollback workflows, or secret rotation → those belong in a future `operations` plugin (don't reuse this skill).
- The repo is **not** an Apps In Toss project (no `@apps-in-toss/web-framework` in `package.json`) → stop and ask the user before continuing.

## Step 1 — Verify the project

Before writing anything:

1. Confirm a `package.json` exists at the working directory and contains either `@apps-in-toss/web-framework` (WebView flavor) or `@apps-in-toss/framework` (React Native flavor). Use `Read` to inspect.
2. Confirm `.github/workflows/deploy.yml` does **not** already exist (`Glob` or `test -e`). If it does, **abort** — show the existing file's first 30 lines and ask the user how to proceed (overwrite / merge / pick a different filename). Never silently overwrite.
3. Confirm the repo has a git remote. Read it with `git remote get-url origin` to extract the GitHub `<owner>/<repo>` for the secrets URL in Step 4.

If any check fails, stop and tell the user what's missing. Do not proceed to Step 2.

## Step 2 — Gather choices

Ask the user (reuse earlier conversation context if it already answers a bullet):

1. **Trigger** — pick one:
   - `main` — every push to `main` triggers a deployment (most common; queue fills up with each commit)
   - `tag` — only `v*` tags trigger a deployment (one deployment per release)
   - `manual` — only `workflow_dispatch` (no automatic runs)

   Default to `main` if the user has no preference. **Always also include `workflow_dispatch`** so the user can re-trigger from the Actions UI without an empty commit.

2. **Package manager** — detect from lockfile in this order:
   - `pnpm-lock.yaml` → `pnpm`
   - `yarn.lock` → `yarn`
   - `package-lock.json` → `npm`
   - none → ask. Default `npm`.

3. **Node version** — read the project's `.nvmrc`, `package.json#engines.node`, or `granite.config.ts` if any of those pin a version. Otherwise default to `20`. Never guess silently between `18`/`20`/`22`.

4. **Lint / typecheck gate** — ask only if the user hasn't said. Most teams run lint pre-push and skip in CI; the default here is **no gate**. If the user wants one, run `<pm> run lint` before the deploy step.

5. **Memo format** — what to put in `--memo`. Default to `main@${{ github.sha }}` (or `tag@${{ github.ref_name }}` for tag trigger). Keep it under 1000 characters (CLI hard limit).

Do not proceed to Step 3 until Trigger and Package manager are settled.

## Step 3 — Write `.github/workflows/deploy.yml`

Create the file with `Write`. Template (substitute `<...>` with the user's choices):

```yaml
name: Deploy to Apps in Toss

on:
  <trigger-block>      # see "Trigger blocks" below
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: <node-version>
          cache: <pm>

      - run: <pm-install>     # npm ci / pnpm install --frozen-lockfile / yarn install --frozen-lockfile

      - run: <pm-run> build   # npm run build / pnpm run build / yarn build

      - name: Deploy via ait
        env:
          AIT_API_KEY: ${{ secrets.AIT_API_KEY }}
        run: npx ait deploy --api-key "$AIT_API_KEY" --memo "<memo>"
```

### Trigger blocks

```yaml
# main
push:
  branches: [main]

# tag
push:
  tags: ['v*']

# manual (omit `push:` entirely; only workflow_dispatch remains)
```

### Important: why `--api-key` flag (and not env var)

The `ait` CLI (as of 2026-04, `@apps-in-toss/cli`) does **not** read an `AIT_API_KEY` environment variable. It only reads the API key from (1) the `--api-key` flag, (2) `~/.ait/credentials` (interactive setup file), or (3) an interactive prompt — and the prompt hangs CI. So in GitHub Actions you must pass it via `--api-key "$AIT_API_KEY"` while exposing the secret via the step's `env:` block.

If a future CLI version adds env var support, the flag form still works. Don't switch unless the user explicitly asks.

To re-verify on a future ait CLI version:

```sh
grep -E "process\.env\.[A-Z_]+" node_modules/@apps-in-toss/cli/dist/index.js | sort -u
```

If `AIT_API_KEY` (or similar) appears, you can drop the `--api-key` flag and rely on the env block alone.

## Step 4 — Tell the user how to finish setup

After writing the file, output (do **not** run these — the secret has to come from the user):

1. **Register the API key as a secret.** Direct link, with the actual repo path filled in:
   ```
   https://github.com/<owner>/<repo>/settings/secrets/actions
   ```
   New repository secret → Name: `AIT_API_KEY`, Secret: their Apps In Toss deploy API key.

   Where to find the key:
   - If they've ever run `ait deploy` locally and saved credentials, it's in `~/.ait/credentials` (JSON, keyed by profile/workspace).
   - Otherwise issue a new key from the 앱인토스 콘솔.

2. **Test the workflow** (recommend in this order):
   - Actions tab → "Deploy to Apps in Toss" → "Run workflow" (uses `workflow_dispatch`, doesn't require a commit).
   - Or: `git commit --allow-empty -m "ci: test deploy workflow" && git push`.

3. **Where to verify success**:
   - GitHub Actions log — the `Deploy via ait` step should print a deployment ID.
   - 앱인토스 콘솔 → 배포 관리 → the new deployment shows up with the `<memo>` you set, in review queue.

## Step 5 — Optional: commit on the user's behalf

Only if the user explicitly asks ("커밋도 해줘", "commit and push"). Otherwise leave the file staged-or-unstaged and let them review.

If asked, use a Conventional Commits-style message that matches the repo's existing log:

```sh
git add .github/workflows/deploy.yml
git commit -m "chore: add Apps In Toss auto-deploy workflow"
```

Do not push automatically unless the user also says "push".

## Guardrails

- **Never overwrite an existing `.github/workflows/deploy.yml`.** Abort if it exists. The user may already have a deploy pipeline you'd silently break.
- **Never write the API key into the workflow file.** Always reference `secrets.AIT_API_KEY`.
- **Never invent the API key value.** It must come from the user / 콘솔.
- **Never run `ait deploy` from this skill.** This skill only adds the workflow; the actual deploy happens in CI under the user's secrets, not here.
- **Never assume Node version or package manager.** Detect from the repo or ask.
- **Never claim the workflow "publishes" or "releases" the app.** It registers a deployment in the console queue; release is a separate console step.
- **Never skip `workflow_dispatch`.** It's the only way to test without a commit, and adds zero risk.

## Reference

- Apps In Toss release/deploy guide — https://developers-apps-in-toss.toss.im/development/deploy.html
- Apps In Toss console review process — https://developers-apps-in-toss.toss.im/intro/onboarding-process.html
- GitHub Actions context (`github.sha`, `github.ref_name`) — https://docs.github.com/en/actions/learn-github-actions/contexts
- `actions/setup-node` cache options — https://github.com/actions/setup-node#caching-global-packages-data

## Implementation notes (for the skill author)

- CLI source verification used while writing this skill: searched `node_modules/@apps-in-toss/cli/dist/index.js` for `process.env.*` and confirmed only `DEBUG`, `SENTRY_AUTH_TOKEN`, `SENTRY_ORG`, `SENTRY_PROJECT` are read. The deploy command's API key path goes through `TokenStorage.get(workspace)` → `~/.ait/credentials` → interactive prompt fallback.
- Re-run that verification when bumping support to a new `ait` CLI major version, and update both this skill and `--api-key` guidance if the CLI starts honoring an env var.
