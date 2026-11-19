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
2. **A browser.** The config uses **Microsoft Edge** (`--browser msedge`), which ships with Windows.
   To use another browser, change `--browser` in `.mcp.json` to `chrome`, `firefox`, `webkit` or
   `chromium` (the last three need `npx playwright install <browser>` first).

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

The agent opens a visible Edge window, steps through each slide, and writes
`output/<course-slug>/course.md` (with images in `output/<course-slug>/images/`).

### Courses that need a login

The browser uses a persistent profile in `./playwright-profile`, so you only need to sign in once.
When the agent reaches a login page it will pause and ask you to sign in yourself in the opened
browser window, then continue.

## Configuration notes

- **Windows:** `npx` is wrapped with `cmd /c` in `.mcp.json`, which is required on native Windows.
  On macOS/Linux, change `"command"` to `"npx"` and drop the `"/c", "npx"` args.
- **Headless mode:** add `"--headless"` to the args in `.mcp.json` if you don't want to see the browser.
- **Screenshots/traces** from the MCP server are written to `./.playwright-mcp` (git-ignored).
- To commit generated courses, remove the `output/*` lines from `.gitignore`.
