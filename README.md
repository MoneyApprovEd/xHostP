<p align="center">
  <img src="banner.svg" alt="xHostP Web Panel">
</p>

<div align="center">

![Paper](https://img.shields.io/badge/Paper-1.20+-0082C9?style=for-the-badge&logo=paper&logoColor=white)
![Java](https://img.shields.io/badge/Java-17-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-22C55E?style=for-the-badge)
![Build](https://img.shields.io/badge/Build-Maven-C71A36?style=for-the-badge&logo=apachemaven&logoColor=white)
![Dependencies](https://img.shields.io/badge/Dependencies-Zero-8B5CF6?style=for-the-badge)

</div>

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [API](#api)
- [Architecture](#architecture)
- [Security](#security)
- [Compatibility](#compatibility)
- [Building](#building)
- [License](#license)

---

## Overview

xHostP is a **zero-dependency** Paper plugin that serves a premium, single-page administration panel directly from your Minecraft server. No external web server, no complicated setup — just drop the JAR into `plugins/`, restart, and open `http://localhost:8080`.

The entire frontend is embedded inside the plugin itself as a Java text block — HTML, CSS, and JavaScript all in one file, with zero external assets at runtime.

---

## Features

### Console
Live server console with real-time log streaming. Commands execute on the main thread via the Bukkit scheduler. Color-coded output (INFO / WARN / ERROR), auto-scroll, and an unread badge for activity when scrolled up.

### Status Dashboard
Four real-time metric cards: RAM usage (used / max + percentage), CPU load, TPS with color thresholds (green > 19, yellow > 15, red < 15), and online player count.

### Player Management
Online players displayed in a table with avatars (mc-heads.net), health bars, food, XP, ping indicators, and inventory slot usage. Click any player to open a modal with 15 management actions:

| Category | Actions |
|---|---|
| Permissions | OP, DeOP |
| Gamemode | Survival, Creative, Adventure, Spectator |
| Movement | TP, Fly, Land |
| Health | Heal, Feed |
| Control | Clear, Kill, Kick, Ban |

### Plugin Store
Browse and search the Modrinth plugin catalog directly from the panel. Each card shows icon, title, author, description, and download count. Click for a detail modal with statistics and one-click install.

Downloaded plugins are auto-loaded at runtime using Paper's internal `PaperPluginManagerImpl` API — the same approach as PlugManX — bypassing the Bukkit remap that fails on Paper and Folia.

### File Manager
Full filesystem access through the browser. Breadcrumb navigation, directory listing with sizes and dates, inline text editor, multi-file upload, folder creation, and file deletion with confirmation.

---

## Quick Start

```bash
# 1. Download the plugin
wget https://github.com/emirlqq1_/xHostP/releases/latest/download/xHostP.jar

# 2. Install
mv xHostP.jar /path/to/server/plugins/

# 3. Restart your server
# 4. Open in your browser
open http://localhost:8080
```

The panel binds to **127.0.0.1** only and is not accessible from other machines on the network.

---

## Configuration

```yaml
# plugins/xHostP/config.yml
web-port: 8080   # HTTP port (default: 8080)
```

---

## API

All endpoints return JSON unless noted otherwise.

### Console

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/console` | Buffered log text (`text/plain`) |
| `POST` | `/api/console` | Execute command (`command=...`) |

### Status

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/status` | `{ memUsed, memMax, cpuUsage, tps, onlinePlayers, maxPlayers }` |

### Players

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/players` | Online players with stats |
| `GET` | `/api/player?name=<name>` | Full player detail |

### Store

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/store?q=<query>` | Modrinth search |
| `GET` | `/api/project?slug=<slug>` | Project metadata |
| `POST` | `/api/download` | Download + auto-load (`slug=...`) |

### Files

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/files?path=<path>` | List / read |
| `POST` | `/api/files` | Write, upload, mkdir, delete |

---

## Architecture

```
┌──────────────────────────────────────────┐
│            Browser (SPA)                 │
│        http://localhost:8080             │
└─────────────────┬────────────────────────┘
                  │  HTTP
┌─────────────────▼────────────────────────┐
│  com.sun.net.httpserver.HttpServer       │
│  Bound to 127.0.0.1:{port}              │
├──────────────────────────────────────────┤
│  StaticHandler         →  /              │
│  ConsoleHandler        →  /api/console   │
│  StatusHandler         →  /api/status    │
│  PlayersHandler        →  /api/players   │
│  PlayerDetailHandler   →  /api/player    │
│  StoreHandler          →  /api/store,    │
│                           /api/project,  │
│                           /api/download  │
│  FileHandler           →  /api/files     │
├──────────────────────────────────────────┤
│           Paper API / Bukkit             │
└──────────────────────────────────────────┘
```

### Design Decisions

- **Zero dependencies** — HTTP server is JDK built-in (`com.sun.net.httpserver`). No Netty, Jetty, or Tomcat.
- **Single-page app** — Entire frontend (HTML + CSS + JS) stored as a Java text block in `StaticHandler.java`. No external assets at runtime.
- **Anti-AI design** — Glassmorphism, strict opacity hierarchy, CSS-only icons, inset shadow buttons. Every detail avoids common AI-generated patterns.
- **Runtime plugin loading** — Uses reflection to access Paper's `PaperPluginManagerImpl` directly, avoiding the `"Failed to remap plugin jar"` error.

---

## Security

- Server binds to **127.0.0.1** — localhost only
- File manager rejects path traversal (`../`, absolute paths escaping root)
- Commands sanitized to prevent injection
- No authentication — intended for local access only

---

## Compatibility

| Platform | Status |
|---|---|
| **Paper 1.20+** | Fully tested |
| **Folia** | Compatible |
| **Spigot** | Limited — TPS fallback; Bukkit API may fail on complex plugins |

---

## Building

Requirements: **JDK 17+**, **Maven 3.8+**

```bash
git clone https://github.com/emirlqq1_/xHostP.git
cd xHostP
mvn clean package
```

Output: `target/xHostP.jar`

---

## License

Distributed under the **MIT License**. See [LICENSE](LICENSE) for more information.
