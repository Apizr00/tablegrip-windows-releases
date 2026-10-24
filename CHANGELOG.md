# Changelog

All notable changes to TableGrip. The format follows [Keep a Changelog](https://keepachangelog.com/), and versions
follow [Semantic Versioning](https://semver.org/).

## [2.3.0] - 2026-10-24

### Added
- **MongoDB** (phase J5, second half). Pick MongoDB in the new-connection picker: host and port (27017), optional user and
  password, TLS, and the database (empty is `test`). Instead of a host you can paste a full address
  (`mongodb://user:pass@host:27017/app?authSource=admin`, or `mongodb+srv://...` for Atlas), which is used as it is.
  - **A collection is a table, a document is a row.** The columns are the fields found in the first 200 documents (`_id`
    first); a field that turns up later on a page is added. Each column has the type its values have (`string`, `int`, `long`,
    `double`, `decimal`, `bool`, `datetime`, `objectid`, `uuid`, `object`, `array`, `binary`, or `mixed`). Objects and arrays
    show as JSON, an ObjectId as its 24 hex digits, a date as a date. The Structure tab shows the indexes, how many documents
    each field is in, the options of the collection and the shell commands that would create it again.
  - **The server does the searching.** Paging, sorting, the filter bar (equals, contains, `IN`, `BETWEEN`, null, ...), quick
    search (which finds numbers, dates and ids too) and the count are turned into a MongoDB query, so a big collection is not
    read into the program. A condition the server cannot be asked (a search through objects, a function) is evaluated on the
    documents, up to 100,000 of them, and a bigger collection is told to use a simpler condition.
  - **Edit like a table.** Change a field (it keeps the type it has: `5` stays text in a text field and is a number in a number
    field), set it to null, edit an object or array as JSON, add a row with **+ Row** (an empty `_id` gets a new ObjectId) and
    delete rows. Changes are staged, the SQL is shown first, and the commit is one bulk write. Every update names the fields it
    changes with the value they had, so a document changed by somebody else is reported and not overwritten; on a replica set
    the commit is a transaction and is rolled back as a whole, on a single server the changes made before a conflict stay
    and the message says so. **Truncate** deletes the documents and keeps the indexes.
  - **A console for shell commands.** A query tab takes `db.users.find({age: {$gt: 30}}).sort({age: -1}).limit(5)`, `findOne`,
    `countDocuments`, `distinct`, `aggregate`, `insertOne`/`insertMany`, `updateOne`/`updateMany`, `replaceOne`, `deleteOne`/
    `deleteMany`, `createIndex`, `dropIndex`, `getIndexes`, `drop`, `db.runCommand({...})`, `show collections` and `show dbs`,
    with the shell's looser JSON (`{name: 'Ada'}`, `ObjectId("...")`, `ISODate("...")`, `/regex/i`) and `//` comments. SQL
    can be mixed in. Safe mode sorts commands by what they do: reads pass, `insertOne` or `$out` asks, `deleteMany`, `drop` and
    `dropDatabase` are destructive, and a read-only connection refuses all of them.
  - **Administration** shows the operations running now (with kill), the server status, the databases and, on a replica set,
    its members.
  - A user made in another database than `admin` is found by trying that database after `admin`. A refused sign-in says so
    and how to name the database (`?authSource=`).
  - The driver and its compression libraries add about 6 MB to the download (102 to 109 MB for the full package). Tables, schemas, ER diagrams, migrations, backups and the other SQL tools are
    hidden or say that MongoDB has none.

### Changed
- `long` (a 64-bit integer) is a whole number to the grid, and an `object` is JSON: they are checked when typed, and the
  conflict check of an edit covers them.

## [2.2.0] - 2026-10-24

### Added
- **Redis** (phase J5, first half). Pick Redis in the new-connection picker (host, port 6379, optional user and password, TLS,
  database number; empty is 0). The keyspace of the database opens as one table, `db0`, with a row per key and the columns
  **key**, **value**, **type** and **ttl**, like in TablePlus: strings show as text, and hashes, lists, sets, sorted sets and
  streams show as JSON (a hash is an object, a list or set an array, a sorted set a flat `[member, score, ...]` array).
  - **Browse and search.** Paged with the count from `DBSIZE`, sorted by key, with the filter bar (equals, contains, `IN`,
    `BETWEEN`, null, ...), quick search, sorting and the *Structure* tab (key, value, type and ttl). Only the keys on screen
    have their values read; a string over 1 MB or a collection of more than 5,000 items shows a placeholder and is left alone.
  - **Edit like a table.** Change a value, the **ttl** (seconds; empty makes the key permanent) or the **key** (a rename), add a
    row with **+ Row** (give the value of a hash, list, set, sorted set or stream as JSON) and delete rows. Changes are staged,
    the SQL is shown first, and the commit is one `WATCH` / `MULTI` / `EXEC` transaction: if another client changed a key in the
    meantime nothing is applied and you are told which key. A key with binary content is listed but never written back.
    **Truncate** is `FLUSHDB`.
  - **A console for commands.** A query tab on a Redis connection takes Redis commands, one per line (`GET user:1`,
    `HGETALL session:abc`, `CONFIG GET maxmemory`, quoting as in `redis-cli`; lines starting with `#` are comments), and
    the SQL the grid writes. Replies are shown as rows (pairs for hashes, a line each for `INFO`). Safe mode sorts commands by
    what they do: a read passes, `SET` or `EXPIRE` asks (or is refused in read-only), `DEL`, `UNLINK`, `FLUSHDB` and
    `FLUSHALL` are destructive. `SELECT n`, subscriptions and `MULTI` are not available in a tab, with a message that says why.
  - **Administration** shows the sessions (`CLIENT LIST`, with kill), the server status and the databases (`db0` to `dbN`) from
    `INFO`, `CONFIG` and `CLIENT`.
  - It is a small client of its own (RESP2 over a socket, TLS, AUTH with user and password), so the download does not grow.
    Tables, schemas, views, ER diagrams, migrations, backups and the other SQL tools are hidden or say that Redis has none.
- **Engines without SQL.** The way Redis is mapped is general: the SQL that TableGrip writes (filters, pages, row edits) is read
  by a small parser and run against the engine's own commands, and a statement that is not SQL is sent as a command. MongoDB
  (next release) uses the same parts.

## [2.1.1] - 2026-10-24

### Added
- **Explain for DuckDB.** The **Explain** button of a query tab now works on a DuckDB connection: the plan is a tree of operators
  with the table, join condition and filter of each step and the rows DuckDB expects, a scan of a big table with no filter is
  marked as the hot spot, and **Analyze** (reads only) runs the query and shows each step's real rows and time.

### Changed
- **A smaller download.** The Windows 10 SDK projection that Windows Hello needs is no longer compiled ahead of time (it was
  53 MB of the program for two calls, and is now 24 MB; it is compiled when Hello is used). The installer and the update
  packages shrink by the same share.
- **No build warnings.** The 23 analyzer and nullable warnings of the solution are gone (a partial method with two names for one
  parameter, `Task` returns that could be concrete, a shared `JsonSerializerOptions`, log calls that built their arguments when
  logging was off, a null check in the start-up code, and the test classes). Nothing changed for the person using the program.
- **The guide says how to use a second DuckDB file**: attach it in a script (`ATTACH 'other.duckdb' AS other; SELECT ...`) and run
  the script in one go. Attached files are not listed in the sidebar yet, because the places that name a table (diagrams, usages,
  the designer) do not carry the file with them.

## [2.1.0] - 2026-10-24

### Added
- **DuckDB** (phase J2, first half). The new-connection picker offers DuckDB and a `.duckdb` or `.ddb` file is opened like a SQLite
  one (**File > Open database file**, drop, command line); other file names are told apart by their header, so the right engine is
  used for a recent file too. The sidebar shows the schemas, tables, views and sequences of the file; the structure view shows
  keys, indexes, foreign keys, checks and comments; lists, structs and maps read as DuckDB prints them; queries, editing,
  import, export, backup and restore, clone, truncate, the table designer and the command line (`tablegrip-cli`) all work, and
  `duckdb://path` is understood in a connection URL.
- **New...** next to the file name of a SQLite or DuckDB connection starts a new, empty database in a file you name.
- **Maintenance and status for DuckDB**: *Update statistics* (`ANALYZE`), *Optimize* for the whole file (`CHECKPOINT`) and a server
  status with the version, file size and memory in use.
- A DuckDB file is one database and has no users, so the pages for users, sessions and locks say so.

### Changed
- **The designer works around what DuckDB refuses.** DuckDB will not run `ALTER TABLE` on a table that has an index, so the SQL
  drops the indexes, changes the table and creates them again (and says so); a NOT NULL column is added in two steps; auto-numbered
  columns and foreign keys with ON DELETE actions are refused with an explanation; and foreign keys, checks and an existing primary
  key cannot be changed on an existing table, which is said in a warning instead of sending a statement that would fail.
- Renaming a table from the sidebar reads its indexes first on DuckDB, for the same reason.
- The download is about 14 MB larger, for the DuckDB engine that travels with it.

### Fixed
- A **date** or **time** read as a `DateOnly` or `TimeOnly` was written as `02/03/2000` in a backup, a copied INSERT and a CSV or
  JSON export (the format of the invariant culture), which engines read differently or refuse. They are now `2000-02-03` and
  `01:02:03`.

## [2.0.0] - 2026-10-24

### Added
- **CockroachDB and TimescaleDB** (phase J1). Both are in the new-connection picker and use the PostgreSQL driver: browsing,
  editing, import and export, the table designer, SSH and the TLS modes work as for PostgreSQL, and connection URLs may start
  with `cockroachdb://` or `timescaledb://`.
  - **TimescaleDB** is PostgreSQL itself. Its internal schemas (`_timescaledb_*`, `timescaledb_*`) are hidden from the sidebar.
  - **CockroachDB** opens on port 26257 and suggests the database `defaultdb`. It has no PostgreSQL monitoring views, so the
    sessions, locks, status, replication, users, table size and Explain pages say that the engine does not offer them
    instead of failing, and *Update statistics* (`ANALYZE`) is its only maintenance command. A cloned table keeps its
    numbers with a plain `INSERT`, since CockroachDB has no `OVERRIDING SYSTEM VALUE`.
  - Neither engine was run against a live server (none is available on the development machine); they are covered by
    tests of the statements and of the engine choice. Please report what a real cluster refuses.

### Changed
- The version is 2.0.0 because phase J (more engines) starts here; nothing that worked before changed.
- Amazon Redshift, the third engine of J1, is not included: its catalogs are those of PostgreSQL 8.0 and need their own
  queries, which cannot be checked without a Redshift cluster.

## [1.13.0] - 2026-10-24

### Added
- **Safe mode levels like TablePlus.** Each connection is now **Silent**, **Warn**, **Verify** or **Read-only**. Warn is the old
  *Confirm changes*. **Verify** asks first like Warn and then checks it is you: **Windows Hello** (face, fingerprint, PIN or the
  Windows password) or, where Hello is not set up, your **TableGrip password**; if neither exists the connection name is asked
  for instead. It covers running statements, saving row edits (also in a query result), applying a structure change, restoring a
  backup and applying migrations.
- **A TableGrip password and an app lock.** **Preferences > Safety** can set a password (kept only as a salted PBKDF2 hash) and
  **lock TableGrip when it starts**: Windows Hello or the password is asked for before any window with your connections opens.
  Turning the lock on first asks you to unlock once, and TableGrip opens unlocked rather than lock you out when nothing is left
  to unlock with.
- **Vim mode** in the SQL editor (Preferences > Editor). Normal, insert, visual and visual-line modes with a mode badge and a
  block caret; counts; the motions `h j k l w W b B e E 0 ^ $ gg G f F t T ; , % { }`; the operators `d c y > <` and `gu gU g~`
  with motions and the text objects `iw aw i" a" i' a' i( a( i{ a{ i[ a[ i< a<`; `x X D C s S Y r J ~ p P u Ctrl+R .`
  and `i a I A o O`; `v` and `V` selections. Yanks also go to the Windows clipboard. A whole insert is one undo step, and `.`
  repeats a change with the text you typed. Ctrl shortcuts Vim does not use (Ctrl+Enter, Ctrl+S, Ctrl+C...) keep working.
- **Several carets** in the SQL editor: **Ctrl+D** adds the next match of the selected text, **Ctrl+Shift+L** selects every match,
  **Ctrl+Alt+Up / Down** adds a caret above or below, **Ctrl+Alt+click** adds one where you click. Typing, Backspace, Delete,
  Enter, Tab, the arrows, Home / End, copy, cut and paste act at every caret as one undo step; Esc goes back to one caret.
- **Custom themes.** **Preferences > Appearance > Theme file** applies a `.json` theme over the light or dark colours, including
  the SQL syntax colours. *Export the theme in use...* writes every colour as a starting point, *Import...* checks a file and
  copies it into the `themes` folder, and a theme edited by hand is read again when it is chosen.
- **A font for the data grid** (Preferences > Data grid), next to the existing size.

### Fixed
- The **editor font, editor font size and grid font size** in Preferences did nothing since 1.0.0 (the glass look replaced the
  service that applied them). They apply again, and immediately.

### Changed
- Safe mode names in the connection form: *Off* is now *Silent* and *Confirm changes* is *Warn (ask before changes)*; saved
  connections keep their setting.
- TableGrip is built against the Windows 10 (19041) SDK so it can use Windows Hello; the installer and the update feed are unchanged.

## [1.12.1] - 2026-10-22

### Added
- **SQL export like in TablePlus.** On the **SQL** tab of the export window every ticked table has three boxes, **Structure**
  (its `CREATE`), **Drop** (a `DROP ... IF EXISTS` before it) and **Data** (its rows as multi-row `INSERT`s), and the boxes in the
  header tick or clear a whole column. Tables are written in foreign-key order and views last, as a definition only (a view has no
  Data box). Only the chosen tables are written, as one `.sql` file that **Import from SQL dump** can run again.
- **Save As and a file name.** The export window now has a *File name* box and a **Customize** button (CSV delimiter, quote, line
  break and decimal; SQL rows per `INSERT` and one-file-or-many), and **Export...** opens the *Save As* window for one file or asks
  for a folder when each table goes into its own file.
- **Gzip.** *Compress the file using Gzip* writes `.sql.gz`, `.csv.gz` or `.json.gz` (not Excel, which is a ZIP already).
- **Export in Background.** The running export is a small card with the table's name, *rows done / rows in all the tables* and a
  bar, and **Export in Background** hides it: the export carries on, shown in the notifications (the bell), and the console
  records the result when it finishes.
- **A pixel pill for waiting.** The bars of export, import, backup and restore are now the same pill of pixels as the connection
  pill; while the size of the work is not known yet (reading structure, counting rows) a band of pixels sweeps across it.

- **Notifications.** A bell in the title bar lists the background jobs: the name, the tables, the pixel pill with the percentage
  and time left while it runs, then **Completed** with **Open Folder** (or what went wrong). The list opens by itself for a few
  seconds when a job ends; **Clear** removes the finished ones. It replaces the chips that were in the footer.
- **A short success window.** After an export the window says only *Export completed successfully*, with the logo and the
  buttons *Open Containing Folder*, *Don't show this again* and *Close*. The rows, time and notes go to the console.

### Fixed
- The project did not build: `DetailsView` had a method defined twice.
- The success window listed every table of a database with circular foreign keys (hundreds of names). The note now names five
  and counts the rest.
- *Cannot locate resource 'themes/colors.dark.xaml'*: the colours are now loaded by the name of the TableGrip assembly, so the
  theme also applies when the program is started through another host (for example `dotnet TableGrip.dll`).

## [1.12.0] - 2026-10-19

### Added
- **Edit the structure in the table tab.** On the **Structure** side press **Edit structure**: the columns, indexes, foreign keys
  and checks become a grid you can change (add, rename, retype, remove, move). Nothing runs while you edit: **Review SQL** (or
  **Ctrl+S**) shows the exact `ALTER` statements, and **Discard changes** puts everything back. A tab with unapplied structure
  edits asks before it is closed. After the SQL ran, the table opens again on the new structure. The same grids are used by the
  designer window.
- **Import like in TablePlus.** Right-click a table > **Import** > **From CSV...**, **From JSON...** or **From SQL dump...**: the
  file is chosen first (with the right file filter), then the import window opens already on that file; a SQL dump (`.sql` or
  `.sql.gz`) starts the restore straight away, after the usual confirmation.
- **A better progress panel for export, import, backup and restore.** One panel shows what is being written, a percent bar,
  *rows done of rows in all*, the speed (rows or statements per second), the time elapsed and the time left, and, when several
  tables are written, a second bar for the whole job ("Table 3 of 12"). A restore now reports every statement it has run, a
  backup counts the rows of each table first, an import shows the share of the file, and a single-table export counts the table
  so its bar is real. The footer chip of a background job shows the same percentage and time left.
- **The sidebar filter remembers its text.** The tables filter comes back with the tabs next time the connection opens, and the
  tables, queries and history lists each keep their own filter while you switch between them.
- **Saved queries in folders.** Right-click a saved query > **Move to folder...**: the Queries list then groups them under the
  folder names (queries without a folder come first). **Make a snippet with a keyword...** turns a saved query into a snippet:
  type the keyword in any query and press Tab to insert it.

### Changed
- The provider call that runs a list of statements in one transaction can report how many have run (used by restore).

## [1.11.2] - 2026-10-19

### Fixed
- **The export window no longer grows with the number of tables.** With hundreds of tables it became taller than the screen and the
  Export and Cancel buttons were out of reach. The window now has a fixed size that never exceeds the screen: the list of tables
  scrolls inside it and the buttons stay at the bottom.

## [1.11.1] - 2026-10-19

### Added
- **A fuller table menu.** Right-click a table: **Item overview** (rows, size, columns, key, indexes, foreign keys, checks), **Pin to
  top** (a PINNED section at the top of the list, kept per connection), **Export** > Export tables... / with column selection,
  **Import** (CSV or JSON file, SQL dump), **New** (table, view, schema, database), **Copy Script As** (the SQL straight onto the
  clipboard), **Clone** (the table with or without its rows), **Truncate** and **Delete**. Clone and Truncate show their SQL first.
- **A new export window.** Tick the tables to export (the selected ones start ticked), choose CSV, JSON, SQL or Excel and set the
  CSV file up: NULL as empty or the word NULL, line breaks inside values as spaces, the header row, delimiter, quoting (if needed,
  always, never), `\r\n` or `\n` line ends and a dot or comma for decimals. The single-table export has the same CSV options.
- **Export progress and a Success message.** The window shows the table being written and rows done of rows in the table with a bar
  and Cancel; when it finishes a Success message offers *Open containing folder* and *Don't show this again* (Preferences turns it
  back on).

## [1.11.0] - 2026-10-19

### Added
- **Edit the result of a query.** When a query is a plain `SELECT` from one table (columns or `*`, with at most WHERE, ORDER BY and
  LIMIT), its result can be edited like the table: an edit bar appears, Ctrl+S commits, and the query runs again to show what is
  stored. Joins, grouping, DISTINCT, UNION, computed or renamed columns and CTEs stay read-only.
- **Tables without a primary key can be edited.** Rows are found by a unique key of NOT NULL columns, or else by all their values; a
  change that would touch more than one row is refused and nothing is saved. A note beside the edit bar says which is in use.
- **No overwriting someone else's change.** When you commit, the values you are replacing must still be what you read (text, integers,
  decimals, booleans, GUIDs), or the commit is refused and nothing is written. Preferences > "Refuse to save an edit over a value that
  someone else changed" turns it off.
