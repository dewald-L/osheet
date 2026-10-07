# osheet

Interactive terminal tool for logging timesheet entries into an Excel file built from the `TimesheetUpload.xlsx` template, ready to upload.

## Install

```bash
ln -s "$PWD/osheet" ~/.local/bin/osheet
```

Requires Python 3.8+ (standard library only). On Windows, menus fall back to numbered input.

## Setup

The first time you run `osheet` it walks you through setup, then exits; run `osheet` again to log entries. Re-run setup any time with `osheet --setup`:

1. Asks for your `TimesheetUpload.xlsx` template and reads its columns, projects, entry types, billable flags, locations and sentiments.
2. Proposes default project, entry type, location, sentiment, file grouping (by week or by month) and output location; confirm them or change them.
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
6. **Description** (optional)

In menus: ↑/↓ to move, Enter to select, type to filter, digits to jump.

## Output

Entries go into one Excel file per week or per month, depending on the grouping chosen during setup, based on the entry's date:

| Grouping | Example file |
|---|---|
| Week (Mon–Sun, `dd-dd_MM`) | `~/timesheets/2026 10/TimesheetUpload_05-11_10.xlsx` |
| Month | `~/timesheets/2026 10/TimesheetUpload_2026-10.xlsx` |

Set `OSHEET_FILE` to write a single run to one specific file instead. The file starts as a copy of your template, so the Lookup/Validation sheets, dropdowns and table are preserved; each entry becomes a new row in the `TimesheetEntry` table, with the date stored as a real Excel date.

Files are stored in a `<yyyy MM>` folder per month; a week that spans two months is filed under the month its Monday falls in. A new file is created from the template whenever an entry falls in a new week or month.

Upgrading from an earlier version: osheet asks once how to group files, then moves entries from the old CSV or single Excel file into the grouped files and keeps the old file as `.bak`.

### Finding the file

```bash
osheet --export
```

Prints the folder holding the current week's/month's timesheet as a clickable link and copies the path to the clipboard (`wl-copy`, `xclip` or `xsel` on Linux, `pbcopy` on macOS, `clip` on Windows).
