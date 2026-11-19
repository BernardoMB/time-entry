# Course → Markdown with Claude + Playwright MCP

A small workspace that lets a Claude agent (Claude Code / Claude desktop Code tab) drive a real
browser through the [Playwright MCP server](https://github.com/microsoft/playwright-mcp), walk
through a slide-based online course, and write every slide's content (text, bullets, images) into a
Markdown file.

## Project layout

| Path | Purpose |
|------|---------|
| `.mcp.json` | Registers the `playwright` MCP server for this project (`npx @playwright/mcp@0.0.86`, pinned; bump it to upgrade). |
| `.claude/settings.json` | Enables the project MCP server and pre-approves its tools and writes to `output/`. |
| `.claude/commands/extract-course.md` | `/extract-course <url>` slash command with the step-by-step extraction workflow. |
| `CLAUDE.md` | Rules the agent follows: how to browse, handle logins/iframes, and format the Markdown. |
| `output/` | Generated courses go here (`output/<course-slug>/course.md` + `images/`). Git-ignored. |

## Prerequisites

1. **Node.js 18+** (provides `npx`, which launches the MCP server):
   ```powershell
   winget install OpenJS.NodeJS.LTS
   ```
   Restart the terminal / Claude app afterwards so `npx` is on the PATH.
2. **Microsoft Edge with the Playwright extension.** The server runs with `--extension`, so instead
   of launching its own browser it attaches to **your already-open Edge window**, with your
   existing tabs, cookies and logins.
   1. In Edge, open `edge://extensions` and turn on **Allow extensions from other stores**.
   2. Install the **Playwright Extension** from the Chrome Web Store:
      <https://chromewebstore.google.com/detail/playwright-extension/mmlmfjhmonkocbjadbfplnigmagldckm>
   3. Pin it to the toolbar so you can see when it's connected.

   The tabs the agent controls are grouped in a tab group named **Claude**; your other tabs are
   left alone. The first time the agent connects, the extension asks you to approve the
   connection — click **Allow** (or choose which tab to share).

## Setup

1. Open this folder in Claude Code (or the Code tab of the Claude desktop app).
2. The first time, approve the `playwright` project MCP server when prompted.
3. Check it is connected: run `claude mcp list` in a terminal, or type `/mcp` in Claude Code.

## Usage

Use the slash command:

```
/extract-course https://example.com/my-course
```

or just ask in plain words:

> Use Playwright MCP Server to open the browser, navigate to https://example.com/my-course,
> navigate through the course and make a markdown file writing the content of all the slides.

Keep Edge open while it runs. The agent works in your Edge window, steps through each slide, and writes
`output/<course-slug>/course.md` (with images in `output/<course-slug>/images/`).

### Courses that need a login

Because the agent uses your own Edge session, just sign in to the course in Edge beforehand (or
when the agent pauses at a login page and asks you to), then tell it to continue.

## Configuration notes

- **Windows:** `npx` is wrapped with `cmd /c` in `.mcp.json`, which is required on native Windows.
  On macOS/Linux, change `"command"` to `"npx"` and drop the `"/c", "npx"` args.
- **Edge profile:** by default it connects to the last-used Edge profile that has the extension.
  To pick one, add `"--profile-dir-name", "Profile 1"` to the args.
- **Separate browser instead:** to have Playwright launch its own Edge window (with a saved profile
  instead of your session), replace `"--extension"` in `.mcp.json` with
  `"--user-data-dir", "./playwright-profile"`. Add `"--headless"` to hide that window.
- **Screenshots/traces** from the MCP server are written to `./.playwright-mcp` (git-ignored).
- Generated courses in `output/` are tracked in git.
