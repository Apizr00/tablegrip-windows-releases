# TableGrip user guide

## Connecting

1. **Create connection** (Ctrl+N), choose the database type, fill in host, port, user, password and **database name**.
   Each connection opens exactly one database.
2. **Test** checks the settings without saving. **Save** keeps it in the list; **Connect** saves and opens it.
3. SQLite files can also be opened directly (Ctrl+O) or dropped onto the window.
4. **Groups and order.** Right-click the list and choose New > Group... to make a group. Drag a connection onto a
   group header to put it last in that group, or onto another connection (a line shows whether it goes above or below)
   to place it there, in that connection's group or out of any group when that one has none. Dropped on the empty space it
   goes last, out of any group. *Move to group* in a connection's menu does the same without the mouse. Drag a group's
   header above or below another group to reorder the groups (dropped on another group's connections it goes below that
   group; on the ungrouped connections, first; on Recent files or the empty space, last). *Remove group*
   (right-click the group) keeps its connections and just ungroups them.
5. **Group pictures.** Right-click a group and choose *Set picture...* to show an image beside its name. The image is
   cropped to a square and scaled down; *Remove picture* clears it. Pictures are kept with the group on this computer
   (in `connections.json`) and are not part of an exported connections file.
6. **Connection URL.** Instead of filling in each box, paste a URL into the **Host** box, for example
   `postgres://app:secret@db.example.com:5432/shop?sslmode=require`. TableGrip fills in host, port, database, user, password
   and the TLS mode. It understands `postgres://`, `postgresql://`, `mysql://`, `mariadb://`, `sqlserver://` (with
   `;databaseName=...;user=...` properties) and `sqlite:` URLs, `jdbc:` in front, and `%40`-style escapes in the password.
   A URL for another engine than the one you are editing is refused with a message.
7. **Timeout and retries.** *Timeout (s)* is how long to wait for the server (default 15). *Retries* is how many more
   times to try when the server cannot be reached (a timeout, a refused or dropped connection); a wrong password or an
   unknown database is never retried. Each retry is written to the console log.
8. **Diagnose.** When a connection does not work, *Diagnose* in the editor walks the path to the database one step at a time:
   the settings, the database file, the SSH server (name and port), the SSH sign-in and tunnel, the database server (name and
   port), the database login and a first query. Each step shows its time; the first one that fails stops the check, says what
   it most likely means and what to try, and the steps after it are marked as skipped. *Copy report* puts the result on the
   clipboard for a bug report. Recently opened tables come first in the command palette (Ctrl+K, Ctrl+P).

### Passwords
- *Store securely* encrypts the password for your Windows account. It is never written to `connections.json`.
- *Ask every time* asks when you connect and keeps it in memory only.
- *No password* is for servers that do not need one.

### Over SSH
Turn on **Over SSH** in the connection form, then enter the SSH server, user and either a password or a private key file.
The database host is the address *as seen from the SSH server*. The first time you connect, TableGrip shows the server's
fingerprint; compare it with the one your administrator gave you before you trust it. If it ever changes, TableGrip
refuses to connect and tells you why.

### SSL mode
| Mode | Meaning |
|---|---|
| Disabled | No encryption. |
| Preferred | Encrypt if the server offers it. |
| Required | Always encrypt; the certificate is not checked. |
| Verify CA | Also check that the certificate comes from the CA you give (or a trusted one). |
| Verify identity | Also check that the certificate matches the host name you typed. |

Through an SSH tunnel the host name is `127.0.0.1`, so use *Verify CA* there.

### Tag
The *Tag* drop-down marks what a connection is for: **None**, **local**, **development**, **testing**, **staging** or
**production**. Choose **Custom...** to type a name of your own (up to 24 characters); your tags stay in the list for
the next connection. While one of your own tags is chosen, the bin button next to the drop-down deletes it from the list
(after asking; connections that used it simply have no tag). The built-in tags cannot be deleted. The tag is shown next to the name on the start screen and in the title-bar pill. Only
**production** switches safe mode to *Confirm changes* by itself.

