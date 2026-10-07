# osheet

Interactive terminal tool for logging timesheet entries to a CSV that matches the `TimesheetUpload.xlsx` template.

## Install

```bash
ln -s "$PWD/osheet" ~/.local/bin/osheet
```

Requires Python 3.8+ (standard library only). On Windows, menus fall back to numbered input.

## Setup

The first time you run `osheet` it walks you through setup (re-run any time with `osheet --setup`):

1. Asks for your `TimesheetUpload.xlsx` template and reads its columns, projects, entry types, billable flags, locations and sentiments.
2. Proposes default project, entry type, location, sentiment and output CSV; confirm them or change them.
3. Saves everything to the config file:

| OS | Config file |
|---|---|
| Linux | `$XDG_CONFIG_HOME/osheet/config.json` (default `~/.config/osheet/config.json`) |
| macOS | `~/Library/Application Support/osheet/config.json` |
| Windows | `%APPDATA%\osheet\config.json` |

The template contents are copied into the config, so the `.xlsx` can be moved or deleted afterwards. Re-run `osheet --setup` when the template changes.

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
6. **Description** (optional)

In menus: ↑/↓ to move, Enter to select, type to filter, digits to jump.

## Output

Rows are appended to the CSV chosen during setup (default `~/timesheets/TimesheetUpload.csv`; override per run with `OSHEET_FILE`), using the template's column order:

```
Date,Project,Category,Hours,Minutes,Billable,Description,TicketNumber,Sentiment,WorkedFrom
2026-10-06,Some Project,Meetings,2,15,Yes,standup,,Neutral,Home
```
