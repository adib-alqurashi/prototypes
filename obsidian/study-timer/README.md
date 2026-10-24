# Study Timer & Calendar

An Obsidian plugin: one start/stop study timer, a stats card (today, streak, this week, total, best day), and a month calendar of your dated tasks that you can add to from the calendar itself.

No build step, no dependencies. Three files: `main.js`, `manifest.json`, `styles.css`.

## Install

1. In your vault, create the folder `.obsidian/plugins/study-timer/`.
2. Copy `main.js`, `manifest.json` and `styles.css` into it.
3. Obsidian → Settings → Community plugins → turn off Restricted mode if asked → enable **Study Timer & Calendar**.

To update, copy the three files again and reload Obsidian (Ctrl+R).

## Use

**Timer**

- Click the timer icon in the left ribbon, or run the command **Study Timer: Start / stop** (give it a hotkey).
- The status bar shows the running clock. Stop it the same way.
- **Study Timer: Discard the running session** throws away the current run without saving it.
- Sessions that cross midnight are split correctly between the two days.

**Stats card**: put this in any note.

````markdown
```study-timer
```
````

It shows today's minutes, the current streak (days with at least one minute), this week, the total, and the best day.

**Calendar**: a month view of every task with a 📅 or ⏳ date in your vault, in the format the Tasks plugin uses (`- [ ] text 📅 2026-10-24`).

````markdown
```planner-calendar
tasks: 1 Organize/Tasks.md
name: Term 261
start: 2026-08-24
end: 2026-12-17
```
````

| Option | What it does |
| --- | --- |
| `tasks` | The note that new tasks are appended to when you add one from the calendar (path from the vault root). |
| `name`, `start`, `end` | Optional. Shows a progress line: "Term 261 · week 6 of 17 · 54 days left". |

Click a day to select it, type a task (optional time, optional deadline flag) and press Enter. The line is written as `- [ ] 14:00 text #deadline 📅 2026-10-24`, so the Tasks plugin sees it too.

## Where the data lives

Minutes per day and the running session are in `.obsidian/plugins/study-timer/data.json`. Tasks are never stored by the plugin; they are read from, and written to, your notes.

If you sync your vault with git, add `data.json` to `.gitignore` so each device keeps its own history.

## How it works

`main.js` is plain JavaScript using only the Obsidian API (`Plugin`, `Notice`, `MarkdownRenderChild`):

- `addSpan` adds a `[start, end)` interval to a `{day: minutes}` map, splitting at midnight.
- `computeStats` derives streak, week and best day from that map. It is a pure function, so it is the easiest thing to test.
- `CalendarView` renders the month grid, reads tasks from the vault's Markdown files, and appends new ones.
- Two code block processors register the `study-timer` and `planner-calendar` blocks.

## Ideas for improving it

Pull requests are welcome. Some directions, roughly in order of effort:

- **Pomodoro mode**: a fixed-length session with a break reminder, using the existing timer.
- **Per-subject tracking**: tag a session with a course or project and break the stats down by tag.
- **Export**: a command that writes the day map to CSV for a spreadsheet.
- **Weekly chart**: a small bar chart in the stats card.
- **Tests**: `computeStats` and `addSpan` have no side effects; a `node --test` file for them would make changes safer.
- **TypeScript**: the plugin is a single JS file for simplicity. Moving to the official sample-plugin TypeScript setup would help once it grows.

To develop: edit `main.js` in place inside a test vault's `.obsidian/plugins/study-timer/` and press Ctrl+R in Obsidian to reload.

## What I learned

Code block processors are the simplest way to put live UI inside a note, and `MarkdownRenderChild` handles the lifecycle for you. The hardest part was not the timer but the dates: split sessions at midnight once, in one place, and everything downstream stays simple.