### Colour
Each connection has a colour dot on the start screen and in the connection rail. Under *Colour* pick one of the swatches
(the first one is the database type's default) or any colour of your own: type **R**, **G**, **B** (0-255) or a
`#RRGGBB` value, and press Tab or click away to apply it. Click the colour block next to *Status color* for the **colour
picker**: a wheel for hue and saturation, a brightness and a saturation slider, the hex value with copy and done buttons,
and a star that saves the colour to your own list (click a saved colour to use it, right-click to remove it).

### Safe mode
Per connection: **Off**, **Confirm changes** (every statement that writes or changes the schema asks first; destructive
ones are marked) and **Read-only** (writes are refused). Marking a connection *production* turns Off into Confirm.
Safe mode is a guard against mistakes, not a security boundary: for a hard guarantee use a database account without
write privileges.

**Production lock.** On a connection tagged *production*, a destructive statement (`DROP`, `TRUNCATE`, `UPDATE` or
`DELETE` without a `WHERE`), committing deleted rows, and restoring a backup each ask you to **type the connection's
name** before they go on, instead of just clicking OK. Turn it off in Preferences > Safety. Read-only connections still
refuse writes.

## Backup and restore

On the start screen, **Backup database...** and **Restore databases...** work on any saved connection (or a recent
SQLite file), without opening it in the workspace. Inside a database, the same two actions are in the **⋮** menu.

- **Backup** writes one SQL script: the structure the database reports for each table and view, then the rows as
  multi-row INSERT statements. Tables come in foreign-key order and views last. Options: structure, data, and
  `DROP TABLE IF EXISTS` before each table so a restore replaces what is there.
- **Restore** runs a SQL script (a TableGrip backup, or any dump) on the chosen connection **as one transaction**, so on
  PostgreSQL, SQL Server and SQLite a failure leaves the database exactly as it was. MySQL and MariaDB commit every
  CREATE/DROP on their own, so a failed restore there can be partly applied; the message says so. You are asked to
  confirm first, with the number of statements and how many existing objects the script drops. A read-only connection
  refuses to restore, and a connection in *Confirm* mode shows the same question.
- The script is held in memory while it runs, so a file above 256 MB is refused. For a backup that large use the
  database's own tools (`pg_dump`, `mysqldump`, `sqlcmd`).
- Not included: users and privileges, stored procedures and functions, triggers the engine does not report with the
  table, and (PostgreSQL) sequence positions.

## Moving connections to another computer

Right-click empty space in the connection list, or use **File** in the **⋮** menu:

- **New** creates a connection or opens a database file.
- **Export Connections** writes all connections, or just the selected one, to a JSON file. **Passwords and SSH secrets are
  never included**; they are asked for again.
- **Import Connections** adds the connections of such a file. They get new identities and a free name ("Shop (2)"), so
  nothing is overwritten.

## The menu

The **⋮** button at the top right of the start screen opens a menu: **File** (new, open, import, export, backup,
restore, exit), **View** (theme), **Tools** (preferences) and **Help** (user guide, report a problem, releases, check for
updates, **About TableGrip**).

Inside a connection, the **⋮** button beside the sidebar button opens the same actions grouped as **File**, **Edit**, **View**,
**Connection** (reconnect, disconnect, health), **Database** (refresh, ER diagram, backup, restore), **Query** (run, run
all, explain, format, cancel), **Tools** (compare data, change log), **Window** (new window, next and previous tab) and
**Help**. **Ctrl+K** finds any of them by typing.

## Several connections at once

The **new connection button** (a database with a plus) at the top left of a connection opens the **Open connection** list. Pick one (search, group folding,
**New...**) and it opens **in the same window**; the connection you are in stays open.

Once two or more connections are open, a **rail of connection tabs** appears on the far left, one entry per connection
(coloured by its environment tag, captioned "Name : database"). Click an entry to switch. Each connection keeps its own
sidebar, tabs, selection and unsaved changes while you work in another one. Middle-click an entry, use its small **x**, or
right-click and choose **Close connection** to close just that one.

The window's **X** closes the active connection (the next one becomes active); with none left you are back on the start
screen, and one more press closes the application. If any open connection has unsaved changes you are asked first.

### Switching database

A connection is bound to one database, but you do not have to save a connection per database. **Database > Switch database...**
(or **Ctrl+Shift+D**, or "Switch database" in the palette) lists the databases this login can see on the server; pick one and it
opens with the same login, SSH tunnel and settings in place of the current one. If the current one has unsaved edits you are
asked first, and if you keep them both stay open. The saved connection is not changed. SQLite has nothing to switch to (a file is one database).

## Working with tables
- Click a table to preview it, double-click to keep the tab. Click a column header to sort (on the server).
- **Select several tables** with Ctrl+click or Shift+click, or all of them with **Ctrl+A**, then right-click for **Export**
  (CSV, JSON, SQL INSERT or Excel) or press Ctrl+Shift+E. The data goes to one file per table in a folder you choose, or to a
  single .sql file for all of them.
- **Search** every table: **Ctrl+F** puts the cursor in the search box of the open table (server side, all columns), or of a
  query result (the loaded rows, with a "12 of 250 rows" count). Ctrl+F in the sidebar filters the list of tables.
- **Filter** with conditions (column, operator, value; **+** adds a line, **-** removes it) or the quick search; filters apply
  to the whole table, and to export. The operators are listed in groups: `=` `<>` `<` `>` `<=` `>=`; `IN` / `NOT IN` (values
  separated by commas or lines; put quotes around a value that contains a comma); `IS NULL` / `IS NOT NULL`; `BETWEEN` /
  `NOT BETWEEN` (two boxes, both ends included); `LIKE` (your own pattern: `%` any text, `_` one character); and the plain
  text searches *Contains*, *Not contains*, *Has prefix* and *Has suffix*, which treat `%` and `_` as ordinary characters.
- **Edit** a cell by double-clicking or pressing F2. Changes are *staged* (coloured) until you commit. **Preview** shows
  the exact SQL; **Commit** writes it in one transaction, and nothing is saved if one statement fails.
  Tables without a primary key are read-only.
- **Copy** the selected cells with Ctrl+C (tab-separated, full values). Right-click > **Copy as** adds a header line
  (**Ctrl+Shift+C**), CSV, JSON, or SQL INSERT statements for the selected rows and columns.
- **Follow relationships.** A foreign key cell that holds a value ends in an arrow (**→**): click it to open the table it
  points at, showing just that row (a column that is in several keys asks which). Right-click a cell > **Follow
  relationship** also lists *Rows that point at this row*: every table with a foreign key to this one, opened showing the
  rows that reference the row you are on. The arrows need only the table's own keys; the second list reads the keys of
  the whole database the first time (up to 500 tables) and keeps them for a few minutes.
- **Right-click a cell** for: *Edit value*, *Set value* (NULL, empty string), *Reload rows*, *Paste* (the clipboard
  text as one value into the cell; *Paste to selected cells* into every selected cell; staged like typing, so it can be
  undone), *New row*, *Duplicate row*, *Copy* and *Copy as*, *Copy all values of this column*, *Quick filter* (equals,
  does not equal, contains, is NULL, is not NULL, or clear), *Follow relationship*, *Import CSV*, *Export rows* and
  *Delete row*. The menu acts on the cell under the pointer.
- **Structure** shows columns, keys, indexes and the CREATE statement.
- **Analyze columns** (Structure side) counts, for every column, the share of NULLs, the distinct values and the smallest
  and largest value. It only reads; a table of more than two million rows is analyzed on its first 200,000. The Structure
  side also shows how big the table and its indexes are (MySQL, MariaDB, PostgreSQL, SQL Server and SQLite with its dbstat
  table), and the cell value of each key can be followed with the arrow described above.
- **Find usages.** Right-click a table in the sidebar (or a column on the Structure side, or press Ctrl+K and search
  *Find usages*) to list the foreign keys that point at it or leave it, and the views, routines and triggers whose stored
  text mentions it (a whole-word name search, so it says "mentions"). Double-click a row to open the table, or to read the
  routine in a query tab. Do this before dropping or renaming something.

## SQL editor
| Action | Keys |
|---|---|
| Run the statement at the caret, or the selection | Ctrl+Enter or F5 |
| Run every statement | Ctrl+Shift+Enter |
| Stop | Esc |
| Suggest names | Ctrl+Space (also automatic after two letters or a dot) |
| Format the SQL | Ctrl+Shift+F |
| Find and replace | Ctrl+H |
| Split the editor / zoom | Ctrl+ / Ctrl+wheel, Ctrl+0 resets |

**Format, find and fold.** **Format** (Ctrl+Shift+F) lays out the selection, or all the SQL, with one clause per line, one
select item per line and keywords in capitals; nothing but white space and keyword case changes, and Ctrl+Z undoes it.
**Ctrl+H** opens find and replace above the editor (Match case, Whole word, Regex with `$1` groups; *Replace all* is one
undo step). Ctrl+F still opens the editor's own search. The margin has fold markers for multi-line statements, bracketed
groups, CASE and BEGIN blocks and block comments. **Split** (Ctrl+\) shows a second view of the same text below the first.
**Ctrl+wheel** zooms the text of the tab and **Ctrl+0** resets it. Hold **Alt** and drag to select a block of columns
(box selection) and type to change every line of it at once.

**Parameters.** Write `:name` in a query (`WHERE customer_id = :id`) and TableGrip asks for the value each time it runs;
the values are sent as parameters, never pasted into the text. Leave a box empty or type NULL for NULL; numbers and
true / false are sent as such, anything else as text (put quotes around a number to send it as text, or write
`CAST(:d AS date)` for a date). Colons inside strings, comments and `::` casts are left alone. History keeps the query
with its `:name` placeholders, so running it again asks again.

**Snippets.** Type a trigger word and press **Tab** to expand a snippet, or pick it from the suggestions (marked
*snip*). In its text, `${1:name}` is a place to type with a default (Tab moves to the next one; the same number repeats
what you typed) and `$0` is where the caret finishes. A few starters ship with TableGrip (`sel`, `selc`, `ins`, `upd`,
`del`, `join`, `grp`, `dup`); **Snippets...** in the workspace menu adds your own, and *Save the selection as a snippet*
(Ctrl+K) turns the SQL you selected into one. Yours are kept in `snippets.json`.

**Several statements, several results.** Run all (Ctrl+Shift+Enter) or select a script and run it: each statement that returns
rows gets its own tab (**Result 1**, **Result 2**, ...) under the editor, and a **Messages** tab lists every statement with
its row count and time (or the error that stopped the script, which is then the tab you land on). Click a tab to look at that
result; search, copy and export work on the tab you are looking at. A single statement shows no tabs, as before. Up to 25
result tabs are kept per run; the rest still appear in Messages.

**SQL files.** **File > Open SQL file...** (Ctrl+Shift+O, or *Open* on the editor bar) reads a `.sql` file into a query tab
(a file bigger than 8 MB is refused: use *Import SQL dump* for those). **Ctrl+S** saves the text to that file; for a tab
that has no file yet it asks for a name, and **Ctrl+Shift+S** saves under another name. The tab is named after the file and
shows a dot (`orders.sql ●`) while its text differs from the file; closing it then asks first. **Recent SQL files** in the
File menu lists the last ten. Files are written as UTF-8 without a byte order mark.

**Tabs come back.** When a connection is opened again, its tabs are as you left them: each table with its sort, filter, page
and Data / Structure side; each query with its text, cursor and file. Staged edits that were not committed are not kept.
The tabs are written down every couple of seconds and when you close the connection or the window, in `sessions.json` next
to the other app files (it holds your query text, never passwords). Turn it off in **Preferences > Reopen the tabs of the
last session**.

**History.** Every run is added to **History** in the sidebar; type in the search box to find one. Right-click >
**Pin to top** keeps a query at the top of the list, and pinned queries survive *Clear history*. **Save query** keeps
one under **Queries**.

## Changing the structure
Right-click a table in the sidebar for **Design table...**, **Rename...**, **Drop table...** and **Generate SQL**; the
**Database** menu (⋮ beside the sidebar button) has **New table...**, **New view...**, **New schema...**, **New database...**,
**Drop schema...** and **Drop database...**.
- **The designer.** *Columns*: name, type (click a suggested type to put it in the selected column), NULL, default (a SQL
  expression such as `0`, `'new'` or `CURRENT_TIMESTAMP`), key and auto-numbering (only when a column is created).
  *Indexes*, *Foreign keys* (the table it points at, its columns, ON DELETE / ON UPDATE) and *Checks* work the same way.
  A table with unsaved edits has to be committed or discarded first; after a change its open tabs are closed.
- **Review SQL.** The designer's **Review SQL...** and every other structure action show the SQL, with warnings, before
  anything runs. **Run** executes it as one transaction (MySQL and MariaDB do not roll back structure changes, and the window
  says so if a script stops half way); read-only connections refuse it, confirming connections ask, and the production lock
  asks for the connection name. It is recorded in the change log. **Open in a query tab** lets you edit it first.
- **SQLite.** SQLite can add, rename and drop plain columns and indexes in place. Anything else (a type, NULL or default
  change, the primary key, foreign keys, checks) rebuilds the table: copy the rows into a new table and swap it in. The review
  lists the steps and warns that triggers on the table must be recreated by hand.
- **Other objects.** Functions, procedures, triggers, sequences and materialized views are listed below the tables. Double-click
  one to read its source in a query tab; right-click for *Show definition* and *Drop...*.
- **Generate SQL.** SELECT, INSERT, UPDATE and DELETE templates with the columns written out (a `?` marks each value to fill in),
  plus the table's CREATE and DROP statements, open in a query tab.

## Administration
**Database > Administration...** (or Ctrl+K and type *Administration*) opens a window with five pages. Each has *Refresh*, and
*Refresh every 5 seconds*. They only read until you press an action button, and every action goes through the Review SQL window.
- **Sessions.** One row per other connection: id, user, database, host, state, seconds and query. Select one to read its whole
  query. *Cancel query* stops the running query and keeps the connection (PostgreSQL, MySQL, MariaDB); *End session* closes the
  connection (SQL Server can only do this). *Show idle sessions* adds the ones doing nothing.
- **Locks.** Sessions that wait for another one, with the blocking session beside them. End the blocking one (carefully) from
  Sessions.
- **Server status.** Version, uptime, connections, database size and the engine's own health numbers.
- **Users and roles.** The accounts, and for the selected one its privileges and roles. *New user...*, *Grant...* (read, add,
  change, delete or everything, on one table or on everything in the schema), *Revoke...*, *Password...* and *Drop...*.
- **Replication.** The replicas of this server, or the primary it follows; a note when none is set up.

A page that the engine cannot show (SQLite has no sessions, users or replication) says so. If the account is not allowed to
look, the page shows the server's message.

## Maintenance and backups
- **Maintenance.** Right-click a table > *Maintenance* (or Database > *Maintenance* for all tables): *Update statistics*,
  *Optimize / vacuum*, *Rebuild indexes*, *Check for corruption*. The review shows the exact command (ANALYZE, VACUUM,
  OPTIMIZE TABLE, REINDEX, UPDATE STATISTICS, DBCC, PRAGMA integrity_check) and warnings about locks and time. What the command
  printed is shown under it afterwards. VACUUM cannot run inside a transaction, so it runs alone; a SQLite VACUUM needs free
  disk space about the size of the file.
- **Backup manager.** Database > *Manage backups...* shows the `.sql` and `.sql.gz` files in the backup folder (change it with
  *Change folder...*). *Restore...* (or double-click) opens the restore panel on the file, *Check* reads the whole file to be
  sure it is complete, *Show in folder*, *Delete*, and *Keep newest...* removes all but the newest few after listing them.

## Plans, diagrams and comparing data
- **Explain (query plan).** In a query tab, **Explain** (Ctrl+L) shows how the engine would run the statement at the caret
  or the selection, step by step: the operation, what it works on, estimated rows and cost, and each step's share of the
  work. Steps that deserve a look are marked: a red edge for the likely hot spot (a full scan of a big table, a step with
  half the cost, a sort or hash that spilled to disk) and an amber one for remarks (a whole index read, a temporary sort,
  an estimate ten times off). Nothing is executed. On PostgreSQL, **Analyze** runs a *read* and reports real row counts and
  times; it is not offered for a statement that changes data. MySQL and MariaDB show the tabular plan, SQLite its query
  plan and SQL Server the estimated showplan (with the missing indexes it suggests). *Copy plan as text* puts the plan in the
  clipboard.
- **ER diagram.** **ER diagram...** in the workspace menu (or *Show in ER diagram...* on a table) draws the tables of the
  database with their columns and the foreign keys between them, related tables side by side and tables without relations
  below, in crow's-foot notation (a foot at the table that holds the key, bars at the table it points at, a circle where the
  key may be NULL); primary key columns come first. The window is resizable, **Full screen** (F11) fills the screen, and **Fit**
  zooms so the whole picture is visible. Choose a table and *within N relations* to see only its neighbourhood, show all columns, only key columns or just
  names, zoom with Ctrl+wheel, click a table to highlight its relations, double-click to open it, and **Export PNG** or
  **Export SVG**. It reads the keys of up to 500 tables.
