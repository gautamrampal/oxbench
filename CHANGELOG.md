# Changelog

One feature per release; see [docs/ROADMAP.md](docs/ROADMAP.md) for what comes next.

## 0.8.2 — Redis Cluster, GridFS, shell transactions and FileZilla-style SFTP (2026-10-28)

The rest of the 0.3–0.5 gaps (planned as 0.6.1, renumbered so versions only go up): Oxbench now works with a whole
Redis Cluster, browses and edits MongoDB GridFS buckets, and runs transactions in the MongoDB shell. The SFTP browser
gets FileZilla-style transfers.

### Added — Redis Cluster
- **Cluster mode is detected by itself:** connect to any node and Oxbench works with the whole cluster. Each key
  goes to the node serving its slot, MOVED / ASK redirects are followed, and every node is dialed through your SSH
  route and TLS settings. The server shows "Redis Cluster: N primaries, M replicas".
- The **Key Browser**, delete by pattern, FLUSHDB, export and import cover every primary. Key counts are summed and
  only db0 is offered.
- **Console:** commands with a key run on that key's node; a node picker chooses where keyless commands (DBSIZE,
  KEYS, INFO, MONITOR…) run, and each answer names its node. Transactions stay on one node, cross-slot keys get a
  hash-tag hint, and sharded pub/sub (SSUBSCRIBE / SPUBLISH) works.
- **Server Info:** a new **Cluster** tab (state, slot coverage, a slot bar per primary, nodes with role, slots, link
  and flags, slots being migrated) and a node picker for INFO, Slowlog, Clients and Config. Config, ACL and slow-log
  changes can apply to every node.
- **Script Editor:** runs go to their first key's node (keyless runs to a picked node); script cache and function
  library changes reach every primary. Keyspace events stream from all primaries.

### Added — MongoDB
- **GridFS:** each database has a GridFS folder of buckets. A bucket tab lists its files (filter, sort, paging),
  shows their properties and metadata, and previews images, JSON and text. Download, rename and delete files, or drop
  a bucket, with confirmations. A bucket's `.files` / `.chunks` collections are marked "GridFS".
- **GridFS upload:** upload files from disk into a bucket (or a new one) with a name, content type and metadata.
  Uploads and downloads show progress and can be cancelled; a cancelled upload leaves no chunks behind.
- **Transactions in the shell:** `session.startTransaction()`, `commitTransaction()`, `abortTransaction()` and
  `endSession()` (the mongosh `db.getMongo().startSession()` forms work too). Reads and writes, `bulkWrite`
  included, run in the transaction, so a bulkWrite becomes all or nothing.
- The shell shows an open transaction with its age and writes, Commit and Abort buttons, and warns before the
  server's transaction time limit (60 s by default). Closing the tab or disconnecting aborts it.
- Confirmations for writes inside a transaction are asked at commit, naming what each write did. Errors show their
  TransientTransactionError / UnknownTransactionCommitResult label and what to do next; a standalone server says
  transactions need a replica set.

### Added — SFTP, FileZilla-style
- **Target exists prompt** for every clashing file, including those inside folders: Overwrite, Overwrite if source
  is newer, Resume (continue a partial file), Rename (with a free `name (1).ext` suggested) or Skip, with "Always use
  this action" for the rest of the transfer. Cancelling it stops the whole batch.
- **Transfer queue** with Queued / Failed / Successful lists, a configurable number of simultaneous transfers
  (default 2), cancel, retry and clear.
- **Speed limits** for uploads and downloads (KB/s, shared by every transfer), and **preserved modification times**
  (on by default) so Compare and "Overwrite if newer" stay meaningful.
- **Drop files from Explorer** onto the host pane (or onto one of its folders) to upload them.
- **Sortable columns** (name, size, modified, permissions; folders stay first) and a **filter** box per pane
  (Ctrl+F).
- **Directory comparison**: marks files that exist on one side only, differ in size, or are newer.
- **Synchronized browsing**: opening a folder (or going up) on one side does the same on the other.
- **Bookmarks** of local / remote folder pairs per host, and each host reopens at the folders last used.

### Changed
- SFTP: dragging between the panes no longer relies on HTML5 drag-and-drop, which the desktop webview keeps for file
  drops from Windows; rows can also be dropped onto a folder in the other pane.
- MongoDB shell: `bulkWrite` inside a transaction is atomic; outside one it still runs per operation.

