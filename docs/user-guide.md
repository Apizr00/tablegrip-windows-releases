# TableGrip user guide

## Connecting

1. **Create connection** (Ctrl+N), choose the database type, fill in host, port, user, password and **database name**.
   Each connection opens exactly one database.
2. **Test** checks the settings without saving. **Save** keeps it in the list; **Connect** saves and opens it.
3. SQLite and DuckDB files can also be opened directly (Ctrl+O) or dropped onto the window.
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

### DuckDB
DuckDB is an embedded analytical database kept in one file (`.duckdb`, `.ddb`, or any name: TableGrip reads the file's header).
It is chosen in the new-connection picker, or a file is opened with **File > Open database file**, dropped on the window or
given on the command line; the engine follows the file. **New...** next to the file name starts a new, empty database.
- **One program at a time.** DuckDB lets only one program write to a file, so while a connection is open the file is locked
  (close the connection to release it; a SQLite file is released after each action). A file that another program has open
  says so when you connect; *Read-only* in the connection's safe mode opens it with `access_mode=READ_ONLY`.
- **What you see.** The schemas of the file (`main` first; DuckDB's own are hidden), tables, views, sequences (under
  *Sequences*; DuckDB has no triggers or routines to list), comments, primary and unique keys, indexes, foreign keys and checks.
  Lists, structs and maps show the way DuckDB prints them (`[1, 2]`, `{'a': 1}`), a `BLOB` opens in the binary viewer, and a
  `UNION` column, which the driver cannot read, shows `(unsupported type ...)` instead of failing the result.
  `SELECT * FROM 'file.parquet'` and `read_csv(...)` work in a query tab like in any DuckDB client.
- **Editing** works as for other engines (staged, SQL shown first, one transaction). *n rows affected* is real for INSERT,
  UPDATE and DELETE.
- **Changing a table.** DuckDB does not let an `ALTER TABLE` run on a table that has an index (rename, retype, NOT NULL, drop
  column, rename table), so the review SQL drops the indexes, changes the table and creates them again, and says so. It also
  cannot add a column with NOT NULL (the SQL adds it, then sets NOT NULL), has no auto-numbered columns (give the column a
  default of `nextval('sequence')`), knows only NO ACTION and RESTRICT for foreign keys, and cannot add or drop a foreign key,
  a check or an existing primary key on an existing table: those changes are left out with a warning. To get such a table,
  make a copy with *Clone table...* from a design that has them.
- **Maintenance.** *Update statistics* (`ANALYZE`) for a table or the whole file, and *Optimize* for the whole file (`CHECKPOINT`).
  There are no users, sessions or locks to show; *Server status* shows the version, the file size, the memory in use and more.
- **Explain** works as for PostgreSQL: the plan of a query as a tree of operators with estimated rows, and **Analyze** for the real
  rows and time of a read.
- **Another file.** The sidebar shows the one file of the connection. To use a second one, attach it in a script and run the
  script in one go, for example `ATTACH 'other.duckdb' AS other (READ_ONLY); SELECT * FROM other.sales;` (a script is run on one
  connection, so the attached file is there for the next statement). Attached files are not listed in the sidebar.

### Redis
Pick **Redis** in the new-connection picker: host, port (6379), the user (empty for the `default` user or none) and password,
TLS if the server needs it, and the **database number** (`0` to `15` on a default server; empty is `0`).
- **One table per database.** The keys of the database are the rows of a table named `db0` (`db3` for database 3), with the
  columns `key`, `value`, `type` and `ttl` (seconds left; empty when the key never expires). Open it with a double-click.
  Strings show as text; hashes, lists, sets, sorted sets and streams show as JSON (a hash is an object, a list or set an array,
  a sorted set a flat array of member and score). **Database > Switch database...** lists `db0`, `db1`, ... of the server.
- **Browse, search, sort.** The filter bar, quick search and column sort work as for any table. Paging asks the server only for
  the keys on the page, in name order (the first 200,000 keys of a very large database), and reads the values of the rows
  shown. A string over 1 MB or a collection over 5,000 items shows a note instead of its value and cannot be edited here.
- **Edit.** Change the value, the ttl or the key, add a row with **+ Row** and delete rows; changes are staged and the SQL is
  shown before it is committed. A **ttl** of 60 is `EXPIRE key 60`, an empty ttl is `PERSIST`, a new key name is `RENAME`.
  A collection is written back as JSON (`{"name": "Ada"}` for a hash, `["a", "b"]` for a list or set, `["x", 1.5]` for a sorted
  set); to add one with **+ Row** give the value as JSON, and a *type* of `hash`, `list`, `set` or `zset` (empty is `string`).
  The commit is one transaction (`WATCH`, `MULTI`, `EXEC`): when another client changed a key you edited, nothing is
  applied and the message names the key; reload and try again. A key with binary content is shown as bytes and is read only.
  **Truncate** on the table runs `FLUSHDB` and asks first.
- **Commands.** In a query tab type Redis commands, one per line (`GET user:1`, `HGETALL session:abc`, `TTL user:1`,
  `CONFIG GET maxmemory`); quote a value with spaces as in `redis-cli` (`SET greeting "hello world"`). Lines that start with `#`
  are comments and Ctrl+Enter runs the line at the caret (or the selection), Ctrl+Shift+Enter every line. Replies appear as a result grid: pairs for a hash
  or `CONFIG GET`, one line per row for text like `INFO`. `SELECT n` (use the toolbar), `SUBSCRIBE`, `MONITOR`, `MULTI` and
  the blocking commands need a connection of their own and are refused with a message.
- **Safe mode** knows Redis: reads pass, `SET`, `EXPIRE`, `HSET`... count as changes, and `DEL`, `UNLINK`, `FLUSHDB` and
  `FLUSHALL` are destructive. A *Read-only* connection refuses everything but reads.
- **Not for Redis.** There is no schema, so design, clone, rename, scripts, ER diagram, migrations, schema compare, SQL dumps and
  maintenance are hidden or say so. CSV and JSON export and import of the keys work. Administration shows the clients (kill
  works), the server status (`INFO`) and the databases. Redis Cluster, Sentinel, pub/sub and modules are not covered.

### CockroachDB and TimescaleDB
Both are chosen in the same picker as the other engines and use the PostgreSQL driver, so tables, views, indexes, foreign keys,
editing, import, export, the designer and TLS work as they do for PostgreSQL. A connection URL can start with `cockroachdb://`
or `timescaledb://`.
- **TimescaleDB** is PostgreSQL with an extension, so everything PostgreSQL offers works. Its own schemas (`_timescaledb_*`,
  `timescaledb_*`) are hidden from the sidebar; hypertables are listed as the tables they look like.
- **CockroachDB** defaults to port **26257** and the database `defaultdb`, and a secure cluster needs the TLS mode set
  (*Required* or one of the *Verify* modes, with the CA file). CockroachDB has no PostgreSQL server-monitoring views, so
  *Sessions*, *Locks*, *Server status*, *Replication*, *Users*, the size of a table and **Explain** report that the engine does not
  offer them, and the only maintenance command is *Update statistics* (`ANALYZE`). These two engines were built against the
  PostgreSQL catalogs and tested without a running server, so please report anything that a real cluster refuses.

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
**production** switches safe mode from Silent to Warn by itself.

### Colour
Each connection has a colour dot on the start screen and in the connection rail. Under *Colour* pick one of the swatches
(the first one is the database type's default) or any colour of your own: type **R**, **G**, **B** (0-255) or a
`#RRGGBB` value, and press Tab or click away to apply it. Click the colour block next to *Status color* for the **colour
picker**: a wheel for hue and saturation, a brightness and a saturation slider, the hex value with copy and done buttons,
and a star that saves the colour to your own list (click a saved colour to use it, right-click to remove it).

### Safe mode
Per connection, four levels (like TablePlus):

| Level | What happens when something would change the database |
|---|---|
| **Silent** | Nothing extra: statements run at once. |
| **Warn** | Every statement that writes or changes the schema asks first; destructive ones are marked. |
| **Verify** | Like Warn, and then TableGrip checks it is you: **Windows Hello** (face, fingerprint, PIN or the Windows password), or your **TableGrip password** when Hello is not set up. If neither exists the connection name is asked for instead. |
| **Read-only** | Writes are refused. Editing is off. |

Marking a connection *production* turns Silent into Warn. Verify covers running statements, saving staged row edits (and
edits to a query result), applying a structure change, restoring a backup and applying migrations; reading never asks.
Safe mode is a guard against mistakes, not a security boundary: for a hard guarantee use a database account without
write privileges.

**Production lock.** On a connection tagged *production*, a destructive statement (`DROP`, `TRUNCATE`, `UPDATE` or
`DELETE` without a `WHERE`), committing deleted rows, and restoring a backup each ask you to **type the connection's
name** before they go on, instead of just clicking OK. Turn it off in Preferences > Safety. Read-only connections still
refuse writes.

### The TableGrip password and the app lock
**Preferences > Safety** has the pieces that Verify and the lock share:
- **Windows Hello.** When it is set up on the computer TableGrip uses it first. The line under *Safety* says whether it is.
- **TableGrip password.** *Set password...* keeps a salted hash (PBKDF2) of a password of at least four characters in
  `settings.json`; the password itself is never stored and cannot be recovered. It stands in for Windows Hello, and it is
  asked for with three tries. *Change password...* and *Remove password* ask for the current one first.
- **Lock TableGrip when it starts.** Before any window with your connections opens, TableGrip asks for Windows Hello or the
  password; *Quit* leaves without opening anything. Turning the lock on first asks you to unlock once, so you cannot set up
  a lock you cannot open. If both Hello and the password are gone, TableGrip opens unlocked rather than lock you out.

The lock keeps someone at your unlocked computer out of your saved connections. It is not encryption: someone who can edit
the files in `%APPDATA%\TableGrip` can switch it off, saved logins stay protected by Windows (DPAPI) as before, and the
command line (`tablegrip-cli`) is not locked.

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
- **The table menu.** Right-click a table or view in the sidebar: *Open in new tab*, *Open structure*, **Item overview** (rows,
  size, columns, key, indexes, foreign keys, checks and comment in one window), *Show diagram*, *Find usages*, *Copy name*,
  **Pin to top** (the table moves to a PINNED section at the top of the list and stays there for this connection),
  **Export** and **Import** (below), **New** (table, view, schema, database), **Copy Script As** (SELECT, INSERT, UPDATE, DELETE,
  CREATE or DROP straight onto the clipboard; *Generate SQL in a new tab* opens it in an editor instead), *Maintenance*,
  *Design table*, *Rename*, **Clone** (a new table with the same columns, key, plain indexes, foreign keys and checks, with
  or without the rows), **Truncate** (all rows removed; SQLite deletes them and restarts its AUTOINCREMENT counter) and *Delete*.
  Clone, Truncate and Delete show the SQL first; on a production connection the destructive ones ask for the connection name.
- **Search** every table: **Ctrl+F** puts the cursor in the search box of the open table (server side, all columns), or of a
  query result (the loaded rows, with a "12 of 250 rows" count). Ctrl+F in the sidebar filters the list of tables.
- **Filter** with conditions (column, operator, value; **+** adds a line, **-** removes it) or the quick search; filters apply
  to the whole table, and to export. The operators are listed in groups: `=` `<>` `<` `>` `<=` `>=`; `IN` / `NOT IN` (values
  separated by commas or lines; put quotes around a value that contains a comma); `IS NULL` / `IS NOT NULL`; `BETWEEN` /
  `NOT BETWEEN` (two boxes, both ends included); `LIKE` (your own pattern: `%` any text, `_` one character); and the plain
  text searches *Contains*, *Not contains*, *Has prefix* and *Has suffix*, which treat `%` and `_` as ordinary characters.
- **Edit** a cell by double-clicking or pressing F2. Changes are *staged* (coloured) until you commit. **Preview** shows
  the exact SQL; **Commit** writes it in one transaction, and nothing is saved if one statement fails.
  **Which row is changed.** By the primary key. A table without one is found by a unique key of NOT NULL columns, or,
  failing that, by all the values the row had (a note beside the edit bar says so); an edit that would then touch more than
  one row is refused and nothing is saved. Views stay read-only. **Someone else changed it?** When you commit, the values
  you overwrite must still be what you read (text, integers, decimals, booleans and GUIDs; not dates, times or floating-point numbers), or the
  commit is refused and nothing is written; turn this off in Preferences.
- **Copy** the selected cells with Ctrl+C (tab-separated, full values). Right-click > **Copy as** adds a header line
  (**Ctrl+Shift+C**), CSV, JSON, or SQL INSERT statements for the selected rows and columns.
- **Edit the result of a query.** When a query is a plain `SELECT` of columns (or `*`) from one table, with at most a `WHERE`,
  `ORDER BY` and `LIMIT`, its result is editable like the table itself: an edit bar appears over the grid, **Ctrl+S**
  commits (the SQL is shown first on a safe-mode connection) and the query runs again to show what is stored. A join,
  `GROUP BY`, `DISTINCT`, `UNION`, a computed or renamed column, a CTE or a result that leaves out the key is not editable,
  because an edit could then change the wrong row.
- **Pickers.** A cell can be filled from a list instead of typed. A column with a fixed set of values (a MySQL `ENUM`, a
  PostgreSQL enum type, or a `CHECK` that lists the allowed values) opens a list when you edit it; a true / false column
  opens **TRUE / FALSE / NULL** (and **Space** flips it); **F4** (or Alt+Down, or right-click > *Pick a value*) opens the
  picker of a cell: the **rows of the referenced table** for a foreign key of one column (search them, choose one, its key
  goes into the cell), a **calendar and time** for date, time and timestamp columns, or the list. Double-click or F2 still
  types.
- **Copy as** also offers **Markdown table**, **SQL UPDATE (by key)** (one statement per selected row, setting the selected
  columns and finding the row by its key) and **WHERE ... IN list** (the distinct values of the one selected column,
  ready to paste into a query; NULLs become `OR col IS NULL`).
- **Row details.** The Details panel is editable when the table is: leaving a box (Tab, a click, or Ctrl+Enter) stages
  the value like a cell edit, a bar marks fields that changed, **NULL** sets a field to NULL, **Format** lays JSON out, Esc
  puts the stored value back, and **Alt+Up / Alt+Down** move the grid to the previous / next row without leaving the panel.
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
- **Structure** shows columns, keys, indexes and the CREATE statement. **Edit structure** turns the columns, indexes, foreign keys
  and checks into a grid (add, rename, retype, remove, move); nothing runs until **Review SQL** (Ctrl+S) shows the `ALTER`
  statements and you run them, and **Discard changes** puts everything back. Row edits that are not saved must be committed or
  discarded first. The table opens again on its new structure afterwards.
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
| Add the next match as a caret / select every match | Ctrl+D / Ctrl+Shift+L |
| Add a caret above / below, or where you click | Ctrl+Alt+Up, Ctrl+Alt+Down, Ctrl+Alt+click |

**Format, find and fold.** **Format** (Ctrl+Shift+F) lays out the selection, or all the SQL, with one clause per line, one
select item per line and keywords in capitals; nothing but white space and keyword case changes, and Ctrl+Z undoes it.
**Ctrl+H** opens find and replace above the editor (Match case, Whole word, Regex with `$1` groups; *Replace all* is one
undo step). Ctrl+F still opens the editor's own search. The margin has fold markers for multi-line statements, bracketed
groups, CASE and BEGIN blocks and block comments. **Split** (Ctrl+\) shows a second view of the same text below the first.
**Ctrl+wheel** zooms the text of the tab and **Ctrl+0** resets it. Hold **Alt** and drag to select a block of columns
(box selection) and type to change every line of it at once.

**Several carets.** **Ctrl+D** selects the word at the caret and then adds the next match of the selected text (wrapping round to
the start), **Ctrl+Shift+L** selects every match, **Ctrl+Alt+Up / Down** adds a caret on the line above or below in the same
column, and **Ctrl+Alt+click** adds one where you click. Typing, Backspace, Delete, Enter, Tab, the arrow keys (also with
Shift), Home / End, Ctrl+C, Ctrl+X and Ctrl+V then act at every caret, and the whole edit is one undo step. Paste puts one
line at each caret when the clipboard holds as many lines as there are carets, the same text at each otherwise. **Esc**, a
click, undo or any command collapses them back to the last caret.

**Vim mode.** **Preferences > Editor > Vim mode** turns the SQL editor into a modal editor; a badge in the corner shows the
mode and the keys typed so far. The editor starts in normal mode (press **i** to type, **Esc** to come back).
- *Moving:* `h j k l` (and the arrows), `w W b B e E`, `0 ^ $`, `gg G` (with a line number: `12G`), `f F t T` with `;` `,`,
  `%` (matching bracket), `{ }` (paragraphs); a count in front repeats (`3w`, `5j`).
- *Editing:* `x X D C s S r J ~ p P`, `i a I A o O`, `u` and **Ctrl+R** (undo and redo), `.` repeats the last change
  (with the text you typed in it). `d c y > <` and `gu gU g~` take a motion or a text object (`dw`, `c$`, `y2j`, `>>`,
  `dd`, `cc`, `yy`); text objects are `iw aw`, `i" a"`, `i' a'`, ``i` a` ``, `i( a( ib`, `i{ a{ iB`, `i[ a[`, `i< a<`
  (so `ci'` and `di(` work as in Vim).
- *Selecting:* `v` (characters) and `V` (lines), motions and text objects extend the selection, then `d y c > < u U ~ J r p`;
  `o` swaps the ends.
- *Clipboard:* yanks and deletes are also put on the Windows clipboard, and `p` pastes what you copied elsewhere.
- *Not included:* `:` commands, `/` search (Ctrl+F still works), marks, macros, registers other than the unnamed one and block
  selection. Ctrl shortcuts that Vim has no use for (Ctrl+Enter to run, Ctrl+S, Ctrl+C...) keep working in every mode.

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

**Saved queries.** Right-click a saved query for *Move to folder...* (the list then groups the queries by folder) and *Make a
snippet with a keyword...* (type the keyword in any query and press Tab to insert it). The filter box above the list remembers
its text: the tables, queries and history lists each keep their own, and the tables filter comes back with the tabs next time.

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
  times (DuckDB too); it is not offered for a statement that changes data. DuckDB shows its operators with the tables, joins and
  filters, a scan of a big table without a filter marked as the hot spot, and with Analyze each step's real rows and time. MySQL and MariaDB show the tabular plan, SQLite its query
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
- **Export** (Ctrl+Shift+E, or right-click a table > Export > *Export tables...*): one window with the tables on the left
  (tick the ones you want; the ones selected in the sidebar start ticked, with a search box and All / None) and the format on
  the right: **CSV**, **JSON**, **SQL** (INSERT statements) or **Excel**. CSV options: *Convert NULL to EMPTY* (otherwise the
  word NULL is written), *Convert line break to space*, *Put field names in the first row*, the byte order mark, the
  **delimiter**, **quote** (if needed, always, never), **line break** (`\r\n` or `\n`) and the **decimal** separator (dot or
  comma; **Customize** shows these last four). **Export...** opens the *Save As* window for one file (the *File name* box
  gives its name; several tables on the SQL tab are one `.sql` file named after the database) or asks for a folder when
  every table goes into its own file. *Compress the file using Gzip* writes `.gz` files (not for Excel).
  On the **SQL** tab each ticked table has **Structure** (`CREATE`), **Drop** (`DROP ... IF EXISTS` first) and **Data**
  (`INSERT`s) boxes; the boxes in the header tick or clear a column for the ticked tables shown. Tables come in foreign-key
  order and views last (their definition only). While it runs the window is a small card with the table, *rows done / rows in
  all the tables* and a pill of pixels (a band of pixels sweeps while the work is still being measured), with Cancel and
  **Export in Background**; afterwards a **Success** message
  offers *Open containing folder* and *Don't show this again* (Preferences turns it back on). *Export this table with
  column selection...* opens the window of a single table where you can leave columns out; from a table tab, **Export**
  there can also write only the rows loaded in the grid.
- **Import CSV or JSON** (Ctrl+Shift+I): the format is detected from the file. For CSV the delimiter is detected and you can
  choose the text encoding; for JSON the file is an array of objects or one object per line, and the property names are the
  columns. Columns are matched by name and can be remapped; beside each one the window says what the values look like. The
  file is validated *before* anything is written, and an error leaves the table untouched.
- **If the key exists.** Choose what happens to a row whose primary key is already in the table: *Stop and change nothing*
  (default), *Skip the existing row*, or *Update the existing row* (only the columns in the file change). Skip and update
  need the table's primary key columns to be mapped; update needs a primary key.
- **Import from the table menu.** Right-click a table > **Import** > *From CSV...*, *From JSON...* or *From SQL dump...*: the file
  is picked first, then the import window opens on it (a SQL dump starts the restore, after you confirm).
- **Progress.** Export, import, backup and restore share one panel: a percent bar, rows (or statements) done of the total, the
  speed, the time elapsed and the time left, and for several tables a second bar for the whole job.
- **Import SQL dump.** *File > Import SQL dump (restore)...* runs a `.sql` or `.sql.gz` script on the connection as one
  transaction. Backups can be written compressed: tick *Compress the file* in the backup panel.
- **Run in background.** While an export, import, backup or restore runs, *Run in background* (*Export in Background* for tables)
  closes its window. The **bell** in the title bar lists it with a pill of pixels, the percentage and a Stop button; when the job
  ends the list opens for a few seconds (and the window flashes in the taskbar if another program is in front). The entry stays,
  with *Completed* and **Open Folder** for files, until you dismiss it with the x or press **Clear**.

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
  `--yes-production`. The command line cannot show Windows Hello or ask for the TableGrip password, so on a **Verify**
  connection `--write` is the confirmation, as on a Warn one; the app lock does not apply to it either.
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
| F4, Alt+Down | Open the picker of the cell (list, calendar, referenced rows) |
| Alt+Up, Alt+Down (in Details) | Previous / next row |
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
- **Theme file**: a `.json` file that lays your colours over the light or dark set (and sets the colours of the SQL
  syntax). *Export the theme in use...* writes every colour of what you see now, so you can change a few values in a text editor
  and *Import...* the file again (the themes are kept in the `themes` folder, *Open folder* shows it). A file looks like
  `{ "name": "Sunset", "base": "light", "colors": { "Accent": "#D9480F", "Window": "#F4E6DA" }, "syntax": { "keyword": "#D9480F" } }`:
  `base` is `light` or `dark` and decides which one the theme starts from (it then ignores the colour scheme choice),
  colours are `#RRGGBB` or `#AARRGGBB`, names are the tokens of the app without the `Brush.` prefix, and the syntax roles are
  `keyword`, `type`, `function`, `string`, `number`, `comment` and `quoted`. Colours left out keep their built-in value, names the app does not know
  are ignored, and an entry that is not a colour is skipped with a note. With transparency off the app uses the `...Opaque`
  twin of a translucent colour (`Window` becomes `WindowOpaque`), so a theme that changes one should change both.
- **Fonts**: the editor font and size, and the **data grid font** and size (the grid font list shows the installed fonts of a
  short list; the first entry is the app font).

The title bar's connection name is a command bar: click it, or press **Ctrl+K**, to jump to a table, a tab or an action.
A thin line in the environment's colour along the top of the workspace shows which environment you are in.

## Where TableGrip keeps its data
`%APPDATA%\TableGrip`: `connections.json`, `settings.json` (with the hash of the TableGrip password, if you set one),
`history.json`, `favorites.json`, `known_hosts.json`, `window.json`, the `themes` folder (your theme files) and
`secrets.json` (encrypted). Delete the folder to start from scratch. Set the `TABLEGRIP_DATA_DIR`
environment variable to use another folder (useful for a second, separate profile).