- **Compare data.** **Compare data...** in the workspace menu compares two tables by key, each from this connection or
  another saved one (so staging against production, or a table against its backup). The key columns default to the
  primary key. It lists the rows that exist only in the source, only in the target, and the rows whose values differ
  (column by column), then writes the script that would make the target match the source: INSERTs, UPDATEs of the changed
  columns and, if you tick it, DELETEs. Values are compared by meaning (1 equals 1.0, true equals 1). **Nothing is changed**:
  review the script, copy or save it, or open it in a query tab and run it there, where safe mode, the production lock and
  the change log apply. The target is read into memory, so it is limited to 500,000 rows.

## Change log and connection health
Both are in the workspace menu (the button beside the sidebar) and in the command palette (Ctrl+K).
- **Change log** lists what was committed on this connection (tick *All connections* for every one): table edits you
  committed, statements run from a query tab that change data or structure, and CSV imports, each with the time and the
  SQL as it ran, values included. Search it, copy the SQL of an entry, or clear it. The file (`audit.json` in your data
  folder) stays on this computer; passwords typed into a statement are masked.
- **Connection health** times a few do-nothing statements and shows the typical round trip with a rating, plus the server
  version, TLS and (MySQL, MariaDB, PostgreSQL, SQL Server) uptime. A slow round trip means the network, VPN or tunnel is
  slow; if it is fast, a slow query is slow inside the server. It changes nothing.