- **Pickers in cells.** A column with a fixed set of values (MySQL or PostgreSQL enums, or a CHECK that lists them) opens a list; a
  true / false column opens TRUE / FALSE / NULL (Space flips it); F4 or Alt+Down opens the picker of any cell: the rows of the
  referenced table for a foreign key (searchable), or a calendar with a time for date, time and timestamp columns.
- **More Copy as formats:** Markdown table, SQL UPDATE (by key) and a WHERE ... IN list.
- **Edit in the Details panel.** The fields are live when the table is: leaving a box stages the value, NULL and Format JSON buttons,
  Esc restores the stored value, Alt+Up / Alt+Down move to the previous / next row.
- SQLite check constraints are now read from the table definition, so they show in the Structure tab as well.

### Fixed
- The Messages tab of a script was hard to read on dark backgrounds.

## [1.10.0] - 2026-10-18

### Added
- **The tabs come back.** When a connection is opened again, its tabs are as you left them: each table with its sort, filter,
  page and Data / Structure side, each query with its text and cursor. They are written down every few seconds and when you
  close the connection or the window. Preferences > "Reopen the tabs of the last session" turns it off.
- **SQL files.** File > Open SQL file... (Ctrl+Shift+O) reads a `.sql` file into a query tab; Ctrl+S saves it (asking for a name
  the first time), Ctrl+Shift+S saves under another name, and File > Recent SQL files lists the last ten. A tab whose text
  differs from its file shows a dot after its name, and closing it asks first.
