# Changes to thehive-cli (formerly hiveCLI.py, hiveGetTicket.py)

## Version 1.1.0 (2026-06-09)

Adopts semantic versioning (previous releases used build-date strings). This release
consolidates everything since `2026-03-01-report`, including the JSON output work and a
large set of new write/edit capabilities.

### Renamed
- **Renamed from `hiveCLI.py` to `thehive-cli`** — lowercase, no `.py`, and the `thehive-` prefix avoids clashing with Apache Hive. Invoke as `./thehive-cli`; regenerate shell completion to pick up the new command name.

### New Features
- **JSON Output (`--json`)**: Machine-readable output for every mode, for scripting and `jq` pipelines
  - Suppresses decorative text and the clipboard write
  - Works with `--list-cases`, `--list-alerts`, `-t/--ticket`, `--live-feed`, and all the modes below
  - Example: `./thehive-cli --list-cases --status new --json | jq '.cases[].title'`

- **List Observables (`--list-observables`)**: Full observable detail for a case (requires `--ticket`)
  - Shows dataType, data, IOC/sighted flags, TLP/PAP, tags, message, timestamps; file observables show attachment metadata
  - Example: `./thehive-cli -t 666 --list-observables --json | jq '.observables[] | select(.ioc)'`

- **Add Observables (`--add-observable`, `--add-ioc`)**: Create observables in a case (requires `--ticket`); `--add-ioc` is a shortcut that flags everything it adds as an IOC
  - Single value, several at once (comma-separated or `--obs-data - < file`), or a file observable (`--obs-file`)
  - Flags: `--obs-type`, `--obs-data`, `--obs-file`, `--obs-ioc`, `--obs-sighted`, `--obs-tags`, `--obs-tlp`, `--obs-message`
  - `--interactive` opens a line-based IOC table in `$EDITOR` (`<type> <data> [ioc] [sighted] [tags=a,b]`)
  - Example: `./thehive-cli -t 666 --add-observable --obs-type ip --obs-data - < iocs.txt`

- **Case Pages (`--list-pages`, `--add-page`, `--update-page`)**: Manage case knowledge pages (requires `--ticket`)
  - Create with `--page-title`/`--page-content`/`--page-category` (`-` reads content from stdin), or `--add-page --interactive` to compose a new page in `$EDITOR`
  - `--update-page PAGE_ID --interactive` edits the page in `$EDITOR`, applying only changed fields
  - Example: `./thehive-cli -t 666 --add-page --page-title "Runbook" --page-content - --page-category Notes < runbook.md`

- **Create Case (`--create-case`)**: Open a new case from the CLI (does not require `--ticket`)
  - `--interactive` fills in title/description in `$EDITOR`, or use `--case-title`/`--case-description`
  - Optional `--case-template NAME` applies a case template (server-side expansion); `--list-case-templates` lists available templates
  - Example: `./thehive-cli --create-case --interactive --case-template "Phishing"`

- **Update Case (`--update-case`, alias `--case-update`)**: Edit an existing case's title/description (requires `--ticket`)
  - `--interactive` edits in `$EDITOR` (applies only changed fields); or non-interactive `--case-title`/`--case-description`
  - `-t NUM --interactive` is a shorthand for the interactive form
  - Example: `./thehive-cli -t 666 --update-case --interactive`

- **Timeline Custom Events (`--list-events`, `--add-event`, `--update-event`, `--delete-event`)**: Manage custom events on a case timeline (requires `--ticket`)
  - The auto-derived timeline stays read-only (see `--history`); these manage analyst-authored events
  - `--event-title`/`--event-date`/`--event-end`/`--event-description`; dates accept `YYYY-MM-DD HH:MM[:SS]` (local) or epoch-ms (defaults to now on add)
  - `--add-event`/`--update-event` support `--interactive` ($EDITOR with a Title/Date/EndDate header + description body)
  - Example: `./thehive-cli -t 666 --add-event --event-title "Containment started" --event-date "2026-06-09 14:30"`

- **Tasks (`--list-tasks`, `--add-task`, `--add-task-log`)**: Manage case tasks and work logs (requires `--ticket`)
  - `--add-task` takes `--task-title` (+ `--task-description`/`--task-group`) or `--interactive` ($EDITOR with a Title/Group header + description body)
  - `--add-task-log TASK_ID` adds a work-log entry; message via `--task-log-message`, `-` for stdin, or `--interactive`/`$EDITOR`
  - Example: `./thehive-cli -t 666 --add-task --task-title "Collect logs" --task-group forensics`

- **Attachment download (`--download-attachment`)**: List a case's attachments + IDs (bare) or download one by ID (requires `--ticket`); save path via `--output-file`

- **Case export (`--export-case`, `--export-misp`)**: Export a case to an encrypted `.thar` archive for re-import elsewhere (`--export-password` or an interactive prompt), or push it to a configured MISP server with `--export-misp MISP_SERVER` (both require `--ticket`)