## Export and import
- **Export** (Ctrl+Shift+E): CSV, JSON, SQL INSERTs or Excel. A table can be exported whole (current sort and filter) or
  just the loaded rows, and you can untick columns you do not want in the file.
- **Import CSV or JSON** (Ctrl+Shift+I): the format is detected from the file. For CSV the delimiter is detected and you can
  choose the text encoding; for JSON the file is an array of objects or one object per line, and the property names are the
  columns. Columns are matched by name and can be remapped; beside each one the window says what the values look like. The
  file is validated *before* anything is written, and an error leaves the table untouched.
- **If the key exists.** Choose what happens to a row whose primary key is already in the table: *Stop and change nothing*
  (default), *Skip the existing row*, or *Update the existing row* (only the columns in the file change). Skip and update
  need the table's primary key columns to be mapped; update needs a primary key.
- **Import SQL dump.** *File > Import SQL dump (restore)...* runs a `.sql` or `.sql.gz` script on the connection as one
  transaction. Backups can be written compressed: tick *Compress the file* in the backup panel.
- **Run in background.** While an export, import, backup or restore runs, *Run in background* closes its window. A chip in
  the footer shows the progress and has Stop; when the job ends the window flashes in the taskbar if another program is in
  front, and the chip stays (with *Show in folder* for files) until you dismiss it with the x.

