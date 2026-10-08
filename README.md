# osheet

Interactive terminal tool for logging timesheet entries into an Excel file built from the `TimesheetUpload.xlsx` template, ready to upload.

## Install

```bash
ln -s "$PWD/osheet" ~/.local/bin/osheet
```

Requires Python 3.8+ (standard library only). On Windows, menus fall back to numbered input.

## Setup

The first time you run `osheet` it walks you through setup, then exits; run `osheet` again to log entries. Re-run setup any time with `osheet --setup` (or `-s`):

1. Asks for your `TimesheetUpload.xlsx` template and reads its columns, projects, entry types, billable flags, locations and sentiments.
2. Proposes default project, entry type, location, sentiment, file grouping (by week or by month) and the folder to save timesheets in (default `~/timesheets`); confirm them or change them.
3. Saves everything to the config file:

| OS | Config file |
|---|---|
| Linux | `$XDG_CONFIG_HOME/osheet/config.json` (default `~/.config/osheet/config.json`) |
| macOS | `~/Library/Application Support/osheet/config.json` |
| Windows | `%APPDATA%\osheet\config.json` |

A copy of the template is kept next to the config (`template.xlsx`), so the original can be moved or deleted afterwards. Re-run `osheet --setup` when the template changes.

## Usage

```bash
osheet
```

Prompts for:

1. **Day**: defaults to today; any day of the current week (Mon–Sun)
2. **Project**: defaults to your configured default
3. **Entry type** (category), filtered by project; Billable is set from the project
4. **Total time**: `1:20`, `1h20`, `80m`, `1.5` or `90`; rounded **up** to the next 15 minutes
5. **Location** (WorkedFrom), using the options from the template
6. **Description**: required, up to 255 characters

Answers fill in a panel at the top (`Day`, `Project`, `Entry type`, `Time`, `Location`, `Description`) that updates in place, with the current question shown below it; when you're done, only the completed panel and the saved file path stay in the terminal. Setup works the same way for the template and defaults. Text that doesn't fit the terminal's width wraps onto the next line (the panel, questions, menus, messages and the `--hours` table). Without a TTY (piped input) the questions and answers are printed as plain lines instead.

An unrecognised argument prints an error and exits with status 2 without logging anything.

In menus: ↑/↓ or `j`/`k` to move, Space, Enter or an option's number to select, type to filter. In checklists (setup's "asked every time" question), Space or a number toggles an option and Enter submits. Choose `0. ← Back` (or type `<` and Enter at a text question) to go back to the previous question.

### Logging several entries

```bash
osheet --multi    # or -m
```

Keeps osheet open: after each entry is saved it asks `Log another entry? (Y/n)`. Press `y` or Enter to log another, or `n` or `q` to finish. Each new entry starts with the previous entry's day, project, entry type and location selected. The time and description are blank again. Each entry is saved as soon as it's done, so Ctrl+C only drops the entry in progress. Without a TTY, type `y`, `n` or nothing (counts as yes); the end of input finishes.

### Skipping questions

Each question has an `ask_every_time` flag in the config file (all `true` by default). Set one to `false` to skip that question and use its default instead:

```json
"ask_every_time": {
  "day": false,
  "project": true,
  "category": false,
  "time": true,
  "location": false,
  "description": false
}
```

| Question | Default used when skipped |
|---|---|
| `day` | today |
| `project`, `category`, `location` | the value in `defaults` (still asked if it isn't valid, e.g. the entry type doesn't belong to the chosen project) |
| `time` | `defaults.time` (e.g. `"time": "1h"`); still asked if that isn't set |
| `description` | `defaults.description`; still asked if that isn't set or is over 255 characters |
Skipped answers still show in the panel, marked `(skipped)`, and Back passes over them.

## Output

Entries go into one Excel file per week or per month, depending on the grouping chosen during setup, based on the entry's date:

| Grouping | Example file |
|---|---|
| Week (Mon–Sun, `dd-dd_Month`) | `~/timesheets/2026_October/05-11_October.xlsx` |
| Month | `~/timesheets/2026_October/2026-10.xlsx` |

Set `OSHEET_FILE` to write a single run to one specific file instead. The file starts as a copy of your template, so the Lookup/Validation sheets, dropdowns and table are preserved; each entry becomes a new row in the `TimesheetEntry` table, with the date stored as a real Excel date.

Only the folder is configurable; files are named after their week or month. They are stored in a `<yyyy_Month>` folder per month (e.g. `2026_October`); a week that spans two months is filed under the month its Monday falls in. A new file is created from the template whenever an entry falls in a new week or month.

Upgrading from an earlier version: osheet asks once how to group files, then moves entries from the old CSV or single Excel file into the grouped files and keeps the old file as `.bak`. Files from earlier versions named `<name>_<period>.xlsx` (or with the old `dd-dd_MM` week label) are renamed to the current names on the next run.

### Viewing and editing entries

```bash
osheet --list     # or -l
```

Lists the entries from this week and the four before it, newest first. Pick one to see all its details in the panel, including the full description, billable flag, sentiment, ticket number and the file it's in. Then press:

| Key | Action |
|---|---|
| `e` | edit: asks every question again (none are skipped), starting from the entry's current values: menus have its answer selected, and the time and description are already typed in, ready to edit, so press Enter to keep a value. Back from the first question returns to the entry (at a text question, clear the line with Ctrl+U before typing `<`) |
| `d` | delete: asks `Delete this entry? (y/N)`; `y` deletes it and the rows below move up, `n` or Enter keeps it |
| `0` or `b` | back to the list |
| Enter or `q` | finish |

When editing, the day can be moved within the entry's own week; the row is updated in place, or moved to another file if the new day belongs to a different week or month. The sentiment and ticket number are kept as they were. Once an edit or delete is saved you return to the list, with a changed entry highlighted.

Without a TTY the details are printed as plain lines, followed by a question where `e` edits the entry and `d` deletes it; osheet exits after the change is saved.

### Hours summary

```bash
osheet --hours    # or -H
```

Asks whether to show today, this week (Mon–Sun) or this month, then shows a table of the time logged per project and entry type, with subtotals for projects that have more than one type and an overall total. Use `osheet --hours-day`, `osheet --hours-week` or `osheet --hours-month` to skip the question.

### Finding the file

```bash
osheet --export   # or -e
```

Prints the folder holding the current week's/month's timesheet as a clickable link and copies the path to the clipboard (`wl-copy`, `xclip` or `xsel` on Linux, `pbcopy` on macOS, `clip` on Windows).
