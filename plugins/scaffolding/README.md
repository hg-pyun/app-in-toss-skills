# `scaffolding` plugin

Skills that **set up or extend the structure of an Apps In Toss mini-app project** — creating a new project, adding framework features, wiring up configuration files. If the user is asking "make me a new app" or "add X feature to my app", the skill probably belongs here.

Skills that are *not* about project structure (e.g. running the sandbox, looking up docs, building a release) belong in other plugins.

## Skills

| Skill | Command | Purpose |
| --- | --- | --- |
| [`create-mini-app`](./skills/create-mini-app/SKILL.md) | `/scaffolding:create-mini-app [app-name] [webview\|react-native]` | Scaffold a new mini-app using `create-ait-app` (WebView) or `create granite-app` (React Native), then wire up `granite.config.ts` and TDS. |

## Proposed future skills

These aren't shipped yet. Open a PR when you're ready to add one.

- `add-in-app-purchase` — wire in-app purchase APIs into an existing mini-app.
- `add-in-app-ad` — wire in-app ads APIs.
- `add-tds` — retrofit TDS into an existing project.
- `migrate-to-ait` — convert an existing React Native project to Apps In Toss framework.

## Authoring guidelines for this plugin

- **Check before you create.** Every skill that scaffolds files should verify the target path doesn't already exist and surface a clear error if it does.
- **Never hide interactivity.** Upstream CLIs (`create-ait-app`, `create granite-app`, `ait init`) are interactive. If the Bash tool cannot drive the prompts, hand control back to the user.
- **Keep framework choice explicit.** When WebView vs React Native matters, ask; never guess silently.
- **Treat `appName` as sacred.** It's the deeplink key (`intoss://{appName}`) and must match the 앱인토스 콘솔 registration.
- **Package manager detection lives in each skill for now.** If a third scaffolding skill needs the same logic, extract it into `references/package-manager.md` and reference it from every skill.