## Columns, sorting and pasting
- **Sort** by several columns: click a header for that column alone (ascending, descending, off); **Shift+click** adds a
  column to the sort or reverses / removes it. The headers show the order.
- **Hide columns.** Right-click a header: *Hide "name"*, *Show all columns*, or *Columns* to tick which are shown. The
  choice lasts as long as the tab is open.
- **Paste rows and columns.** **Ctrl+Alt+V** pastes tab-separated rows (from Excel or another grid) starting at the active
  cell; rows past the last one are added as new rows. Plain **Ctrl+V** still pastes the clipboard as one value.
- **Binary values.** Double-click a binary cell, or press Ctrl+E, to see its type, a picture (PNG, JPEG, GIF, BMP, WebP), its
  text when it is text, and a hex dump; save it to a file or copy it as `0x...` hex.

## Comparing schemas and migrations

**Compare schema** (Database > *Compare schema...*) shows how two databases differ in structure. Pick the *Source* (the
one that is right) and the *Target* (the one to bring in line); either can be another saved connection. *Compare* lists the
tables that are new, changed, or only in the target, and selecting one shows what differs: columns, types, defaults, keys,
indexes and foreign keys. Untick a difference to leave it out of the script. *Include drops* adds what exists only in the
target (tables, columns, indexes), which is off by default so nothing is dropped by accident.

