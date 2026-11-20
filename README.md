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
| `output/` | Generated courses go here (`output/<course-slug>/course.md` + `images/`). Tracked in git. |
| `timesheets/` | Weekly PMDB Blue time-entry CSVs generated from your calendar. Git-ignored (personal data). |

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

## Weekly timesheet (every Friday)

Generates the PMDB Blue CSV import file for the week from your Outlook calendar, using the rules in
[`output/engineering-timekeeping-2024/time-entry-course.md`](output/engineering-timekeeping-2024/time-entry-course.md).
Needs the **Microsoft 365** connector enabled in Claude and Edge open (for PMDB Blue). The agent
generates and validates the file only; you import it yourself in PMDB Blue
(**Software Development → CSV Import**). Timesheets auto-submit Monday 4:00 PM UTC.

Copy and paste this prompt on Friday after work:

````text
Generate my PMDB Blue timesheet CSV for this week (Monday–Friday of the week that ends today).

Source and rules
- Follow the time-entry rules in output/engineering-timekeeping-2024/time-entry-course.md.
- Read my calendar with the Microsoft 365 connector for Monday 00:00 to Saturday 00:00 local time.
  Read every result page; the connector returns times in UTC.

My schedule
- Time zone: Mexico City (UTC-6). Convert every event from UTC before allocating it.
- Workday: Monday–Friday, 8:00 AM–4:00 PM local = 8.0 hours per day, 40 hours per week.

Allocation decisions
- Count tentative/accepted events as attended. Skip cancelled and declined events, and events I'm
  marked "free" for. Clip events to 8:00–4:00; ignore anything entirely outside it.
- If two events overlap, count the overlapping time only once (the earlier-starting event keeps it).
- Classify each event per the course rules, using these PMDB projects:
  - Platform Daily, Platform Refinement, Platform Demo, Iteration Kickoff, AI Review / technical
    check-ins, and other meetings about the work under development → "Nebula - Application Development"
  - Any event with "Onboarding" in the title → "Nebula - Preliminary Project"
  - All remaining unscheduled 8:00–4:00 time → "Nebula - Preliminary Project",
    description exactly "Onboarding process" (no suffix such as "(unscheduled time)")
  - Manager 1:1s, department/company meetings, company trainings, recruiting → "General Admin"
  - Holidays, vacation, sick or out-of-office days → "PTO" (8.0 h for a full day off)
  - If an event doesn't clearly fit, or a new project name appears, ask me before writing the file.

CSV format (PMDB Blue "Timesheet Example")
- Columns exactly: User,Date,Project,Hours,Description
- User: my name exactly as PMDB Blue shows it in the top bar, written "Last, First".
- Date: M/D/YYYY. Hours: decimals in 0.1 increments. Description: the event title as-is, or
  "Onboarding process" for unscheduled time; never append notes or qualifiers in parentheses.
- Project: exact PMDB dropdown name without curly braces. Before writing, open PMDB Blue in Edge
  with Playwright (I'll sign in myself if needed), go to Software Development → Time Sheet and
  confirm the names exist in the page's hidden "Projects" list. Don't save or submit anything there.
- One row per project + description per day (merge same-titled blocks). Quote fields that contain
  commas. No extra columns.

Validation (must all pass before saving)
- Every workday is covered from 8:00 to 4:00 with no gaps or overlaps and totals exactly 8.0 h.
- The week totals 40.0 h. Every project name is in the PMDB list.

Output
- Save to timesheets/timesheet-<Monday YYYY-MM-DD>.csv (git-ignored; don't commit it).
- Show me a per-day timeline in local time, a per-day/per-project hours table, and any
  assumptions (tentative meetings, unclear classifications, skipped events).
- Do not import or submit the timesheet; I'll do that myself.
````

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