### Not yet included
- Redis Cluster: reads from replicas (READONLY), a merged MONITOR across nodes, and following primaries added while
  a keyspace watch runs. A cluster behind SSH or TLS is built for but was tested only directly.
- MongoDB: `session.withTransaction()` and other shell variables; catalog changes inside a transaction (refused,
  stricter than the server); one session per shell tab; the upload dialog adds files one at a time.

## 0.8.1 — Dockerfile Builder (2026-10-24)

Point Oxbench at a project folder and get a production-ready Dockerfile: it detects the stack, asks a few guided
questions, previews the Dockerfile, .dockerignore and docker-compose.yml as you answer, and then builds and runs the
image.

### Added
- **Dockerfile Builder** (Docker sidebar "New Dockerfile…", palette "Docker: Create Dockerfile for a project…",
  Images "Build from a project…"): pick a folder, see the detected stack and why, then base image, dependencies and
  build, run, extras, and save / build / run, with a live Dockerfile | .dockerignore | docker-compose.yml preview
  that can be edited by hand. Saving asks before overwriting existing files.
- Detection and templates for Node (Next.js, Vite, CRA, Angular, Astro, Nest, Express, Fastify, Nuxt, SvelteKit;
  npm, pnpm, Yarn, Bun), Python (Django, FastAPI, Flask; pip, Poetry, uv, Pipenv), Go, Rust, Java (Maven, Gradle,
  Spring Boot), .NET (ASP.NET Core, console), PHP (Laravel with its Vite assets, Symfony, plain; Apache or
  PHP-FPM) and static sites: pinned major tags, the lockfile copied first, multi-stage builds, a non-root user,
  CMD in exec form, a HEALTHCHECK.
- .dockerignore defaults per stack; an optional docker-compose.yml with a PostgreSQL, MySQL, MariaDB, Redis or
  MongoDB service, a healthcheck, `depends_on: service_healthy` and the connection variables wired in.
- Builds through the Docker CLI (BuildKit) on the connection's engine: streamed log, per-stage step list, Cancel,
  image id and size; Run starts the image with ports on 127.0.0.1 and opens its container tab.
- Answers remembered per folder (secret values are never stored); Dockerfile syntax highlighting in the editor.

### Not yet included
- **0.8.2:** Redis Cluster; MongoDB GridFS and shell transactions.
- Building without the Docker CLI (Engine API `/build`), remote engines over SSH, registry push.
- The language base images are not built in our own tests (templates are checked as text and the build path with a
  small image).
- Compose with more than one database; published database ports, volumes and networks in the Run form.
- Process-tree cancel verified on Windows only so far.

## 0.8.0 — Docker workspace (2026-10-24)

Manage the Docker engine on this computer, or one on another host over TCP / TLS, next to your databases: containers with
logs, stats and shells, images, volumes and networks, and databases launched in a container in one step.

### Added
- **Docker workspace** (rail, Ctrl+Shift+4):
  - The local engine is found by itself (DOCKER_HOST, the current docker context, Docker Desktop's pipe or the Docker
    socket); saved TCP engines, with TLS (CA, client certificate and key) or without.
  - "Docker is not running" with Retry, an automatic re-check, and Start Docker Desktop on Windows; the status bar shows
    the engine version.
  - Lists follow the engine's event stream.
- **Containers:**
  - Grouped by compose project (running / total, start or stop a whole project); state, health, image, status and
    published ports; filter and All / Running / Stopped; multi-select.
  - Start, Stop, Restart, Pause, Resume, Kill (choose the signal), Remove (optionally with its anonymous volumes) and
    Rename; destructive actions confirm what happens.
- **Container tab:**
  - **Overview:** state, configuration, resources, ports, mounts, networks, environment (secret-looking values and URL or
    command-line passwords masked) and labels.
  - **Logs:** follow, tail, since, timestamps, stdout / stderr filter, search with highlights, wrap, ANSI colours; the
    last 50,000 lines are kept; copy and download.
  - **Stats:** live CPU, memory (with the limit), network and block I/O charts, and PIDs.
  - **Processes:** filter, auto refresh.
  - **Shell:** bash or sh, several shells per container, a custom command with user and working directory; pop out to
    its own tab.
  - **Inspect:** the raw JSON with find.
- **Images:**
  - One row per tag; pull with per-layer progress and cancel; remove or untag (force on request); tag; inspect and
    history; prune dangling images.
- **Volumes and networks:**
  - Create, inspect, remove and prune (confirmed); networks with driver, internal, subnet and gateway; connect and
    disconnect containers.
