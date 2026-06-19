<p align="center">
  <img src="banner.svg" alt="xHostP Web Panel">
</p>

<div align="center">
  <a href="#"><img src="https://img.shields.io/badge/Paper-1.20+-0082C9?style=for-the-badge&logo=paper&logoColor=white" alt="Paper"></a> <a href="#"><img src="https://img.shields.io/badge/Java-17-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white" alt="Java"></a> <a href="#"><img src="https://img.shields.io/badge/License-MIT-22C55E?style=for-the-badge" alt="License"></a> <a href="#"><img src="https://img.shields.io/badge/Dependencies-Zero-8B5CF6?style=for-the-badge" alt="Dependencies"></a>
</div>

---

## Overview

xHostP is a lightweight, zero-dependency server administration panel designed for Paper and Spigot. It serves a modern, single-page application directly from your Minecraft server without requiring external web servers or reverse proxies. Just drop the JAR into your `plugins` folder, restart, and manage your server securely from `localhost`.

## Features

* **Real-Time Console:** Stream server logs directly to your browser with color-coded output and execute commands on the main thread.
* **Status Dashboard:** Monitor live server metrics including RAM usage, CPU load, TPS, and online player count.
* **Advanced Player Management:** View online players with avatars, health bars, and latency. Instantly manage users (Kick, Ban, Heal, Gamemode, Teleport) via the interactive modal.
* **Integrated Modrinth Store:** Browse, search, and install plugins directly from the Modrinth catalog. Installed plugins are auto-loaded at runtime seamlessly.
* **Web File Manager:** Navigate your server filesystem, read/edit text files, upload assets, and manage directories directly through the browser.

---

## Installation

1. Download the latest `xHostP.jar` from the [Releases](#) page.
2. Place the JAR file into your server's `plugins/` directory.
3. Restart or reload your server.
4. Open your web browser and navigate to `http://localhost:8080`.

*Note: For security purposes, the web panel binds strictly to `127.0.0.1` and is inaccessible from external networks.*

---

## API Reference

xHostP provides a lightweight RESTful API for external integrations. All endpoints return JSON format.

### Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/console` | Retrieve buffered log text |
| `POST` | `/api/console` | Execute server command |
| `GET` | `/api/status` | Fetch RAM, CPU, TPS, and Player stats |
| `GET` | `/api/players` | List all online players and base stats |
| `GET` | `/api/store?q=<query>` | Search Modrinth catalog |
| `POST` | `/api/download` | Download and auto-load a plugin |
| `GET` | `/api/files?path=<path>` | List directory contents or read file |
| `POST` | `/api/files` | Upload, edit, or delete files |

---

## Configuration

The plugin generates a minimal `config.yml` on first boot:

```yaml
# plugins/xHostP/config.yml
web-port: 8080   # HTTP port for the web interface
