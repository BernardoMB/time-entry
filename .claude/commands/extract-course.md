---
description: Open a slide-based course with Playwright MCP and transcribe every slide into Markdown
argument-hint: <course-url> [extra instructions]
---

Use the Playwright MCP server to extract the full content of the course at: $ARGUMENTS

Follow the browsing and output rules in CLAUDE.md. Steps:

1. Navigate to the URL with `browser_navigate`. If a login page appears, stop and ask me to sign in
   in the browser window, then continue when I say so.
2. Take a snapshot, identify the course title, the slide/navigation structure (Next button,
   menu/outline, progress indicator) and the total number of slides if shown. Check for iframes.
3. Create `output/<course-slug>/course.md` with the title, source URL and today's date.
4. For each slide, in order:
   - Read all text (headings, paragraphs, bullets, tables, captions, notes).
   - Open any tabs/accordions/click-to-reveal elements and capture their text.
   - Save images into `output/<course-slug>/images/` (named `slide-NN-M.ext`) and embed them.
   - Append the slide to `course.md` as `## Slide N – <title>`.
   - Move to the next slide and wait for it to load. Detect the end (no Next button, last
     slide number reached, or content repeats).
5. When done, add a table of contents at the top, re-read `course.md` for completeness and tell me
   the number of slides captured and anything that could not be extracted.