- **Quick launch:**
  - PostgreSQL, MySQL, MariaDB, Redis and MongoDB in a container: generated password, a free host port on 127.0.0.1,
    optional named data volume; pulls the image when missing.
  - Waits until the database answers, then registers it in Databases (group Docker, password in the OS keychain) and
    offers Open in Databases; a failed or stopped launch offers to remove what it created.
- **Disk usage:** images, containers, local volumes and build cache, with what is reclaimable.
- **Palette:** Quick launch database…, Pull image…, Images, Volumes, Networks, Connections…, Open container… (searches
  containers).

### Changed
- New crate `devkit-docker` (feature `docker`): a thin Docker Engine API client over the named pipe, unix socket, TCP
  or TLS (no Docker CLI needed).
- Shortcuts help: Ctrl+Shift+1 … 4 switch workspaces.

### Not yet included
- **0.6.1:** Redis Cluster; MongoDB GridFS and shell transactions.
- **Docker 0.8.x:** engines over SSH (`ssh://` endpoints report "not supported yet"); the Dockerfile Builder
  (0.8.1).
- **Out of scope for 0.8:** compose up / down and editing compose files, building images (the Dockerfile Builder, 0.8.1), registry login and push,
  Swarm and Kubernetes, Windows containers' stats, browsing container files.
- **Engine:** private registries (pulls send no credentials); on Windows, TLS client keys must be unencrypted RSA keys.
- **Shells:** closing a shell sends it SIGHUP on a best-effort basis (needs sh, tr, grep and /proc in the container).
- **Logs and stats:** wrapping cuts at a fixed column; 16 ANSI colours; the first CPU sample reads 0 %; stats show one
  container per tab.
- **Assets:** "Used by" is a count for images; no per-volume sizes; network create has no IPv6, driver options or labels,
  and connect no alias or IP; volume prune removes anonymous volumes only on Docker Engine 23 and later.
- **Quick launch:** the free-port suggestion sees only ports published by running containers; an existing data volume
  keeps the password it was created with; on a remote engine the port is published on all of its interfaces.

## 0.7.0 — Schema tools: ER diagram and schema diff with migration SQL (2026-10-24)

See a schema as a diagram, and turn one database's schema into another's with reviewed, ordered migration SQL —
for PostgreSQL, MySQL / MariaDB and SQLite.

### Added
- **ER diagram** (Object Explorer database or schema menu, Tools menu, palette):
  - Tables as cards (row estimate, PK / FK / UQ markers, types, nullability), views and materialized views
    dashed, tables of other schemas as header cards; foreign keys as crow's-foot edges with optional circles
    and self-reference loops.
  - Automatic layered layout for up to ~300 tables (components in rows, isolated tables in a grid), pan, zoom,
    Fit and a draggable minimap; moved cards are remembered per diagram, Re-layout resets them.
  - Find and filter tables and columns, column modes All / Keys / Names, Views on/off, highlight of a card's
    relationships, card menu (View Data, Query Tool, Reveal, Copy name), 12 palette commands and shortcuts.
  - Export to SVG (theme colours written in) and PNG (up to 8192 px).
- **Schema diff** (same engine on both sides; servers may differ; MySQL against MariaDB with a warning):
  - Source / target pickers with Swap, options (ignore comments, whitespace, collation; compare table
    options), Cancel.
  - Results: counts and filters, object-kind chips, destructive-only filter, search, a virtualized tree of
    changes with checkboxes, a side-by-side DDL diff with word highlights and F7 navigation.
  - Migration SQL for the checked changes in a dependency-safe order, with warnings: Copy, Save .sql, Open in
    Query Tool, Apply (danger confirmation; type the database name when anything is destructive; compares
    again afterwards). PostgreSQL and SQLite scripts run in one transaction (new enum labels are committed
    first); SQLite uses the 12-step table rebuild; MySQL scripts say they are not transactional.

### Changed
- `run_sql` rolls back a failed script on the shared connection (PostgreSQL and SQLite; SQLite also restores
  foreign_keys and legacy_alter_table).
- New app command `write_binary_file` for binary exports.

### Not yet included
- **0.6.1:** Redis Cluster; MongoDB GridFS and shell transactions.
- **All engines:** renames show as a drop plus an add (except a PostgreSQL primary key rename); no cross-engine
  diffs; no ERD editing; grants and owners are not compared.
