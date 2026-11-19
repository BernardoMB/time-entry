# Project guide for Claude

This repo is a workspace for extracting the content of slide-based online courses into Markdown,
using the **Playwright MCP server** (configured in `.mcp.json`, tools named `mcp__playwright__*`).

## Browsing rules

- Always use the Playwright MCP tools (`browser_navigate`, `browser_snapshot`, `browser_click`,
  `browser_take_screenshot`, `browser_evaluate`, `browser_wait_for`, ...) for the course site.
  Do not use other browsers or web-fetch tools for it.
- The server runs with `--extension`: it controls the user's own, already-open Edge window (their
  real profile, cookies and logins). Only work in the tabs you open for the course; never close,
  read or change the user's other tabs. Do not open unrelated sites.
- If the tools fail to connect, ask the user to make sure Edge is open, the Playwright extension is
  installed and enabled, and to approve the connection in the extension.
- If the course needs a login, **stop and ask the user to sign in themselves in Edge**.
  Never type passwords yourself.
- Prefer `browser_snapshot` (accessibility tree) to read text; use screenshots only to understand
  layout or images.
- After each click to the next slide, wait for the content to change before reading it
  (`browser_wait_for` or a fresh snapshot). Animated slides may need a short wait.
- Courses are often inside an `<iframe>` (SCORM / Articulate / Rise / Captivate). If the snapshot
  shows little text, look inside the frame.

## Output rules

- Write results under `output/<course-slug>/`:
  - `output/<course-slug>/course.md` — the full course content
  - `output/<course-slug>/images/` — images saved from slides (if they can be downloaded/screenshotted)
- Append to `course.md` as you go (one slide at a time) so progress isn't lost if something fails.
- Transcribe slide text faithfully: keep headings, bullet hierarchy, numbered lists, bold/italic,
  tables, and quotes. Do not summarize unless asked.
- For each image: save it to `images/` (download via its `src` or take an element screenshot with
  `browser_take_screenshot`) and embed it with a relative link plus a short alt-text description.
  If it can't be saved, write `> [Image: <description>]` instead.
- Note interactive elements (tabs, accordions, click-to-reveal, quizzes): open each one and
  transcribe all revealed content.

## Markdown format

```markdown
# <Course title>

> Source: <url> · Extracted: <date>

## Table of contents
- [Slide 1 – <title>](#slide-1--title)
...

---

## Slide 1 – <slide title>

<text, bullets, etc.>

![<alt text>](images/slide-01-1.png)
```