- **`--whoami`**: Show the current API user (login/organisation/profile) — a quick credential and connectivity check

- **Interactive `$EDITOR` editing**: A shared front-matter editor underpins interactive case, page, and timeline-event edits
  - Resolves `$EDITOR` → `$VISUAL` → `vi`; aborts cleanly on an unchanged buffer or a removed `---` separator
  - Sends only the fields that actually changed
  - `--add-comment` with no text (or `--add-comment "draft" --interactive`) composes the comment in `$EDITOR` as plain markdown (headings preserved); example: `./thehive-cli -t 666 --add-comment`
  - **Lost-update guard**: TheHive has no optimistic-concurrency control, so before a case/page/event interactive update is applied, the entity is re-fetched and the edit is aborted if a field you changed was modified by someone else while you were in `$EDITOR` (your edit is preserved to a temp file so nothing is lost)

- **Attachment file picker**: Choose the file to attach interactively
  - Bare `--add-attachment` picks from the current directory; `--add-attachment <dir>` picks from inside that directory
  - Uses `fzf` if installed, otherwise a TAB-completing prompt (press Enter to browse a numbered menu)
  - Sorted newest-first by default; `--attachment-sort name|mtime|size` and `--attachment-sort-reverse` to change
  - Example: `./thehive-cli -t 666 --add-attachment ~/Downloads`

- **Rename attachments on upload (`--attachment-name`)**: Store an attachment under a different name than the file on disk
  - The interactive picker also prompts "Upload as:" (pre-filled, editable); composes with `--timestamp-attachment`
  - Example: `./thehive-cli -t 666 --add-attachment screenshot.png --attachment-name case666_evidence.png`

- **Filename tab-completion**: `shtab` completion now completes file paths for `--add-attachment`, `--obs-file`, and `--output-file` (regenerate your completion script to pick this up)

### Aliases & Naming
- **`--list-cases`** is now the canonical name for listing cases; **`--list-all`** is kept as a back-compat alias
- **`--case`** added as an alias for `-t`/`--ticket`
- **`--store-credentials`**, **`--set-api-key`**, and **`--login`** added as aliases for `--store-key`

### Fixes & Maintenance
- **Fixed `--version`/`-v`**: now prints `thehive-cli <version>` and exits cleanly (previously printed the wrong program name, "hive5 getTicket.py"); switched to argparse's built-in version action
- Removed ~370 lines of dead code: an unused legacy comment-display function, a broken `search()` referencing a nonexistent API call, and commented-out getopt/scratch blocks
- Replaced deprecated `datetime.utcnow()` with timezone-aware `datetime.now(timezone.utc)`
- Normalized indentation to tabs (`write_to_clipboard` had been space-indented)

## Version 2026-03-01-report

### New Features
- **List Report Templates**: Added `--list-report-templates` option to display all available case report templates
  - Shows template ID, name, and format
  - Helps identify which templates are available before rendering reports
  - Example: `./hiveCLI.py --list-report-templates`
  - Automatically tries multiple API endpoints for compatibility

- **Render Case Reports**: Added `--render-report` option to generate case reports using templates
  - Requires `--ticket` to specify which case
  - Accepts template ID or name as argument
  - Optional `--output-file` parameter to save report to a file
  - If no output file specified, prints report to stdout
  - Automatically tries multiple API endpoint variations for compatibility
  - Examples:
    - `./hiveCLI.py -t 666 --render-report template_name`
    - `./hiveCLI.py -t 666 --render-report template_id --output-file report.html`
  - Validates template existence and permissions
  - Supports HTML and other report formats supported by TheHive templates

## Version 2026-03-01-shtab

### Program Renamed
- **Renamed from hiveGetTicket.py to hiveCLI.py** to better reflect the comprehensive CLI capabilities

### New Features
- **Shell Tab Completion**: Added shtab support for command-line tab completion
  - Supports Bash, Zsh, and Fish shells
  - Auto-completes all options and flags
  - Install with: `pip install shtab`
  - Generate completion: `./hiveCLI.py --print-completion bash`
  - See SHELL_COMPLETION.md for detailed installation instructions
  - Gracefully degrades if shtab is not installed (optional dependency)

## Version 2026-03-01

### New Features
- **Case History**: Added `--history` option to display activity timeline/audit logs for a specific case
  - Shows all activities with timestamps, users, and action details
  - All timestamp fields (_createdAt, startDate, endDate, etc.) are converted to readable format
  - Requires `--ticket` to be specified
  - Example: `./hiveGetTicket.py -t 666 --history`

- **Case Comments**: Added `--comments` option to display all comments for a specific case
  - Shows comment date, creator, message, and update information
  - Multi-line comments are properly formatted
  - Uses TheHive's query API to retrieve comments directly from cases
  - Requires `--ticket` to be specified
  - Example: `./hiveGetTicket.py -t 666 --comments`
  - Can be combined: `./hiveGetTicket.py -t 666 --history --comments`

