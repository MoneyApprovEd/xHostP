<p align="center">
  <img src="https://img.shields.io/badge/Paper-1.20+-blue?style=flat-square" alt="Paper">
  <img src="https://img.shields.io/badge/Java-17%2B-orange?style=flat-square" alt="Java">
  <img src="https://img.shields.io/badge/License-MIT-green?style=flat-square" alt="License">
  <img src="https://img.shields.io/badge/Build-Maven-red?style=flat-square" alt="Maven">
  <img src="https://img.shields.io/badge/Dependencies-Zero-important?style=flat-square" alt="Zero Dependencies">
</p>

# xHostP

**Embedded web administration panel for Paper 1.20+ servers.**

xHostP is a zero-dependency Paper plugin that serves a premium, single-page administration panel directly from your Minecraft server. No external web server, no complicated setup — drop the JAR into `plugins/`, restart, and open `http://localhost:8080`.

---

## Features

### Console
Live server console with real-time log streaming. Commands execute on the server's main thread via the Bukkit scheduler. Log lines are color-coded (INFO / WARN / ERROR), auto-scroll follows new output, and an unread badge shows activity when scrolled up.

### Status Dashboard
Four metric cards display current server health: RAM usage (used / max + percentage), CPU load, TPS with color thresholds (<span style="color:#22c55e">green</span> > 19, <span style="color:#eab308">yellow</span> > 15, <span style="color:#ef4444">red</span> < 15), and online player count.

### Player List & Management
Online players appear in a table with mc-heads.net avatars, health bars, food, XP percentage, ping indicators (green < 100 ms, yellow < 250 ms, red > 250 ms), and inventory slot usage. Click any player to open a management modal with 15 actions:

| Category | Actions |
|---|---|
| Permissions | `OP`, `DeOP` |
| Gamemode | `Survival`, `Creative`, `Adventure`, `Spectator` |
| Movement | `TP`, `Fly`, `Land` |
| Health | `Heal`, `Feed` |
| Control | `Clear`, `Kill`, `Kick`, `Ban` |

### Plugin Store
Browse and search the Modrinth plugin catalog directly from the panel. Each card shows icon, title, author, description preview, and download count. Click a card for the full detail modal:

- Icon, title, and author
- Full description with a link to the Modrinth body page
- Statistics: downloads, followers, license, project type, created / updated dates
- Category tags
- One-click install with automatic plugin loading

**Auto-load:** Downloaded plugins are loaded at runtime using Paper's internal `PaperPluginManagerImpl` API (the same approach as PlugManX), bypassing the standard Bukkit remap that fails on Paper and Folia.

### File Manager
Full filesystem access through the browser. Breadcrumb navigation, directory listing with size and modification dates, an inline text editor, multi-file upload, folder creation, and file deletion with confirmation dialog.

---

## Installation

1. Download `xHostP.jar`
2. Place it in your server's `plugins/` directory
3. Restart the server
4. Open [http://localhost:8080](http://localhost:8080)

The panel binds to `127.0.0.1` only and is **not** accessible from other machines on the network.

---

## Configuration

`plugins/xHostP/config.yml`

```yaml
# TCP port for the embedded web server (default: 8080)
web-port: 8080
```

---

## Building from Source

**Requirements:** JDK 17+, Maven 3.8+

```bash
mvn clean package
```

Output: `target/xHostP.jar`

---

## Architecture

```
┌─────────────────────────────────────┐
│          Browser (SPA)              │
│     http://localhost:8080           │
└──────────────┬──────────────────────┘
               │  HTTP (plain / JSON)
┌──────────────▼──────────────────────┐
│  com.sun.net.httpserver.HttpServer  │
│  Bound to 127.0.0.1:{port}         │
├─────────────────────────────────────┤
│  StaticHandler        →  /          │
│  ConsoleHandler       →  /api/console │
│  StatusHandler        →  /api/status  │
│  PlayersHandler       →  /api/players │
│  PlayerDetailHandler  →  /api/player  │
│  StoreHandler         →  /api/store,  │
│                          /api/project,│
│                          /api/download│
│  FileHandler          →  /api/files   │
├─────────────────────────────────────┤
│       Paper API / Bukkit            │
└─────────────────────────────────────┘
```

### Key Design Decisions

- **Zero runtime dependencies** — The HTTP server is the JDK's built-in `com.sun.net.httpserver`. No Netty, no Jetty, no embedded Tomcat.
- **Single-page application** — The entire frontend (HTML + CSS + JavaScript) lives inside a Java text block in `StaticHandler.java`. No external assets at runtime.
- **Anti-AI design language** — Glassmorphism (`backdrop-filter: blur`), strict color opacity hierarchy, CSS-only icons (no Unicode, no SVG), inset box-shadow buttons. Every visual detail deliberately avoids common AI-generated design patterns.
- **Paper remap bypass** — Plugin auto-load uses reflection to access `io.papermc.paper.plugin.manager.PaperPluginManagerImpl`, calling `instanceManager.loadPlugin(Path)` and `instanceManager.enablePlugin(Plugin)` directly. This avoids the `"Failed to remap plugin jar"` error that occurs with the standard Bukkit `loadPlugin()` on modern Paper builds.

---

## API Reference

Endpoints return JSON unless noted otherwise.

### Console

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/console` | Returns buffered log text (`text/plain`) |
| `POST` | `/api/console` | Executes a server command (`command=...`) |

### Status

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/status` | `{ memUsed, memMax, cpuUsage, tps, onlinePlayers, maxPlayers }` |

### Players

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/players` | Array of online players with health, food, XP, ping, gamemode, inventory slots |
| `GET` | `/api/player?name=<name>` | Full player detail including armor, inventory, offhand, location |

### Store

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/store` | Modrinth search results (optional `?q=<query>`) |
| `GET` | `/api/project?slug=<slug>` | Full Modrinth project metadata |
| `POST` | `/api/download` | Download and auto-load plugin (`slug=...`) |

### Files

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/files?path=<path>` | List directory or read file content |
| `POST` | `/api/files` | Write content, upload, mkdir, or delete (`path=...&content=...` / `path=...&upload=...` / `path=...&mkdir=true` / `path=...&delete=true`) |

---

## Security

- The HTTP server binds exclusively to `127.0.0.1` — only localhost connections are accepted.
- File manager resolves all paths against the server root and rejects traversal attempts (`../` or absolute paths escaping the root).
- Player action commands are sanitised to prevent injection.
- No authentication is implemented — the panel is intended for local access only.

---

## Compatibility

| Platform | Status |
|---|---|
| **Paper 1.20+** | Fully tested and supported |
| **Folia** | Supported — Paper plugin manager API is compatible |
| **Spigot** | Limited — TPS uses the built-in `TpsTracker` fallback instead of Paper's `getTPS()`; auto-load falls back to Bukkit's `loadPlugin()` which may fail on complex plugins due to remapping differences |

---

## License

This project is licensed under the [MIT License](LICENSE). You are free to use, modify, and distribute it as permitted by the terms of the license.