The script at the bottom makes the target match the source. Nothing is run: copy it, save it, open it in a query tab, or
choose *Keep as a migration*. Two different engines are compared and listed, but no script is written, because types do not
translate.

**Migrations** (Database > *Migrations...*) is a small, honest workflow. A migration is an SQL file named
`0001_add_orders.sql` in a folder (*Change folder...* picks it; by default a folder named after the connection in Documents).
*New migration...* writes the next numbered file, or keep a Compare schema script as one. *Apply all pending* runs the
pending files in order, each in one transaction, and records it in a `tablegrip_migrations` table of the database so it runs
once. The list says whether each file is:

- **pending**: not run on this database yet,
- **applied**: run, with the date,
- **edited**: run, but the file changed afterwards, so the database is not what the file says (nothing is re-run),
- **missing**: recorded in the database but the file is gone.

Safe mode and the production lock apply: a read-only connection refuses, and a production connection asks for its name first.
MySQL and MariaDB commit structure changes on their own, so a failed migration there can leave part of it applied.

## Dashboard

Database > *Dashboard...* shows how the database is doing while the window is open. **Response time** and **Sessions** are
sparklines of the last 60 readings (one every 3 seconds) with the latest value, the median and the highest. Under them are
the status facts the server reports (version, size, uptime, cache hit ratio, ...). *Pause updates* freezes it; *Refresh now*
reads once. SQLite has no other sessions to count, and the facts depend on what the account may see.