- **One result tab per statement.** Running several statements shows a tab for each one that returns rows (Result 1, Result 2, ...)
  and a Messages tab with every statement's row count and time, or the error that stopped the script.
- **Ctrl+S commits.** In a table, Ctrl+S saves the staged edits from anywhere in the window, even while a cell is being typed
  in (on a safe-mode connection the SQL preview opens first). Ctrl+Shift+R shows the SQL of the staged edits without saving.
  These and the SQL file keys can be changed in Tools > Keyboard shortcuts.

### Changed
- The editor bar's *Save* button, which kept the query under Queries in the sidebar, is now called *Favorite*; *Save* writes the file.

## [1.9.3] - 2026-10-18

### Added
- **Switch database.** Database > Switch database... (Ctrl+Shift+D, or the palette) lists the databases of the server that the
  login can open (MySQL, MariaDB, PostgreSQL, SQL Server) and opens the one you choose with the same login, tunnel and settings,
  without saving another connection. The old connection closes once the new one is open; if it has unsaved edits and you keep
  them, both stay open.

### Fixed
- **Builds with SQL Server support no longer fail the security audit.** The SQL Server driver pulled in a version of
  `System.Security.Cryptography.Pkcs` with known vulnerabilities; the patched build is now used.

## [1.9.2] - 2026-10-14