- **PostgreSQL:** 12 or later. Partitions, inheritance, foreign tables, exclusion constraints, matview indexes,
  trigger enabled state, NOT VALID / NO INHERIT, deferrable PK / unique, NULLS NOT DISTINCT, comments on keys /
  types / sequences and extension objects are not compared. Enum labels cannot be removed or reordered. Domain,
  composite and range changes and routines with a new return type are dropped and recreated (fails while in
  use). Aggregates are compared by signature only and never created. Views of a table with a dropped column or
  type change are recreated (recreated materialized views lose their indexes). Changing a generated
  expression in place needs PostgreSQL 17 (stored) or 18 (virtual).
- **MySQL / MariaDB:** ON UPDATE is compared as part of the default; INVISIBLE columns and indexes become
  visible; view ALGORITHM / SQL SECURITY / DEFINER, trigger FOLLOWS / PRECEDES and NOT ENFORCED checks are not
  kept; events and partitions are not compared; changed routines are dropped and recreated; existing rows do
  not get a new column's expression default while binary logging is on; a failed script leaves the statements
  before it applied.
- **SQLite:** a rebuild loses explicit view column lists, PK / FK constraint names, ON CONFLICT clauses and PK
  DESC; AUTOINCREMENT restarts from max(rowid); virtual tables are not compared; a new NOT NULL column without
  a default fails on a table with rows; foreign keys are on after the script whatever they were before.
- **UI:** Apply cannot be cancelled; inside a changed block the DDL view pairs lines in order; very different
  large DDL above the diff's cost cap shows as one replaced block; a moved ERD card keeps its absolute position
  after a column-mode or schema change until Re-layout; no pinch zoom; the minimap hides below 560 px.

## 0.6.0 — Engine parity: MySQL / SQLite tools, Redis and MongoDB administration, SSH keys and X11 (2026-10-23)

The gaps of 0.3–0.5 close: MySQL / MariaDB and SQLite get the tools PostgreSQL had, Redis and MongoDB get
their administration screens, and SSH generates every common key type and forwards X11.

### Added
- **MySQL / MariaDB and SQLite parity:**
  - **Dashboard.** MySQL / MariaDB get the live dashboard: sessions, TPS, rows, buffer pool, locks,
    database sizes and the Configuration tab (copy as SET GLOBAL / SET PERSIST). Cancel is KILL QUERY,
    Terminate is KILL, and each section that the account may not read shows its own error. SQLite gets a
    static file overview: size, WAL, pages, free space, settings, object counts, table sizes (dbstat), and
    a quick or full integrity check with the foreign-key check.
  - **Search Objects** for MySQL (one database or all at once) and SQLite. Hits reveal in the tree.
  - **Create dialogs** with a live SQL tab. MySQL Create Database (character set, collation) and Create
    Table (engine, UNSIGNED, AUTO_INCREMENT, defaults, comments). SQLite Create Table (affinity types,
    AUTOINCREMENT, WITHOUT ROWID, STRICT on 3.37+).
  - **MySQL users and privileges.** A Users folder (MariaDB roles and account locks included). Properties
    from SHOW CREATE USER / SHOW GRANTS with every hash masked. Create / Alter User (plugin, lock, expiry,
    resource limits, rename). A Privileges editor at global, database and table level with a GRANT / REVOKE
    preview. Drop User asks you to type the account. Changes, revokes and drops are confirmed in the app's
    Rust core.
  - Menus and toolbars ask each engine what it supports: MySQL has no schemas, SQLite no Create Database.
- **Redis:**
  - **CONFIG SET** in Server Info → Config: edit a row in place, optionally rewrite the config file. The
    server's refusal shows on the row, and masked passwords are never written back.
  - **ACL editor:** users, an editor for rules, passwords (never shown), keys, channels and selectors on
    7+, a redacted ACL SETUSER preview, ACL DELUSER and the ACL log.
  - **Keyspace events tab** per database, with a guarded "Enable (KEA)", key and event filters, and a
    virtualized stream with drop counters.
  - **Script Editor:** Lua with highlighting, typed KEYS / ARGV, EVAL / EVALSHA / FCALL and their _RO forms
    on 7+, Stop, the script cache, Functions (list, load, delete, dump, restore), and saved scripts per
    server.
  - **Export / import of keys:** JSON Lines (binary-safe) or a redis-cli command file, optionally DUMP /
    RESTORE, with progress and cancel. On import, skip or replace (replace asks first); TTLs are kept.
  - **Sentinel:** "Connect through Sentinel" (Sentinel addresses, master name, Sentinel ACL user, password
    in the keychain, TLS). The app follows a failover (a lost connection or READONLY asks the Sentinels
    again), shows a "via Sentinel" chip and a Sentinel panel. Connecting straight to a Sentinel explains
    the option.