## Command line

`tablegrip-cli.cmd` (next to `TableGrip.exe`) runs TableGrip without a window, for scripts and scheduled jobs. It uses your
saved connections and the passwords stored for them. A connection is its name or its id.

```
tablegrip-cli list
tablegrip-cli tables  -c Shop
tablegrip-cli query   -c Shop --sql "select id, name from customers" -f csv -o customers.csv
tablegrip-cli export  -c Shop --table orders -f json -o orders.json
tablegrip-cli backup  -c Shop -o shop.sql
```

- `query` prints an aligned table by default; `-f` picks `csv`, `json`, `sql` or `xlsx` (a file with `-o`). With several
  statements, the result of the last one is written.
- Queries are **read-only** unless `--write` is given. On a connection tagged Production, `--write` also needs
  `--yes-production`.
- A connection that asks for its password each time reads it from the `TABLEGRIP_PASSWORD` environment variable (an SSH secret
  from `TABLEGRIP_SSH_SECRET`). An SSH host key that is not trusted yet is refused: connect once in the app first.
- The exit code is 0 when it worked, 1 when it failed (the reason is on the error output) and 2 when the command line was
  not understood. `tablegrip-cli help` lists everything.
- Use `tablegrip-cli.cmd` rather than `TableGrip.exe cli`: the program has no console of its own, and the script makes the
  shell wait for it and keep its exit code.

## Extensions

Extensions add commands to the command palette. They are off until you tick **Preferences > Extensions**, because an
extension is a .NET library that runs inside TableGrip with your rights: use only code you trust.

An extension is a class that implements `ITableGripExtension` (from `TableGrip.Core.dll`), in a `.dll` in
`%APPDATA%\TableGrip\extensions`. Each command appears as **Extension: name - command** after a restart. A command receives a
context that can run one read-only statement on the open connection (statements that change anything are refused), open SQL in
a query tab for you to review, and show a message. The command palette entry **Extensions...** lists what was loaded and why a
file was not. A working sample, with build notes, is in `docs/samples/HelloExtension`.

## Shared connections

