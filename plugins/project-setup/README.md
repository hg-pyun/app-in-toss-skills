# `project-setup` plugin

Skills for **one-time setup tasks** when starting (or onboarding to) an Apps In Toss mini-app project — creating the app itself, wiring local dev config, adding CI/CD workflows, retrofitting framework features. If a task is something you do **once at the beginning of a project** and then commit to the repo forever, it probably belongs here.

Tasks that are *not* one-time setup belong in other plugins:
- Looking up docs / API references → `knowledge-skills` plugin
- Running the sandbox / testing on device → `testing` plugin (future)
- Operating an existing release pipeline (rollbacks, secret rotation, preview deploys) → `operations` plugin (future)

## Skills

| Skill | Command | Purpose |
| --- | --- | --- |
| [`create-mini-app`](./skills/create-mini-app/SKILL.md) | `/project-setup:create-mini-app [app-name] [webview\|react-native]` | Scaffold a new mini-app using `create-ait-app` (WebView) or `create granite-app` (React Native), then wire up `granite.config.ts` and TDS. |
| [`add-deploy-action`](./skills/add-deploy-action/SKILL.md) | `/project-setup:add-deploy-action` | Add a GitHub Actions workflow that runs `ait deploy` on push to `main` (or a configured trigger), registering a deployment in the Apps In Toss console. |

## Proposed future skills

These aren't shipped yet. Open a PR when you're ready to add one.

- `add-in-app-purchase` — wire in-app purchase APIs into an existing mini-app.
- `add-in-app-ad` — wire in-app ads APIs.
- `add-tds` — retrofit TDS into an existing project.
- `migrate-to-ait` — convert an existing React Native project to Apps In Toss framework.
- `add-sentry` — wire Sentry monitoring into an existing project.

## Authoring guidelines for this plugin

- **One-time, not ongoing.** A skill belongs here only if its output is committed to the repo and not re-run during normal operation. Recurring/ops tasks go elsewhere.
- **Check before you create.** Every skill that writes files should verify the target path doesn't already exist and surface a clear error if it does. Never silently overwrite.
- **Never hide interactivity.** Upstream CLIs (`create-ait-app`, `create granite-app`, `ait init`) are interactive. If the Bash tool cannot drive the prompts, hand control back to the user.
- **Keep framework choice explicit.** When WebView vs React Native matters, ask; never guess silently.
- **Treat `appName` as sacred.** It's the deeplink key (`intoss://{appName}`) and must match the 앱인토스 콘솔 registration.
- **Package manager detection lives in each skill for now.** If a third skill needs the same logic, extract it into `references/package-manager.md` and reference it from every skill.