- **MongoDB:**
  - **bulkWrite** in the shell with mongosh's result. A deleteMany / updateMany of everything inside it asks
    you to type the collection.
  - **Import** of JSON Lines, JSON arrays and CSV (type inference, per-column overrides, nested a.b
    headers), with a preview of 20 documents, progress, cancel and per-row errors.
  - **Schema analysis** on a sample: paths, presence, type mix, min / max, samples, mixed-type flags,
    Export JSON.
  - **Change streams:** a Watch tab per collection and per database, with pipeline, full documents,
    before-images, resume tokens, pause, and a clear "needs a replica set" state.
  - **Users and roles:** Users / Roles folders and a database tab, a user editor (password, mechanisms,
    roles across databases, custom data) and a role editor (privileges, inherited roles). Drops always ask.
- **SSH:**
  - **RSA** (2048 / 3072 / 4096) and **ECDSA** (P-256 / P-384 / P-521) key generation.
  - **X11 forwarding** per host (trusted, like ssh -Y; MIT-MAGIC-COOKIE-1 with the real cookie from
    Xauthority). `ForwardX11 yes` is imported from ~/.ssh/config, and problems show as notices in the
    terminal.

### Changed
- The Dashboard tab and Cancel / Terminate work per engine. The welcome page opens MySQL and SQLite
  dashboards.
- Encrypted keys imported from ssh-config are registered with the right type, and a generated key with a
  passphrase keeps its comment.

### Not yet included
- **0.6.1:** Redis Cluster; MongoDB GridFS and shell transactions.
- **MySQL / SQLite:**
  - Column- and routine-level privileges, roles, REQUIRE / TLS and proxy grants are not editable.
  - Create Table has no indexes, foreign keys or CHECK constraints.
  - Variables are copied as SET statements, not edited in place.
  - SQLite has no attached databases, and its integrity check blocks browsing while it runs.
  - The MySQL transactions tile counts Handler_commit / Handler_rollback.
  - MariaDB lists InnoDB locks only while another transaction waits for them, and no metadata locks.
- **Redis:**
  - The Sentinel password must be saved, and only the configured Sentinels are asked.
  - Consoles and streams opened before a failover close.
  - A renamed CONFIG is not supported.
  - Saving an ACL user rewrites the whole user.
  - Export is not a snapshot: stream consumer groups and hash field TTLs are not exported, and TTLs are
    relative.
  - Command-file imports run data writes only.
- **MongoDB:**
  - bulkWrite is not atomic and takes no writeConcern.
  - Import only inserts.
  - Watches need a replica set; views and time-series collections cannot be watched.
  - No editor for authentication restrictions.
- **SSH:**
  - No untrusted (SECURITY-extension) X11 mode, and only MIT-MAGIC-COOKIE-1.
  - Unix-socket displays are untested on Windows.

## 0.5.0 — Redis and MongoDB; the app is now Oxbench (2026-10-23)

Two non-SQL engines join the Databases workspace. Register them like any other server (engine picker,
SSH tab, TLS); the Object Explorer shows their databases, and Ctrl+N opens the Redis console or the
MongoDB shell instead of a Query Tool.

### Added
- **Redis and Valkey** (standalone servers or primaries):
  - **Explorer:** every database with its key count, keys with an expiry and average TTL; empty databases
    dimmed. Double-click a database to open its Key Browser.
  - **Key Browser:** incremental SCAN that never blocks the server (Stop / Continue), glob patterns or plain
    text (an exact lookup plus a contains scan), type filter, list or namespace tree on a configurable
    separator, multi-select, rename, TTL, copy to another database, delete, and "Delete keys matching…"
    with a dry run, a typed confirmation and progress.
  - **Viewers and editors** for strings (Text / JSON / Hex / Base64), hashes (with field TTLs on 7.4+),
    lists, sets, sorted sets (score edits, ZINCRBY, reverse order), streams (XADD form, XDEL, consumer
    groups) and RedisJSON. Saves are compare-and-swap: when the value changed on the server you get
    Reload or Overwrite instead of a silent lost update. Large keys load their first page on request.
  - **Console:** redis-cli style replies over RESP3 (RESP2 fallback), command help and completion
    (including key names), history with passwords redacted, MULTI/EXEC, SELECT, Cancel (CLIENT UNBLOCK,
    then Kill script), a pub/sub pane and a MONITOR pane with filters and drop counters.
  - **Server Info:** live overview tiles (ops/s, memory, clients, hit ratio, expired/evicted, network),
    searchable INFO sections, slowlog, clients (kill), read-only configuration.
  - TLS (encrypt only, or verify against the Windows store or a CA file), ACL users, SSH tunnels.