A team can share how to reach its databases without sharing secrets. **File > Publish shared connections...** writes the saved
connections to a file you choose (a network folder or a repository both work) and remembers it. **File > Sync shared
connections...** reads that file: a new name is added, a known name takes the shared host, port, database, user, SSL, SSH, tag
and group, and keeps your own id, your password choice and when you last opened it. Connections that are only on your computer
are left alone.

The file has **no password and no SSH secret**; a connection that had a stored password asks each person for their own the
first time. Database file (SQLite) connections are not published, since a path on one computer means nothing on another.

## Keyboard

These are the defaults. **Tools > Keyboard shortcuts...** (or the palette) changes the ones that work anywhere in the window:
select an action, click the box and press the new keys. A shortcut that another action already uses is refused. The keys of
the SQL editor and the grid (Ctrl+Enter, Ctrl+Z, copy and paste, ...) are fixed.

| Keys | Action |
|---|---|
| Ctrl+K | Command palette |
| Ctrl+P | Search tables and saved queries |
| Ctrl+N / Ctrl+O | New connection / open file |
| Ctrl+T / Ctrl+W | New query tab / close tab |
| Ctrl+Shift+N | New window |
| Ctrl+Tab, Ctrl+Shift+Tab | Next / previous tab |
| F5 | Refresh |
| Ctrl+Shift+D | Switch database |
| Ctrl+S | Commit the staged edits of a table; in a query, save the SQL file |
| Ctrl+Shift+S | Save the SQL file under another name |
| Ctrl+Shift+R | Review the SQL of the staged edits without saving |
| Ctrl+Shift+O | Open a SQL file |
| Ctrl+F | Filter the sidebar (or search connections) |
| Ctrl+, | Preferences |
| Ctrl+Z / Ctrl+Y | Undo / redo staged edits |
| Ctrl+C / Ctrl+Shift+C | Copy selected cells / copy with a header line |
| Ctrl+V / Ctrl+Alt+V | Paste as one value / paste rows and columns |
| Tab / Shift+Tab | Move between controls; every button is reachable |

## Updates
Updates are checked when TableGrip starts (you can turn that off in Preferences).
- **When a new version exists** TableGrip opens a window like other desktop apps: *A new version of TableGrip is
  available!* with the release notes. **Install Update** downloads it (with a progress bar) and then **Install and
  Restart** installs it; your connections and settings are kept, and it asks first if you have unsaved changes.
  **Remind Me Later** closes the window, and the start screen keeps an orange notice with **What's new** and **Update**.
  **Skip This Version** hides that version until a newer one appears.
- **Several versions behind.** The notice and the window say how many updates you are behind (for example *Version 1.0.0
  is available (8 updates behind)*), and the release notes list every version you skipped, newest first. The update
  itself goes straight to the newest version.
- **Help > Check for updates** (or *Check now* in Preferences) looks once, whenever you like, and offers a skipped version
  again. It works for copies installed with the installer, not for the portable zip.
- *Look for a new version when TableGrip starts* (on by default) does the same check on every start, also available as a
  checkbox in the update window. It contacts only github.com and sends nothing about you or your data.

## Appearance
**Preferences > Appearance** has the **Look**, the colour scheme (System, Light, Dark) and two more choices:
- **Look**: *Glass* (frosted panels over a colourful backdrop) or *Classic* (the original flat look: opaque panels in
  the original colours, no backdrop art). The colour scheme and the layout work with both.
- **Glass effects**: *Auto* uses translucent panels while Windows' *Transparency effects* are on (and never in High
  Contrast), *On* always, *Off* gives opaque panels with the same layout and colours. Panels show a blurred copy of
  the colourful backdrop behind them.
- **Layout**: *Floating* (separate rounded panels with gaps) or *Docked* (flush panels, no gaps). Both layouts work with either look.

The title bar's connection name is a command bar: click it, or press **Ctrl+K**, to jump to a table, a tab or an action.
A thin line in the environment's colour along the top of the workspace shows which environment you are in.

## Where TableGrip keeps its data
`%APPDATA%\TableGrip`: `connections.json`, `settings.json`, `history.json`, `favorites.json`, `known_hosts.json`,
`window.json` and `secrets.json` (encrypted). Delete the folder to start from scratch. Set the `TABLEGRIP_DATA_DIR`
environment variable to use another folder (useful for a second, separate profile).
