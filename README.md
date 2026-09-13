# camu.in Attendance Sync

Turns "open camu.in, click through every class, note the present/total, copy it
into a spreadsheet" into something you just ask Claude to do — and keeps a clean
weekly grid (Section x Day = present/total) in a local Excel file for you.

**This is not a standalone program.** It's a [Claude Code Skill](https://docs.claude.com/en/docs/claude-code) —
a set of instructions your own Claude follows, using a connected browser-automation
tool (e.g. the Claude-in-Chrome extension) to read your **already logged-in**
camu.in session. It never sees, stores, or enters your camu.in password.

## Requirements

- **A Claude subscription with Claude Code access** — Pro, Max, Team, Enterprise,
  or pay-as-you-go API/console access. The free Claude.ai tier does not currently
  include Claude Code, so this isn't usable for free. (Check
  [Anthropic's current plans](https://www.anthropic.com/pricing) — this changes
  over time.)
- **A browser-automation tool connected to your real browser** (e.g. the
  Claude-in-Chrome extension), signed into the same Claude account as your Claude
  Code session.
- **Python 3** with `openpyxl` installed (`pip install openpyxl`) for the
  spreadsheet-building script.
- You, logged into your institution's camu.in in that browser, as normal.

## How it works

1. You ask Claude something like *"sync today's camu attendance"*.
2. Claude reads camu.in's Schedule page for the day (via the browser tool) —
   every class you taught, its section, and the present/total count from each
   class's Attendance page.
3. Claude writes those numbers into `scripts/build_attendance_grid.py`'s JSON
   input format and runs it, which creates/updates a `.xlsx` workbook with one
   sheet per group (e.g. per semester) — rows are sections, columns are weekdays.
4. You open the workbook whenever you want to check it or copy it elsewhere.

Nothing runs unattended — you trigger each sync yourself, while logged into
camu.in in your own browser. There's no server, no stored credentials, and no
scheduled automatic login.

## Setup

1. Install Python and `openpyxl`:
   ```
   pip install openpyxl
   ```
2. Copy `skills/camu-attendance-sync/` into wherever your Claude Code setup loads
   skills from (see your Claude Code docs for the skills directory location).
3. Make sure a browser-automation tool (e.g. Claude-in-Chrome) is enabled and
   connected in your Claude Code session.
4. Log into your institution's camu.in in that same browser.
5. Ask Claude to sync your attendance. On the first run, Claude will ask how you
   want your classes grouped into sheets (by semester is the default assumption)
   and where to save the workbook.

## Example output format

See [`examples/sample_config.json`](examples/sample_config.json) for the JSON
shape the skill produces, and run it yourself to see the resulting workbook:

```
python scripts/build_attendance_grid.py --config examples/sample_config.json
```

This writes `./output/attendance.xlsx` with two sheets (`Group_A_Wk1`,
`Group_B_Wk1`), each a Section x Day grid.

Cell values:
| Value | Meaning |
|---|---|
| `45/60` | confirmed present/total count |
| `NA` | confirmed — no class scheduled that section that day |
| `?` (shaded) | a class was held but the count hasn't been looked up yet |
| *(blank)* | that day hasn't been checked at all |

## Adapting this to your institution

camu.in is used, white-labeled, by many different colleges. This skill was built
and tested against one institution's instance — page labels, section-naming
conventions, and minor DOM structure may differ elsewhere. See the "Adapting this
skill to a different institution" section in
[`skills/camu-attendance-sync/SKILL.md`](skills/camu-attendance-sync/SKILL.md)
before relying on it somewhere new. In short: walk through it once with your
Claude watching, and correct anything that doesn't match your institution's
camu.in before trusting it to run unsupervised.

## Privacy & security

- Your camu.in password is never entered, stored, logged, or transmitted by this
  skill. It only reads pages you're already authenticated to view yourself.
- The resulting Excel file lives on your own machine. Nothing is uploaded
  anywhere by this tool.
- Automated reading of an institutional system may or may not be covered by your
  institution's or camu.in's terms of use — this tool automates something you're
  already allowed to view manually, but you're responsible for checking your own
  institution's policies before relying on it regularly.

## Security scanning (Gitleaks)

Since this project is specifically about *never* committing credentials, it's
scanned with [Gitleaks](https://github.com/gitleaks/gitleaks) on every push/PR
via [`.github/workflows/gitleaks.yml`](.github/workflows/gitleaks.yml), using
the rules in [`.gitleaks.toml`](.gitleaks.toml).

To also catch things locally before you commit:

```
pip install pre-commit
pre-commit install
```

This uses [`.pre-commit-config.yaml`](.pre-commit-config.yaml) to run Gitleaks
on every `git commit`. You can also run it manually at any time:

```
pre-commit run gitleaks --all-files
```

## License

MIT — see [LICENSE](LICENSE).