- **MongoDB** (standalone servers, replica sets, and `mongodb+srv://` services such as Atlas):
  - **Server dialog:** host fields or a connection string. A password in a pasted URI is moved to the OS
    keychain; `servers.json` keeps the URI without it. Auth database and mechanism (SCRAM, PLAIN, X.509),
    replica set, direct connection, TLS with a CA file and a client certificate.
  - **Explorer:** databases (system databases dimmed), collections, views and time-series collections,
    and indexes with unique, TTL and "unused" badges. Menus for create, rename, drop, delete all
    documents, insert, create index and export.
  - **Collection tab — Documents:** filter, projection and sort with live validation and error marks,
    completion of field paths, operators and constructors; List, Table and JSON views; counts that say
    when they are estimates; Load more up to 20,000 documents.
  - **Document editor:** typed mongosh-style text (`ObjectId(…)`, `ISODate(…)`, `NumberLong(…)`,
    `NumberDecimal(…)` …) so types survive a round trip; compare-and-swap saves with Reload / Overwrite on a
    conflict; Clone, Insert (one document or an array), Delete; table cells edited in place with their type
    kept.
  - **Aggregation:** stage cards with per-stage previews on a 1,000-document sample (never for `$out` /
    `$merge`), reorder, enable/disable, a text mode, saved pipelines.
  - **Indexes:** usage and size, a Create Index dialog with a live `createIndex(…)` preview, guarded drops.
  - **Explain** for queries and pipelines: COLLSCAN warning, index use, examined vs returned, stage chain.
  - **Shell:** a mongosh-style subset parsed in Rust (no JavaScript engine): `use`, `show`, `it`, `find` and
    its chain, `aggregate`, every CRUD and index method, `runCommand`, and more. Results as lists, tables
    or JSON; errors marked at their position; history with passwords redacted; Cancel stops the operation
    on the server too.
  - **Export** of a query to JSON Lines, JSON (both lossless Extended JSON) or CSV, written by the app.
  - SSH tunnels through a saved host, replica-set members included.
- **Safety modes** for Redis and MongoDB servers: confirm dangerous operations (default), confirm every
  write, or read-only. FLUSHDB, KEYS, `dropDatabase()`, `deleteMany({})` and similar need a confirmation
  that the app's Rust core checks, so the UI cannot skip it; the most destructive ones ask you to type the
  name.

### Changed
- **The app is now called Oxbench** (it was PgAdmin RS): the executable is `oxbench.exe`. Saved servers,
  passwords, SSH hosts, collections and preferences carry over unchanged.
- **No Windows title bar:** the window is frameless, with minimize, maximize and close in the app's own
  title bar. Drag any empty part of it to move the window; double-click to maximize.
- **Windows installers:** besides the standalone `oxbench.exe`, a setup program (NSIS, installs for the
  current user, adds Start-menu and desktop shortcuts, installs the WebView2 runtime when it is missing)
  and an MSI package.
- The About dialog names the author (Gautam Rampal) with email and GitHub links, which copy to the clipboard.
- The Dashboard tab explains that Redis and MongoDB have no dashboard and offers their Console, Server
  Info or Shell.
- The welcome page shows a SQLite server's file path instead of "@:0".
- Buttons inside empty-state panels no longer show oversized icons.

### Not yet included
- **Redis:** Cluster and Sentinel (both detected and explained), client certificates, CONFIG SET and ACL
  editing, a Lua editor, keyspace notifications, import and export, editors for module types other than
  RedisJSON.
- **MongoDB:** GridFS, change streams, transactions and `bulkWrite` in the shell, full JavaScript
  (variables, functions, loops), users and roles, schema analysis, JSON/CSV import, Kerberos / AWS / OIDC
  sign-in, wire compression, the Windows certificate store for TLS.
- The shell types numbers on purpose differently from mongosh: an integer that fits 32 bits is Int32, a
  larger one Int64, anything with a decimal point or exponent a Double.

