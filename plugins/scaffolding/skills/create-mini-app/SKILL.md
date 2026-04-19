---
name: create-mini-app
description: Scaffold a new Apps In Toss (앱인토스) mini-app. Use when the user asks to create / start / scaffold a new Apps In Toss mini-app, 앱인토스 미니앱, AIT app, or Granite app — either the WebView flavor (create-ait-app) or the React Native flavor (create granite-app).
argument-hint: [app-name] [webview|react-native]
allowed-tools: Bash(npx *) Bash(npm *) Bash(pnpm *) Bash(yarn *) Bash(ls *) Bash(test *) Bash(pwd) Bash(ipconfig *) Read Write Edit Glob Grep
---

# Create an Apps In Toss mini-app

Help the user scaffold a brand-new Apps In Toss (앱인토스) mini-app from scratch.
Two frameworks are supported — pick exactly one, then drive the official CLI.

| Flavor | When to use | Scaffolder |
| --- | --- | --- |
| **WebView** | User has (or wants) a web stack — Vite, Next.js, plain HTML/CSS. Fastest to start. | `create-ait-app` |
| **React Native (Granite)** | User wants native-feel UI, file-based routing, or React Native DX. Required for some 앱인토스 features. | `create granite-app` + `ait init` |

Arguments (optional, order-independent):
- `$0` — app name (kebab-case, e.g. `my-mini-app`)
- `$1` — `webview` or `react-native`

If either is missing, ask the user. Never guess.

---

## Step 1 — Gather requirements

Before running anything, confirm all of the following with the user (reuse earlier conversation context if it already answers a bullet):

1. **Framework** — WebView or React Native?
   Heuristics for asking:
   - User says "RN", "React Native", "granite", "native UI" → React Native.
   - User says "웹뷰", "WebView", already has a Vite/Next project, or is unsure → WebView.
   - Otherwise ask a single direct question with both options.
2. **App name** — kebab-case only (`^[a-z][a-z0-9-]*$`).
   Warn: this value becomes the 딥링크 key `intoss://{appName}` and must match what is registered in the 앱인토스 콘솔 later.
3. **Target directory** — verify the current working directory with `pwd`. The scaffolder will create a new folder named `<app-name>` inside it. Abort if `<app-name>` already exists (use `Glob` or `test -e`).

Do not proceed to Step 2 until these three are settled.

## Step 2 — Pick the package manager

Pick in this order, then tell the user which one you chose:

1. Explicit request from the user.
2. Lockfile in the current directory — `pnpm-lock.yaml` → pnpm, `yarn.lock` → yarn, `package-lock.json` → npm.
3. `packageManager` field in a nearby `package.json`.
4. Default to `npm`.

Never silently switch between `npm` / `pnpm` / `yarn` mid-flow.

## Step 3 — Run the scaffolder

Both CLIs are **interactive**. The Bash tool may not forward stdin in every environment. If a prompt-driven command hangs, stop it, tell the user to run the command themselves in a real terminal, and resume from Step 4 once they report completion.

### A. WebView (`create-ait-app`)

Run from the user's current directory:

```sh
# pick the line matching the chosen package manager
npx create-ait-app <app-name>
npm create ait-app <app-name>
pnpm create ait-app <app-name>
yarn create ait-app <app-name>
```

Before executing, warn the user about the prompts they will see:
- **TDS (Toss Design System)** — required for all non-game mini-apps. Say yes unless they are building a game.
- **AI Skills** — adds Cursor / Claude Code / Codex skill files. Recommend **yes** when the user is developing with Claude Code.
- **Example code** — optional starter code for in-app purchase, in-app ads, etc. Pick whatever the user asked for.

After the CLI finishes, run dependency install inside the new directory:

```sh
cd <app-name>
<pm> install   # npm install / pnpm install / yarn install
```

### B. React Native (`create granite-app` + `ait init`)

The Granite CLI is a two-stage setup. Run both stages.

**Stage 1 — scaffold the Granite project**

```sh
npm create granite-app
# or: pnpm create granite-app / yarn create granite-app
```

Prompts:
- App name — enter the kebab-case name from Step 1.
- Tooling — `prettier + eslint` or `biome`. Default to whichever the user prefers; `biome` if unsure.

Then install dependencies:

```sh
cd <app-name>
<pm> install
```

**Stage 2 — add the Apps In Toss framework**

```sh
<pm> add @apps-in-toss/framework
# then:
npx ait init    # or: pnpm ait init / yarn ait init
```

During `ait init`:
- Pick **framework**: `react-native`.
- Enter **appName** (same kebab-case value).

**Stage 3 — install TDS React Native (required for non-game apps)**

Pick the package version by the installed `@apps-in-toss/framework` version (check the new project's `package.json` with `Read`):

| `@apps-in-toss/framework` | Package to install |
| --- | --- |
| `>= 1.0.0` | `@toss/tds-react-native` |
| `< 1.0.0` | `@toss-design-system/react-native` |

```sh
<pm> add @toss/tds-react-native        # for framework >= 1.0.0
<pm> add @toss-design-system/react-native  # for framework < 1.0.0
```

## Step 4 — Configure `granite.config.ts`

Both flavors produce a `granite.config.ts` at the project root. Read it first, then use `Edit` to fill in the user's values:

- `appName` — **must match** the name registered in the 앱인토스 콘솔. Do not change this casually later.
- `brand.displayName` — Korean-friendly display name shown in the nav bar.
- `brand.primaryColor` — hex color (e.g. `#3182F6`).
- `brand.icon` — hosted icon URL. Empty string `''` is acceptable during local testing.
- WebView only: if the user wants to test on a real device, set `web.host` to the dev machine's LAN IP and add `--host` to the Vite dev command. Suggest `ipconfig getifaddr en0` on macOS to find the IP.

Do not invent values — ask the user if a field is unknown.

## Step 5 — Hand off with next steps

Tell the user (do **not** run these automatically):

1. Start the dev server:
   ```sh
   <pm> run dev
   ```
2. Install the 앱인토스 sandbox app on iOS / Android (see https://developers-apps-in-toss.toss.im/development/client).
3. Open the mini-app via the scheme `intoss://<appName>` inside the sandbox app.
4. React Native on Android: if running on a real device/emulator, run
   ```sh
   adb reverse tcp:8081 tcp:8081
   adb reverse tcp:5173 tcp:5173
   ```
5. For Claude Code + MCP integration, suggest:
   ```sh
   claude mcp add --transport stdio apps-in-toss ax mcp start
   ```
   (after installing `ax` via Homebrew/Scoop — see https://developers-apps-in-toss.toss.im/development/llms.html).

## Guardrails

- **Never pick the framework silently.** Always get an explicit answer.
- **Never overwrite an existing directory.** Abort if `<app-name>/` already exists.
- **Never skip the interactive prompts** by piping in `yes` or similar — the CLIs change behavior based on the answers.
- **Never edit 앱인토스 콘솔-registered `appName`** without telling the user it will break deeplinks.
- **Never invent commands** beyond the ones listed here. If the user needs something outside this flow, stop and ask.

## Reference

- WebView tutorial — https://developers-apps-in-toss.toss.im/tutorials/webview.html
- React Native tutorial — https://developers-apps-in-toss.toss.im/tutorials/react-native.html
- AI + docs + MCP guide — https://developers-apps-in-toss.toss.im/development/llms.html
- Config reference — https://developers-apps-in-toss.toss.im/bedrock/reference/framework/UI/Config.html
