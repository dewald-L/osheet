# osheet

Single-file Python CLI (`osheet`, no extension), stdlib only, symlinked to `~/.local/bin/osheet`.
User-facing behaviour is in README.md; update it when the UX or flags change.

## Constraints
- Stdlib only, one file. Match the existing style: `# --- section ---` comments, small helpers,
  ANSI constants (BOLD/DIM/CYAN/GREEN/RED/MAGENTA/RESET).
- No curses or alternate screen: the final panel must stay in the scrollback.
- Don't change the config format or the Excel output (rows appended into the template's table).

## Excel I/O
No openpyxl: the xlsx is read and written as a zip of XML (`read_xlsx`, `append_rows`). Appending
updates the sheet rows, `sharedStrings.xml`, the `<dimension>` ref, the data-validation `sqref`s and
the table's `ref`; dates are serial numbers with the template's date style. Keep these consistent or
Excel reports the file as corrupt. `main()` runs `migrate_csv`/`migrate_ungrouped` on every start.

## Interactive UI (`Form`)
- `Form(fields)` draws a boxed key/value panel and redraws it in place with plain ANSI (cursor up,
  `\033[2K`, `\033[J`). `self.drawn` = lines from the top of the panel down to the cursor; every
  line goes through `fit()` so the count stays correct. Account for any new line you print.
- Flows are lists of step functions run by `form.run(steps)`; `Back` (menu `0`, or `<` at a text
  question) re-runs the previous step. Steps read earlier answers from a state dict so going back
  pre-selects them. A step may return the index of the next step (setup loops back to "confirm").
- Without a TTY (`form.live` False: pipes, Windows), questions print as plain lines and there is no Back.
  This output must stay byte-identical; check it by diffing piped runs against `git show HEAD:osheet`.

## Testing
No test suite. Quick check: `python3 -m py_compile osheet`.
Never run against the real `~/.config/osheet` or `~/timesheets`. Use a throwaway HOME:
```bash
H=$(mktemp -d); mkdir -p $H/.config/osheet
cp ~/.config/osheet/{config.json,template.xlsx} $H/.config/osheet/
printf '\n7\n\nabc\n2h05\n2\nDesc\n' | env -u XDG_CONFIG_HOME HOME=$H ./osheet   # plain fallback
```
(`XDG_CONFIG_HOME` must be unset or it overrides HOME.) For the live UI, drive it with `pty.fork()`,
set the size with `TIOCSWINSZ`, send keys (`b'\x1b[A'`, `b'\r'`, `b'2h05\r'`), and replay the output
through a small ANSI emulator to check the final screen. Read entries back with `read_entries(path)`;
set `OSHEET_FILE=$H/out.xlsx` to pin the output to one known file.

## Git
Commits are GPG-signed (per git config); push to `origin main`.
