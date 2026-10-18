<p align="center">
  <img src="docs/site/logo.png" alt="TableGrip logo" width="128" height="128">
</p>

<h1 align="center">TableGrip</h1>

<h3 align="center">A fast, minimalist database client for Windows</h3>

<p align="center">
  <a href="https://github.com/Apizr00/tablegrip-windows-releases/releases/latest"><img src="https://img.shields.io/badge/DOWNLOAD_%2F_RELEASES-ff9800?style=for-the-badge" alt="Download / Releases"></a>
  <a href="https://github.com/Apizr00/tablegrip-windows-releases/issues"><img src="https://img.shields.io/badge/BUGS_%2F_FEATURES-7c5cff?style=for-the-badge" alt="Bugs / Features"></a>
</p>

#### TableGrip is a native Windows application (WPF, .NET 10) for Windows 10 and 11, on x64 and ARM64

TableGrip lets you browse, query and edit a database in one small window, and shows every change as SQL before it is
written.

Supports **SQLite**, **MySQL**, **MariaDB**, **PostgreSQL**, **SQL Server**

> **Status:** version 1.10.0. All planned features are in. The installers are not code-signed yet, so Windows
> SmartScreen may warn the first time you run one. SQL Server support is
> implemented and unit-tested but has not been run against a live server yet.

### As excited as we are?

#### Check out some of our features:

- **One database per connection.** Pick an engine, name the database, and the sidebar shows only its tables and views.
- **Safe editing.** Edits are staged, shown as the exact SQL, and committed in one transaction. If one statement fails,
  nothing is saved. Undo and redo.
- **SQL editor** with highlighting, autocomplete that understands your aliases, run a statement or a script, cancel,
  history and saved queries.
- **Large tables.** Sorting, filtering and paging happen on the server; a Structure tab shows columns, keys, indexes and DDL.
- **Copy and export.** Copy cells as CSV, JSON or SQL INSERT; export to CSV, JSON, SQL and Excel. **Import** CSV,
  validated completely before a single row is written.
- **Careful with servers.** SSH tunnels (password or key, remembered host keys), TLS from "if offered" to "verify
  identity", safe mode per connection (confirm changes or read-only), Windows Integrated Authentication for SQL Server.
- **Quiet and accessible.** Light, dark or system theme, a command palette (Ctrl+K), keyboard reachable controls with
  visible focus, names for screen readers, per-monitor DPI.
- **Private.** No account, no telemetry. The update check only contacts the release page (and can be turned off).

### Learn more

- [User guide](docs/user-guide.md): connections, SSH and TLS, safe mode, editing, shortcuts
- [Changelog](CHANGELOG.md)

### Feedback

Found a bug or want a feature? Please [open an issue](https://github.com/Apizr00/tablegrip-windows-releases/issues).

TableGrip is free to use under the [MIT License](LICENSE). The source code is not public. The open-source components it is
built on are listed in [THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md).

<br>

<h3 align="center">TableGrip is free and well maintained by <a href="https://github.com/Apizr00">Hafiz Rodzli</a></h3>

<p align="center">TableGrip Team | <a href="https://github.com/Apizr00">Hafiz Rodzli</a></p>