## 0.4.0 — MySQL / MariaDB, SQLite and SSH tunnels for databases (2026-10-22)

The Databases workspace now speaks three engines. Pick one when you register a server; the Object
Explorer, Properties / SQL / Statistics, Query Tool and View/Edit Data work the same for all of them.

### Added
- **MySQL and MariaDB.** Databases, tables, views, functions and procedures, columns, indexes, constraints
  and triggers in the explorer; DDL from `SHOW CREATE …`; sizes, row estimates and server status as
  statistics. Query Tool: multi-statement scripts with one result per statement, `DELIMITER` scripts,
  warnings shown as messages, transaction status, Cancel (`KILL QUERY`), EXPLAIN and EXPLAIN ANALYZE
  (`ANALYZE` on MariaDB). View/Edit Data with grid edits saved in one transaction. TLS modes disable /
  prefer / require / verify-full; `mysql_native_password` and `caching_sha2_password` logins. Tested with
  MySQL 9.1 and MariaDB 11.5.
- **SQLite files.** Open a file (optionally create it, or open read-only); tables, views, indexes, triggers
  and foreign keys; the DDL as SQLite stores it; row counts and sizes. Query Tool with `EXPLAIN QUERY PLAN`,
  error positions and Cancel. Tables without a primary key are edited through their `rowid`.
- **SSH tunnels for PostgreSQL and MySQL.** "Connect through an SSH tunnel" on the server dialog's SSH tab
  uses a host saved in the SSH workspace (jump hosts included). Its host key, password or key passphrase is
  asked for when needed. TLS still verifies the database's real host name, and Backup / Restore go through
  the tunnel too.
- **Server dialog:** engine picker, SQLite file Browse / New, SSH tab.
- **Dialect-aware SQL editor:** MySQL (backticks, `#` comments, `"…"` strings, backslash escapes) and SQLite
  (`[brackets]`, backticks) highlighting, statement splitting and formatting.
- Drop and truncate for MySQL and SQLite objects generate each engine's own SQL.

### Changed
- New crate `crates/devkit-db` (mysql_async; rusqlite with SQLite bundled).
- PostgreSQL-only tools (Dashboard, Maintenance, Backup / Restore, create dialogs, Search Objects) are
  hidden or explained for the other engines.
- The status bar shows the engine and version (for example "MySQL 8.4.3").
- The About dialog shows the real version (it said 0.1.0 since 0.2).
- SSH: a remote forward whose local target cannot be reached closes the channel at once.

### Not yet included
- Dashboard, Search Objects and create dialogs for MySQL and SQLite; MySQL users and privileges.
- MariaDB accounts that sign in only with the ed25519 or GSSAPI plugins.

## 0.3.1 — Terminal size fix (2026-10-22)

### Fixed
- SSH and local terminals opened at 80×24 instead of the pane's size, so full-screen programs such as
  `top`, `htop` and `vim` used only part of the width and height. The terminal now measures itself before
  the session starts, and sends any size change made while connecting once the session is ready.

## 0.3.0 — SSH & terminal (2026-10-22)

A Termius-style workspace for remote Linux servers (Ctrl+Shift+3): hosts, terminals, file transfer and
tunnels, in pure Rust with no OpenSSH install needed.

### Added
- **Host manager.** Hosts with label, group, tags, colour, port, username, jump host (ProxyJump chains),
  keepalive, startup command, environment variables and notes. Grouped, collapsible list with live
  connection dots and search. Type `user@host[:port]` in the search box and press Enter to connect without
  saving. Import hosts from `~/.ssh/config` (Host, HostName, User, Port, IdentityFile, ProxyJump).
- **Authentication:** password, private key (with passphrase), SSH agent (Windows OpenSSH agent or Pageant),
  none, and keyboard-interactive. Passwords and key passphrases live in the OS keychain, asked for on first
  connect, with an option to remember them.
- **Host-key verification.** Unknown keys show the fingerprint before you trust them; a changed key is
  refused with a warning. Trusted keys are written to the app's own `known_hosts`; your
  `~/.ssh/known_hosts` is read but never modified.
- **Terminal.** xterm.js with WebGL rendering, 256 colours and true colour, Unicode 11 widths, search
  (Ctrl+Shift+F), copy/paste (Ctrl+Shift+C/V, Ctrl+C with a selection, right-click), font zoom. Split into
  up to four panes (right or down), broadcast typing to all panes, reconnect a closed session with Enter.
  Local terminals too: PowerShell, Windows PowerShell, Command Prompt, Git Bash, WSL (ConPTY).