### Improved
- **Smoother interface.** Buttons, list rows, tabs, segmented switches and the window buttons now fade their hover, press and
  selected states in and out (about a tenth of a second in, a little longer out) instead of changing in one frame. Text boxes
  fade their focus ring in. Combo box and sub-menu lists fade open.
- **Dialogs and tabs arrive instead of popping up.** A dialog fades in while rising a few pixels into place, and a tab fades
  in each time it is opened or switched to. All of this is skipped when Windows has animation effects turned off.
- **Lists scroll by pixel.** The sidebar and the other lists glide with the mouse wheel instead of jumping a whole row at a time.
- **Sharper, consistent look.** The SQL editor's find panel (Ctrl+F) and tooltips use the app's colours and rounded shape instead
  of the grey system boxes. The find panel has match case, whole word and regular expression switches and previous / next / close
  buttons.

## [1.9.1] - 2026-10-14

### Fixed
- **A script's statements now share one connection.** Each statement used to run on a connection of its own, so in a query tab
  (Run all, or a selection with several statements) and in `tablegrip-cli query`, `BEGIN ... COMMIT / ROLLBACK`, temporary
  tables and `SET` / `PRAGMA` settings made by one statement were gone for the next ("no such table", "cannot rollback - no
  transaction is active"). All four engines now hold one connection for the whole run; a transaction left open at the end is
  rolled back. A single statement is unchanged.
- `tablegrip-cli query` in table format no longer breaks a row apart when a value contains a line break.

### Improved
- Reading the structure of a table in a SQLite database takes about a third of the time in a database with many tables (one
  connection instead of several, each of which had to read the whole schema again). Backing up a database with 1,500 tables
  went from 20 s to 6.6 s for the structure.
- Importing a CSV into SQLite is about 40% faster (the same prepared statement is reused for each batch of rows): 1 million
  rows in 9.6 s instead of 16 s.
- Rebuilding the sidebar list (typing in the filter) tells the list once instead of once per table.

## [1.9.0] - 2026-10-14

### Added
- **Compare schema** (Database > *Compare schema...*). Pick a source and a target (either can be another saved connection) and
  see which tables are new, changed or only in the target, and what differs (columns, types, defaults, keys, indexes, foreign
  keys). Untick a difference to leave it out. The script that makes the target match the source is shown for review: copy it,
  save it, open it in a query tab or keep it as a migration. *Include drops* also lists what exists only in the target. Nothing
  is run. Two different engines are listed but no script is written, because types do not translate.
- **Migrations** (Database > *Migrations...*). A folder of numbered SQL files (`0001_add_orders.sql`) applied in order, each in
  one transaction, and recorded in a `tablegrip_migrations` table so each runs once. The list shows what is pending, applied,
  edited since it was applied (the file's checksum no longer matches) or missing. *Apply all pending*, *Apply selected*, *New
  migration...*, *Open* and *Show in folder*. Safe mode and the production lock apply as everywhere else.
- **Dashboard** (Database > *Dashboard...*). Response time and session count as live sparklines (the last 60 readings, every 3
  seconds, with median and highest), next to the server's status facts (version, size, uptime, ...). *Pause updates* and
  *Refresh now*.
- **Customizable keyboard shortcuts** (Tools or Edit > *Keyboard shortcuts...*). Select an action, click the box and press the
  new keys; it takes effect at once and the menus and the command palette show it. A shortcut another action already uses is
  refused with that action's name. *Remove*, *Reset* and *Reset all*. Dashboard, Administration, Compare schema, Migrations
  and the shortcuts window can be given keys too. The keys of the SQL editor itself are fixed.
- **Diagnose** button in the connection editor: a step-by-step check of the path to the database (settings, the database file,
  the SSH server's name and port, the SSH sign-in and tunnel, the server's name and port, the database login, a first
  query), each with its time, and for a failure what it most likely means and what to try.
- **Recent objects.** Tables you opened come first in the command palette (and Ctrl+P) when the search box is empty.
- **Command line.** `tablegrip-cli list | tables | query | export | backup` (the script `tablegrip-cli.cmd` next to
  TableGrip.exe) uses the saved connections and their stored passwords. `query` prints a table, CSV, JSON, SQL or an Excel
  file; `--out` writes to a file. Queries are read-only unless `--write` is given, and a connection tagged Production also needs
  `--yes-production`. A connection that asks for its password each time reads `TABLEGRIP_PASSWORD`. Exit code 0 is done, 1
  failed, 2 the command line was not understood.
- **Extensions.** Off until *Preferences > Extensions* is ticked. A .NET library in `%APPDATA%\TableGrip\extensions` that
  implements `ITableGripExtension` adds commands to the command palette ("Extension: name - command"); a command can run a
  read-only query on the open connection, open SQL in a query tab for review, or show a message. A sample is in
  `docs/samples/HelloExtension`. Extensions run with your rights, so only use ones you trust.
- **Shared connection profiles** (File > *Publish shared connections...* and *Sync shared connections...*). Writes the saved
  connections to a file a team can read (a network folder, a repository) and brings such a file in line: new names are added,
  known names take the shared host, port, tag and so on, and keep their own password choice. The file holds no password and no
  SSH secret; people are asked for their own the first time. Database file connections are left out because a path means
  nothing on another computer.

### Changed
- The window's keyboard shortcuts are read from your settings instead of being fixed in the window.

### Notes
- Custom database drivers are not planned: the extension API adds commands, not database engines.
- The command line is the windowless TableGrip.exe attached to a console. Use `tablegrip-cli.cmd` (or `start /wait`) so the
  shell waits for it and gets the exit code.
- Schema compare and migrations for PostgreSQL, MySQL, MariaDB and SQL Server are checked as text; the SQLite paths run against
  real files. They have not been run against live servers yet.

## [1.8.0] - 2026-10-13

### Added
- **Administration window** (Database > *Administration...*, or Ctrl+K). Pages for **Sessions** (who is connected, what each
  runs and for how long, with the full query below; *Cancel query* stops it, *End session* closes the connection; idle
  sessions can be shown), **Locks** (which session waits for which, and the blocking one), **Server status** (version, uptime,
  connections, size, cache hit ratio and other health numbers), **Users and roles** and **Replication**. Each page refreshes on
  demand or every 5 seconds. They need the account to be allowed to look; if it is not, the page says so.
- **Users, roles and grants.** The users page lists the accounts and what the selected one may do (privileges per table and
  role memberships). *New user...*, *Grant...*, *Revoke...*, *Password...* and *Drop...* write the statement and show it in
  the Review SQL window first; a password you type is in that text (the change log masks it).
- **Maintenance.** Right-click a table > *Maintenance*, or Database > *Maintenance*: update statistics, optimize / vacuum,
  rebuild indexes, check for corruption, each with the engine's own command (ANALYZE, VACUUM, OPTIMIZE TABLE, REINDEX, UPDATE
  STATISTICS, DBCC CHECKTABLE, PRAGMA integrity_check). Commands that cannot run in a transaction (VACUUM) run on their own, and
  what a command printed (the rows of CHECK TABLE) is shown after it ran.
- **Backup manager.** Database > *Manage backups...* lists the backups in a folder (by default *TableGrip backups* in
  Documents; *Change folder...* remembers another) with date, size, what they hold and the engine they came from. *Restore...*
  opens the restore panel on the selected one, *Check* reads the whole file (unpacking `.gz`) to make sure it is complete,
  *Delete* removes it, and *Keep newest...* deletes all but the newest few after showing the list. *Back up now...* opens the
  backup panel.

### Notes
- The queries for PostgreSQL, MySQL, MariaDB and SQL Server (sessions, locks, status, users, replication) are checked as
  text; the SQLite pages run against real files. They have not been run against live servers yet. PostgreSQL and SQL Server
  need the account to be allowed to see other sessions and the server's dynamic views; MySQL 8 needs the sys schema for
  locks (MariaDB uses information_schema).

## [1.7.0] - 2026-10-13

### Added
- **Table designer.** Right-click a table > *Design table...* (or *New table...* in the Database menu, or Ctrl+K) to edit its
  columns (name, type, NULL, default, key, auto-numbering), indexes (including unique ones), foreign keys (with ON DELETE and
  ON UPDATE) and checks. Renaming a column renames it in the indexes and keys that use it. Nothing changes until you choose
  *Review SQL...*.
- **Review SQL.** Every structure change (designer, rename, drop, new schema or database) opens a window with the exact
  statements and what to watch out for. *Run* executes them in one transaction (safe mode and the production lock apply, and
  the change goes into the change log); *Copy SQL* and *Open in a query tab* let you run or adjust it yourself.
- **SQLite changes it cannot make in place.** Changing a column's type, nullability or default, the primary key, a foreign key
  or a check makes SQLite rebuild the table: a new table is created, the rows are copied, and it replaces the old one. The
  review says so and warns that triggers on the table are not recreated.
- **Routines, triggers, sequences and materialized views** are listed in the sidebar below the tables (PostgreSQL: functions,
  procedures, triggers, sequences, materialized views; MySQL and MariaDB: functions, procedures, triggers; SQL Server:
  functions, procedures, triggers, sequences; SQLite: triggers). Double-click one to read its source in a query tab; right-click
  to drop it.
- **Generate SQL.** Right-click a table > *Generate SQL* for a SELECT, INSERT, UPDATE or DELETE with its columns written out,
  or its CREATE or DROP statement, in a query tab.
- **Drop and rename** a table or view from its right-click menu; *New view...* starts a CREATE VIEW in a query tab.
- **Schemas and databases.** Database > *New schema...*, *Drop schema...* (PostgreSQL and SQL Server), *New database...* and
  *Drop database...* (server engines).

### Notes
- The statements for PostgreSQL, MySQL, MariaDB and SQL Server are covered by tests on their text; the SQLite ones are also
  run against real files. They have not been run against live servers yet, so read the review before you run it there.

## [1.6.0] - 2026-10-13

### Added
- **Import JSON.** The import window now reads a JSON file too: an array of objects (`[{"id": 1, ...}, ...]`) or one object
  per line (JSON Lines / NDJSON). The columns are the property names; a missing or `null` property becomes NULL, a nested
  object or array is kept as JSON text, and numbers keep the digits they were written with.
- **What to do with existing rows.** Importing CSV or JSON can now *Stop and change nothing* (the default), *Skip the
  existing row* or *Update the existing row* when the primary key is already in the table. PostgreSQL and SQLite use
  `ON CONFLICT`, MySQL and MariaDB `INSERT IGNORE` / `ON DUPLICATE KEY UPDATE`, SQL Server a `NOT EXISTS` / `MERGE` per row.
  Update changes only the columns that are in the file.
- **Text encoding for CSV.** UTF-8 (default, byte order marks honoured), UTF-16, Windows-1252, Windows-1250, Windows-1251 and
  ISO-8859-1.
- **Type hints.** Beside each column mapping the import window says what the file column looks like ("looks like whole
  numbers", "dates", ...).
- **Choose the columns to export.** The export window lists the columns with check boxes (All / None); only the chosen ones
  are written, in every format.
- **Compressed backups.** *Compress the file (gzip, .sql.gz)* in the backup panel; restore reads `.sql` and `.sql.gz`.
  *Import SQL dump (restore)...* is now in the File menu.
- **Run in background.** Exports, imports, backups and restores have a *Run in background* button: the window closes, a chip
  in the footer shows the progress (with Stop), and when the job ends the window flashes in the taskbar if you are in
  another program. The chip stays, with *Show in folder*, until you dismiss it.
- **Several sort columns.** Shift+click a column header to add it to the sort, again to reverse it, a third time to remove it;
  the headers show the order (name 1, created 2). A plain click still sorts by that column alone.
- **Hide and show columns.** Right-click a column header: *Hide "x"*, *Show all columns* or tick columns under *Columns*.
- **Paste rows and columns.** Ctrl+Alt+V (or right-click > *Paste as rows and columns*) pastes a block copied from a
  spreadsheet or another grid: the first cell goes into the active cell, the rest to the right and below, and rows past the
  end become new rows. Everything is staged, so it can be reviewed and undone.
- **Binary viewer.** Double-click a binary cell (or right-click > *View binary value...*, or Ctrl+E): the file type it seems to
  be, a picture for PNG, JPEG, GIF, BMP and WebP, the text when it is plain text, and a hex dump. *Save to file...* writes
  the bytes out; *Copy as 0x...* copies them as SQL hex.

## [1.5.0] - 2026-10-13

### Added
- **Format SQL.** Ctrl+Shift+F (or Format on the run bar, or Query > Format SQL) lays out the selection, or all the SQL:
  one clause per line, one select item per line, AND / OR under WHERE, subqueries, CASE and CREATE TABLE indented,
  keywords in capitals. Only white space and the case of keywords change; strings, names and comments stay as they are.
  It is an edit like any other, so Ctrl+Z takes it back.
- **Find and replace.** Ctrl+H opens a bar above the editor with Find next, Replace, Replace all, Match case, Whole word and
  Regex (with `$1` in the replacement). Replace all is one undo step.
- **Code folding.** Statements of several lines, bracketed groups, CASE and BEGIN blocks, block comments and dollar-quoted
  bodies get fold markers in the margin.
- **Split editor.** Split (Ctrl+\) shows a second view of the same text under the first, to read one part of a long script
  while editing another. Zoom the text with Ctrl+wheel; Ctrl+0 resets it.
- **Connection URLs.** Paste `postgres://user:password@host:5432/db?sslmode=require` (or a `mysql://`, `mariadb://`,
  `sqlserver://` or `sqlite:` URL) into the host box of a new connection and the form fills in: host, port, database, user,
  password and TLS mode.
- **Connect timeout and retries.** Each connection has a timeout (1 to 300 seconds) and a number of retries (0 to 5) for when
  the server cannot be reached. A wrong password is never retried; each retry is noted in the console log.
- **Menus.** The ⋮ menu of a connection is grouped as File, Edit, View, Connection, Database, Query, Tools, Window and Help.
  New: **Window > New window** (Ctrl+Shift+N) and **Connection > Reconnect**; both are also in the command palette.

## [1.4.1] - 2026-10-13

### Changed
- **Ctrl+F in a table opens the filter.** It now shows the filter conditions under the search box, and the cursor
  goes to the search box. The Filter button still opens and closes the panel.

## [1.4.0] - 2026-10-11

### Added
- **Reorder tabs by dragging.** Drag a tab along the tab strip to move it; it follows the pointer as it passes its
  neighbours. Its editor text, undo history and scroll position stay as they were.
- **Groups and order on the start screen.** Drag a connection onto a group to put it in, or between two connections
  (a line shows where) to place it there, in that connection's group or out of any group when it is not in one.
  Right-click and choose New > Group... to make a group, which stays listed while it is empty; right-click a group to
  remove it (its connections are kept). A connection's menu also has Move to group, for doing the same without the mouse.
  Groups move too: drag a group's header above or below another group.
- **A picture for a group.** Right-click a group and choose Set picture... to pick an image (PNG, JPEG, BMP, GIF, WebP or
  ICO). It is cropped to a square, scaled down and shown as a round picture beside the group's name, on the start screen
  and in the Open connection list. Remove picture takes it away; removing the group removes its picture too.

## [1.3.0] - 2026-10-11

### Added
- **Query plans (Explain).** Ctrl+L in a query tab shows how the engine would run a statement, step by step, with each
  step's rows, cost and share of the work, and the likely hot spots marked: a full scan of a big table, a step that is
  most of the cost, a sort or hash that spilled to disk, an estimate that is far off. PostgreSQL can Analyze a read for real
  row counts and times; MySQL, MariaDB, SQLite and SQL Server show their estimated plans.
- **ER diagram.** The tables of a database as boxes with their columns and the foreign keys as lines, laid out
  automatically; focus on one table and the tables around it, choose how many columns show, click to highlight relations,
  export a PNG or an SVG. Crow's-foot notation, key columns first, a layout that reads left to right instead of
  running down the page, and a resizable window with Full screen (F11) and Fit.
- **Compare data.** Compare two tables (of one database or two, even of different engines) by key and get the rows only on
  one side, the changed rows, and a script that makes the target match the source. The script is for review; nothing is run.

## [1.2.0] - 2026-10-11

### Added
- **Snippets.** Type a trigger word and press Tab (or pick it from the suggestions) to insert a snippet with places to
  type: `${1:name}` fields, Tab to move on, `$0` for where the caret ends. Eight starters ship (`sel`, `ins`, `upd`,
  `del`, `join`, `grp`, `dup`, `selc`); add your own from Snippets in the workspace menu, or save the selected SQL as one.
- **Production lock.** On a connection tagged production, destructive statements, committed row deletes and restores ask
  for the connection's name to be typed. Preferences > Safety turns it off.
- **Column insights.** Analyze columns on the Structure side shows each column's share of NULLs, distinct values, and
  smallest and largest value; the table and its indexes show their size where the engine says it.
- **Find usages.** Right-click a table in the sidebar or a column on the Structure side: the foreign keys to and from it
  and the views, routines and triggers that mention it.
- **More filter operators, in groups.** The filter's operator list now reads `=` `<>` `<` `>` `<=` `>=`, `IN` / `NOT IN`,
  `IS NULL` / `IS NOT NULL`, `BETWEEN` / `NOT BETWEEN`, `LIKE`, then Contains, Not contains, Has prefix and Has suffix,
  with a divider between the groups. `IN` takes a list (commas or lines, quotes for a value with a comma), `BETWEEN` gets a
  second box, and `LIKE` is your own pattern.
## [1.1.0] - 2026-10-11

### Added
- **Follow relationships.** Right-click a cell > *Follow relationship*: go to the row a foreign key points at, or list
  the rows of other tables that point at the row you are on. The target opens filtered, so the filter bar shows what it
  is looking at.
- **An arrow in foreign key cells.** A key cell that holds a value ends in a small arrow; click it to open the row it
  points at, no right-click needed.
- **A fuller right-click menu on the grid:** *Set value*, *Reload rows*, *Paste* and *Paste to selected cells* (staged and
  undoable), *Copy all values of this column*, *Quick filter* by the cell's value, *Import CSV* and *Export rows*. Right-click
  now acts on the cell under the pointer.
- **Query parameters.** Write `:name` in a query and TableGrip asks for the values when it runs; they are sent as real
  parameters. History keeps the placeholders, so running a query again asks again.
- **Pin history entries.** Right-click an entry in History > *Pin to top*. Pinned queries stay at the top and survive
  *Clear history* and the size limit.
- **Change log.** What was committed on a connection, when, and the SQL (edits, data-changing statements, CSV imports),
  from the workspace menu or the command palette. Search it, copy an entry's SQL, or clear it. It stays on this computer.
- **Connection health.** The typical round trip to the server with a rating, plus server version, TLS and uptime, so a
  slow network can be told from a slow query.

## [1.0.2] - 2026-10-11

### Fixed
- **Release notes showed stray asterisks.** Text written between single stars in the update window's notes and in
  What's new is now shown in italics instead of with the stars around it.

## [1.0.1] - 2026-10-11

### Added
- **How far behind you are.** When several versions were skipped, the update notice, the update window and the status
  line say so (for example *Version 1.0.0 is available (8 updates behind)*), and *What's new* lists the notes of every
  version you skipped, newest first. The update still goes straight to the newest version.

### Removed
- **The update address box** in Preferences: TableGrip always checks the official releases on GitHub. *Check now* and the
  start-up check stay.
- **The Diagnostics setting** (saving a crash report file) and the feature behind it. Unexpected errors are shown in a
  dialog and written to the log as before.

## [1.0.0] - 2026-10-11

First stable release. The look is new (frosted glass, floating panels; the original look stays available as *Classic*),
and everything else from the 0.9 series is unchanged. The installers are not code-signed yet.

### Changed
- **A glassmorphism look.** Frosted, translucent panels float over a soft, colourful abstract backdrop (blue, violet,
  pink and teal shapes, a fine grain on top). Each panel shows a blurred copy of the backdrop behind it,
  tinted white in the light theme and deep navy in the dark theme, with a bright hairline edge. Primary actions and links
  use a blue accent. Text colours are checked against the glass over every backdrop colour.
- **Floating layout.** The navigator, the workspace, the row details and the status bar are separate rounded cards with
  gaps between them; tabs are pills; the connection name in the title bar is a command bar (click it, or Ctrl+K). A
  2 px line in the environment's colour runs along the top of the workspace, so production is unmistakable.
- **The start screen** is two glass cards (brand and actions, connections). Dialogs, the connection editor, the colour
  picker and the command palette are glass sheets over the same backdrop.
- **Choose your look.** Preferences > Appearance has a new **Look** choice: *Glass* (the new frosted look) or *Classic*
  (the original flat look: opaque dark or light panels in the original colours, no backdrop art). The colour scheme
  (System, Light, Dark) and the layout (floating or docked) work with both; Glass effects apply to the Glass look.
- **Preferences > Appearance** gains *Glass effects* (Auto follows Windows' "Transparency effects" and High Contrast; On;
  Off gives opaque panels with the same layout and colours) and *Layout* (Floating, or Docked with flush panels).
- **Refresh no longer draws a blue line** across the top of the workspace: the command bar already fills as it
  progresses. The line remains while a connection opens (the command bar is not shown yet) and inside a tab that is
  running a query or loading a table.
- `docs/roadmap.md`: the suggested features to add next, ranked.

### Fixed
- **Drop-down lists and menus were see-through** on the glass look, so their entries overlapped the text behind them
  (the Preferences font list, for example). Drop-downs, menus and the SQL completion list are opaque now.

## [0.9.11] - 2026-10-10

### Changed
- **Database logos instead of letters.** The database type picker, the start screen and the open-connection list now show
  each engine's logo (white, on a disc in the engine's colour): MySQL, MariaDB, SQLite, PostgreSQL, SQL Server, DuckDB,
  Cassandra, CockroachDB, Snowflake, Redshift, Redis, MongoDB, DynamoDB, Elasticsearch, ClickHouse, BigQuery, libSQL and
  D1. Engines without a logo (Vertica, Greenplum) keep their two letters. The logos come from Simple Icons (CC0) and are
  trademarks of their owners.

### Fixed
- **The colour picker's wheel and sliders did nothing when dragged.** The picker opened as a pop-up window that closed
  itself as soon as the mouse was pressed inside it. It is now a sheet inside the editor (click the dimmed area, press Esc
  or the tick to close), so the wheel and the sliders follow the mouse. Enter in its hex box applies the colour instead
  of saving the connection.

## [0.9.10] - 2026-10-10

### Added
- **A colour picker for connections.** Click the colour block next to *Status color* in the connection editor: a colour
  wheel (hue and saturation) inside a ring that shows the colour in force, a hex box, copy and done buttons, and
  saturation and brightness sliders. The star saves the colour to a list of your own (kept between sessions; click a saved
  colour to use it, right-click to remove it).
- **Your own environment tags.** The *Tag* drop-down keeps the usual list (None, local, development, testing, staging,
  production) and now ends with **Custom...**: type a name of your own (up to 24 characters, e.g. *QA* or *client-x*). Tags
  you create stay in the list for the next connection, and the **bin** button next to the drop-down deletes one (after
  asking; connections that used it simply have no tag). Custom tags show in the start screen, the connection rail and
  the title-bar pill like the built-in ones; only *production* turns confirmations on by itself.

### Changed
- **TableGrip is licensed under the MIT License.** The source code stays private; the program is free to use. `LICENSE`
  and `THIRD-PARTY-NOTICES.md` (the open-source components it is built on, with their licences) now ship in every build
  and are published with the downloads.
- **The new / edit connection window is redesigned.** Captions sit on the left with the fields to their right, in rounded
  sheets: *Name*, *Status color* (the colour in force, the swatches and the environment *Tag* drop-down), the server fields
  (*Host/IP*, *Port*, *User*, *Password* with its storage drop-down, *Database*, *SSL mode*, certificates), an **Over SSH**
  button that lights up while the tunnel is on and opens its own sheet (server, user, password, **SSH key**), then *Group*
  and *Safe mode*. **Save**, **Test** and **Connect** are at the bottom; the **x** or Esc closes it. Nothing was removed.

## [0.9.9] - 2026-10-10

### Added
- **Any colour for a connection.** Under *Colour* in the connection editor, next to the swatches, type red / green / blue
  (0-255) or a `#RRGGBB` value; the preview dot and the boxes follow whatever you pick.

### Fixed
- **Button text is centred.** Text in the dialog buttons (OK, Cancel, Save, ...) sat at the left edge; it is centred now.
  The start screen's icon buttons keep their left alignment.

## [0.9.8] - 2026-10-10

### Added
- **A proper update experience.** TableGrip looks for a new version when it starts and, if there is one, opens
  *A new version of TableGrip is available!* with the release notes. **Install Update** downloads it with a progress
  bar, then **Install and Restart** installs it. **Remind Me Later** keeps an orange notice on the start screen
  (with **What's new** and **Update**), and **Skip This Version** hides that version until a newer one appears.
- Help > Check for updates and Preferences > Check now open the same window.

### Changed
- The start-up update check is **on by default** (turn it off in Preferences or in the update window). It contacts only
  the release page and sends nothing about you or your data. If you had it off before, it is on again once.

### Fixed
- **All popups can be moved.** The connection editor, engine picker, backup / restore, password prompt and the command
  palette (drawn inside the window) can be dragged by their empty areas, like the dialogs; they return to the centre
  each time they open.

## [0.9.7] - 2026-10-10

### Fixed
- **Dialogs can be moved.** About, Preferences, Export, Import, Open connection, Confirm and Prompt are borderless windows
  and could not be dragged. Drag any empty part of a dialog to move it; buttons, inputs and lists behave as before.

## [0.9.6] - 2026-10-10

### Fixed
- **Check for updates and the Help links work for everyone.** 0.9.5 looked for updates in the source repository, which is
  private, so every check ended with "404 (Not Found)". Downloads, the update feed, the user guide, the changelog and the
  issue tracker now live in the public `Apizr00/tablegrip-releases` repository, and the app uses that address.
- The update prompt shows the release notes.

### Note
- Copies of 0.9.5 and older still look at the old address: install 0.9.6 once with the installer; updates work from then on.

## [0.9.5] - 2026-10-10

### Fixed
- **Check for updates works out of the box.** With no update address set it used to only say "No update address is set".
  It now looks at the project's GitHub releases when you ask (Help > Check for updates, or *Check now* in Preferences),
  asks before downloading, and restarts to install a small delta package. An address you enter still takes priority, and
  `github.com/owner/repo` addresses are understood. The start-up check stays opt-in and off by default.
- Releases now carry the update feed files (`releases.<rid>.json` and the `.nupkg` packages) next to the installers; see
  `docs/release.md`.

## [0.9.4] - 2026-10-10

### Added
- **Copy as** in the results grid's right-click menu: copy the selected cells with a header line (tab-separated, **Ctrl+Shift+C**),
  as CSV, as JSON, or as SQL INSERT statements in the connection's dialect. Rows keep the order on screen.

## [0.9.3] - 2026-10-10

### Changed
- **Several connections open at once in one window.** The connection button now opens the picked connection next to the
  current one instead of in a new window. A rail of connection tabs on the left (shown from two connections on) switches
  between them; each connection keeps its own sidebar, tabs and unsaved changes. The window's **X** closes the active
  connection, then the next, then the application.
- The "open another connection" button shows a database with a plus instead of a power symbol.

## [0.9.2] - 2026-10-10

### Added
- **Select several tables** in the sidebar (Ctrl+click, Shift+click, **Ctrl+A**) and export their data together: one file per
  table (CSV, JSON, Excel) or one SQL script, with a right-click menu (open in new tab, open structure, copy name, export).
- **Search in every table**: Ctrl+F focuses the search box of the open table or query result; query results get a
  search box that narrows the loaded rows and shows how many match.
- **Open another connection** from the connection button in the title bar: a searchable list (Open connection) that opens the
  chosen connection in a **new window**, so several connections can be used at the same time. **New...** starts the new-connection
  flow in a new window.

### Changed
- The window's **X** button closes the connection first and returns to the start screen; pressing it again closes the
  application. (Alt+F4 still closes the window at once.)
- Polished header and footer: an environment chip and styled engine / database text in the connection pill, a segmented
  Data | Structure switch, a connection indicator, and a grouped pager with the query time in the footer.
- The filter panel is one line per condition (column, operator, value, Apply, remove, add).

## [0.9.1] - 2026-10-10

### Added
- **Backup database** and **Restore databases** on the start screen: a whole database as one SQL script (structure and
  rows, foreign-key order) and a restore that runs a script as one transaction, with confirmation and safe-mode checks.
- **Export / Import connections** (JSON, never containing passwords) and a right-click menu on the connection list
  (New, Export Connections, Import Connections).
- Application menu (**⋮**) on the start screen: File, View, Tools, Help, and an **About TableGrip** window.

### Changed
- The start screen no longer shows the line of keyboard hints under the buttons (the shortcuts still work and are in the menu).

## [0.9.0] - 2026-10-10

First release candidate: every planned feature is in. What is left before 1.0 is outside the code: a real code-signing
certificate, a test on a clean machine, and a hosted update feed (see `docs/release.md`).

### Added
- **Databases:** SQLite, MySQL / MariaDB, PostgreSQL and SQL Server. One connection opens one database.
- **Start screen** with saved connections (groups, environment tags, colours), recent files, drag-and-drop of database files.
- **Table browser:** server-side sorting and filtering, paging, a Structure tab (columns, keys, indexes, DDL), a details panel for the selected row.
- **Editing:** inline edits staged and reviewed as SQL before they are written, in one transaction; undo and redo; insert, duplicate and delete rows.
- **SQL editor:** syntax highlighting, autocomplete that knows the tables and aliases in the statement, run statement / selection / script, cancel, query history, saved queries.
- **Export** to CSV, JSON, SQL and Excel; **import** from CSV with column mapping and validation before anything is written.
- **Command palette** (Ctrl+K) and **search everything** (Ctrl+P); light, dark and system themes; preferences.
- **SSH tunnels** (password or key, host keys remembered with trust-on-first-use), **TLS modes** (disabled to verify-identity),
  **safe mode** per connection (confirm changes, read-only), **Windows Integrated Authentication** for SQL Server.
- **Windows 10 / 11 integration:** per-monitor DPI awareness, remembered window position that survives a disconnected
  monitor, x64 and ARM64 builds, installer with automatic updates.
- **Opt-in diagnostics:** crash report files (no queries, no data) and update checks. Both are off by default.

### Changed
- New application icon and logo (installer, taskbar, start screen, dialogs).

### Security
- Passwords and SSH secrets are stored only in `secrets.json`, encrypted for the current Windows user (DPAPI).
- Query history masks `IDENTIFIED BY '...'`, `PASSWORD '...'` and similar literals.
- Error messages from drivers are scrubbed of `Password=` / `Pwd=` values before they are shown.

### Accessibility
- Every control has a name for screen readers; list entries are announced with their meaning (table, view, connection and environment).
- Toolbar buttons are reachable with Tab and show a focus ring; clicking them with the mouse does not steal the editor's focus.
- Text colours meet WCAG AA contrast (4.5:1) in both themes; a test keeps them that way.

### Fixed
- Closing the app showed an "unexpected problem" dialog (the service container was disposed synchronously).
- On a second monitor the window jumped to the primary monitor when a database was opened, and could start partly off screen.

### Known limitations
See the end of `docs/architecture.md`.