- **Add Comments**: Added `--add-comment` option to add a comment to a case
  - Requires `--ticket` to specify which case
  - Accepts comment text as command-line argument
  - Supports reading from stdin using `-` as the comment value
  - Examples:
    - `./hiveGetTicket.py -t 666 --add-comment "Fixed the issue"`
    - `echo "Comment text" | ./hiveGetTicket.py -t 666 --add-comment -`
    - `./hiveGetTicket.py -t 666 --add-comment - < comment.txt`
  - Can be combined with other display options to add comment and view case details

- **Add Attachments**: Added `--add-attachment` option to attach files to a case
  - Requires `--ticket` to specify which case
  - Accepts file path as argument
  - Validates file exists before uploading
  - Displays file name, size, and upload confirmation
  - Examples:
    - `./hiveGetTicket.py -t 666 --add-attachment /path/to/evidence.pdf`
    - `./hiveGetTicket.py -t 666 --add-attachment screenshot.png`
    - `./hiveGetTicket.py -t 666 --add-attachment report.docx --add-comment "See attached report"`
  - Can be combined with `--add-comment` to add both in one command

- **List Alerts**: Added `--list-alerts` option to display alerts
  - Lists all alerts from TheHive with detailed information
  - Filter by status using `--alert-status` (e.g., New, Imported, Ignored)
  - Filter by title using `--title`
  - Shows title, source, severity, status, TLP, timestamps, tags, and URLs
  - Compatible with `--showdescription` to include alert descriptions
  - Example: `./hiveGetTicket.py --list-alerts --alert-status New`

- **Live Feed**: Added `--live-feed N` option to display recent activity
  - Shows recent alerts and case updates (similar to TheHive's Live feed page)
  - Displays most recent N activities sorted by timestamp
  - Includes severity, status, source/creator, and direct URLs
  - Example: `./hiveGetTicket.py --live-feed 20` (show 20 most recent items)

- **Case-Specific Live Feed**: Added `--case-live-feed` option to show activity for a specific case
  - Requires `--ticket` to specify which case
  - Shows all recent activities for that case: timeline events, comments, tasks, attachments
  - Activities sorted by timestamp (most recent first)
  - Includes action type, timestamp, user, and details
  - Example: `./hiveGetTicket.py -t 666 --case-live-feed`

- **Enhanced Debugging**: Improved `--DEBUG` option with comprehensive debug output
  - Shows API responses, field names, and data structures
  - Helps troubleshoot issues with comments, observables, and history retrieval
  - Example: `./hiveGetTicket.py -t 666 --comments --DEBUG`

### Bug Fixes
- **CRITICAL**: Fixed AttributeError when accessing case details with `--ticket` and `--history` options
  - The script was treating the API response as an HTTP response object when it's actually a list
  - Fixed all references from `tmpJson[0]` to `response[0]`
  - Added proper handling for field names with and without underscores (`_createdAt` vs `createdAt`, `_createdBy` vs `createdBy`)
  - Added safety checks for optional fields like `summary`
- **Fixed `--status` filter**: Now correctly filters on the `stage` field with case-insensitive matching
  - Valid values: New, InProgress, Resolved, Closed
  - Automatically capitalizes input (e.g., `new` → `New`, `inprogress` → `Inprogress`)
  - Shows helpful error message when no results found
- Fixed SyntaxWarning for invalid escape sequences in regex patterns (now using raw strings)
- Fixed SyntaxWarning in example strings
- Removed non-functional `--filter` option that was defined but never implemented
- Enhanced help text with comprehensive examples for all working options

## Version 2026-03-01 (Initial)

### New Features

1. **Keychain Integration**
   - API credentials are now stored securely in the system keychain
   - Use `--store-key` to save your TheHive URL and API key
   - Keys are automatically retrieved from keychain on subsequent runs

2. **Command-Line API Key Option**
   - Use `--api-key YOUR_KEY` to specify the API key on the command line
   - Use `--url YOUR_URL` to specify the TheHive URL on the command line
   - Command-line options override keychain values

3. **List All Tickets**
   - New `--list-all` option to list all available tickets
   - Can be combined with other filters (e.g., `--list-all --status New`)
   - Displays ticket number, title, stage, severity, and URL for each ticket

### Security Improvements
- Removed hardcoded API credentials from the script
- Credentials are now stored in the system keychain using the `keyring` library

### Usage Examples

```bash
# Store credentials in keychain (one-time setup)
./hiveGetTicket.py --store-key

# List all tickets
./hiveGetTicket.py --list-all

# List all new tickets
./hiveGetTicket.py --list-all --status New

# Get specific ticket (using keychain credentials)
./hiveGetTicket.py -t 666

# Get specific ticket with command-line API key
./hiveGetTicket.py --api-key YOUR_KEY -t 666

# List all tickets created by specific user
./hiveGetTicket.py --list-all --createdby user@example.com
```

### Dependencies
- Added dependency on `keyring` library for secure credential storage
- Install with: `pip install keyring`