- **Snippets.** Saved commands, inserted from the toolbar or Ctrl+Shift+S, or run on several hosts at once
  with the output collected per host.
- **SFTP.** Two-pane file manager (this computer ↔ server): path bar, show or hide dotfiles, multi-select,
  drag and drop between panes, upload and download of files and folders with progress and cancel, new
  folder, rename, delete, chmod, and an in-app editor for remote text files.
- **Keychain.** Generate Ed25519 keys (optionally passphrase-protected), import existing OpenSSH keys, copy
  public keys, install a public key on a host (`authorized_keys`); review and forget known hosts.
- **Port forwarding:** local (-L), remote (-R) and dynamic SOCKS5 (-D) rules, started and stopped from the
  sidebar or automatically when the app starts, with live connection counts. Connections to the same host
  are shared between terminals, SFTP and tunnels.

### Changed
- New crate `crates/devkit-ssh` (russh, russh-sftp, portable-pty); xterm.js 5 is vendored in `ui/vendor/xterm`.
- The preview mock has SSH hosts to try (see the README).

### Not yet included (planned)
- RSA/ECDSA key generation (import works), X11 forwarding, Mosh, and port-forward tunnels for database
  connections (0.4).

## 0.2.0 — API client (2026-10-22)

PgAdmin RS becomes a developer workbench: a rail of workspaces next to the database explorer, starting
with a Postman-class API client.

### Added
- **Workspaces rail.** Switch the sidebar between Databases and API Client (Ctrl+Shift+1 / 2). Tabs stay
  shared, so a query and a request can sit side by side. Each workspace loads its code on first use.
- **Command palette** (Ctrl+K or Ctrl+Shift+P): fuzzy search over commands, servers, open tabs, saved
  requests and environments.
- **API client — requests.** Methods (and custom ones), query and `:path` parameters synced both ways with
  the URL, headers with name and value autocomplete, bodies (JSON, XML, text, form URL-encoded, multipart
  with files, binary file, GraphQL query + variables), Bearer / Basic / API-key auth with inheritance from
  folders and collections, per-request settings (timeout, redirects, TLS verification, HTTP/1.1 only, proxy),
  Markdown docs. Paste a `curl` command into the URL bar to import it.
- **API client — responses.** Status, timing breakdown and size; collapsible JSON tree with search,
  highlighted XML/HTML/text, raw view, sandboxed HTML preview, images; headers, cookies and a timeline of the
  request, redirects and timings. Save bodies to files (large ones stream to disk), save responses as
  examples. Stacked or side-by-side layout.
- **Collections as files.** One pretty-printed JSON file per request, folders as directories, so
  collections can live in a git repository. Drag and drop, inline rename, duplicate, folder and collection
  settings (variables, auth, docs).
- **Environments and globals** with `{{variables}}` highlighted and autocompleted everywhere; unresolved
  variables are flagged. Secret values are stored in the OS keychain, never in files. Dynamic variables such
  as `{{$guid}}` and `{{$timestamp}}`.
- **History** of sent requests (last 1,000), grouped by day and searchable; a persistent **cookie jar**.
- **Import / export:** Postman collections v2.0 / v2.1 and environments, cURL (bash, cmd and PowerShell
  quoting); open an existing collection folder.
- **Code generation** in 14 languages: cURL (bash and cmd), PowerShell, HTTPie, fetch, axios, Node.js,
  Python requests, Go, Rust reqwest, Java, C#, PHP and Ruby.
- **Shared editor engine.** The SQL editor's engine now also highlights JSON, JavaScript, GraphQL, XML/HTML,
  YAML, shell, protobuf and Markdown, with read-only mode and variable completion.

### Changed
- The Cargo project is now a workspace (`crates/devkit-core`, `crates/devkit-http`, `src-tauri`); build
  output moved from `src-tauri/target` to `target`.
- Preview mode serves files without caching: `python ui-dev/serve.py 5180`.

### Not yet included (planned)
- Pre-request and test scripts, the collection runner, OAuth 2.0 and Digest auth, GraphQL schema explorer
  and OpenAPI / HAR import arrive in 0.7. These fields are kept when imported, so nothing is lost.

## 0.1.0 — PostgreSQL workbench

Server registry with keychain passwords, object explorer, properties / DDL / statistics, query tool with
EXPLAIN, data editor, live dashboard, maintenance, backup and restore.
