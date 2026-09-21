# 🎮 Questarr Docker

[![GitHub Stars](https://img.shields.io/github/stars/doezer/Questarr?style=for-the-badge&logo=github)](https://github.com/doezer/Questarr)
[![Docker Pulls](https://img.shields.io/docker/pulls/ghcr.io/doezer/questarr?style=for-the-badge&logo=docker)](https://github.com/doezer/Questarr/pkgs/container/questarr)
[![License](https://img.shields.io/github/license/doezer/Questarr?style=for-the-badge)](https://github.com/doezer/Questarr/blob/main/LICENSE)
[![GitHub Release](https://img.shields.io/github/v/release/doezer/Questarr?style=for-the-badge&logo=github)](https://github.com/doezer/Questarr/releases)

## 📋 Descripción general

**Questarr** es un gestor de videojuegos autohospedado inspirado en el ecosistema *Arr (Sonarr, Radarr, Prowlarr) que permite descubrir, rastrear y descargar automáticamente los videojuegos que deseas. Combina metadatos profesionales de **IGDB**, integración nativa con **Prowlarr** para indexers centralizados, soporte multi-plataforma, auto-búsqueda y auto-descarga, post-procesamiento automático, y gestión de colección con estados de posesión y juego.

Desarrollado en **Node.js + React**, con base de datos **SQLite** autocontenida, listo para **Unraid** y licenciado bajo **GPL v3**.

## ✨ Características principales

- 🎯 **Descubrimiento de juegos**: IGDB API (metadata, ratings, covers), RSS feeds (xREL.to), Steam Wishlist sync
- 📚 **Estados de colección avanzados**: Wanted, Owned, Playing, Completed, Shelved + ratings + notas personales
- 🔍 **Auto-search + Auto-download**: Búsqueda automática de releases hasta encontrar + descarga automática al detectar release
- 🔗 **Prowlarr sync integrado**: Indexers centrales con un click, compatibilidad Torznab, sin configuración per-app
- ⬇️ **Multi-downloader support**: qBittorrent, Transmission, rTorrent, SABnzbd, NZBGet
- 🎮 **Búsqueda avanzada**: Filtrado por plataforma (PC, PS5, Xbox, Nintendo), género (RPG, FPS, etc), keywords
- ⚙️ **Post-processing automático**: Mueve descargas completadas a librería automáticamente, importación automática
- 🔔 **Notificaciones Apprise**: 100+ providers (Discord, Telegram, Slack, Gotify, Email, etc)
- 🚫 **Release blacklisting**: Excluye grupos/plataformas específicas, control granular
- 📊 **Dashboard estadísticas**: Queue status, collection overview, release tracking
- 🌐 **Multi-language UI** + Responsive design
- 💾 **SQLite persistence** autocontenida
- 📦 **Unraid Community App** listo, instalación un-click
- 🔓 **GPL v3 open source**, community-driven, transparente

## 📋 Requisitos del sistema

- Docker & Docker Compose v2+
- 1 GB - 2 GB RAM mínimo
- 500 MB - 10+ GB espacio disco (DB + cache metadata + juegos)
- Puerto 5000 TCP (web UI)
- Acceso internet persistente (IGDB API, feeds, downloads)
- **IGDB API key** (gratuito, registro simple en [igdb.com](https://igdb.com))
- Downloader configurado (qBittorrent, Transmission, etc)
- Opcional: **Prowlarr** (indexers centralizados)

> **Nota sobre IGDB API**: Requiere key gratuita (5 minutos setup). Proporciona metadata de juegos profesional.
> **Recomendado**: Docker compose con Prowlarr + qBittorrent mismo stack para flujo completo.

## 🐳 Instalación

### Opción 1: Docker Compose simple (recomendado)

```bash
mkdir -p questarr && cd questarr
cat > docker-compose.yml << 'EOF'
version: '3.8'
services:
  questarr:
    image: ghcr.io/doezer/questarr:latest
    container_name: questarr
    restart: unless-stopped
    ports:
      - "5000:5000"
    volumes:
      # Persistencia: DB, config, cache
      - ./data:/app/data
      # Path opcional: librería juegos descargados
      - ./games:/games
    environment:
      SQLITE_DB_PATH: /app/data/sqlite.db
      TZ: Europe/Madrid
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000"]
      interval: 30s
      timeout: 5s
      retries: 3
EOF
mkdir -p data games
docker compose up -d
```

### Opción 2: Stack completo (Questarr + qBittorrent + Prowlarr)

```bash
cat > docker-compose.yml << 'EOF'
version: '3.8'
services:
  questarr:
    image: ghcr.io/doezer/questarr:latest
    container_name: questarr
    restart: unless-stopped
    ports:
      - "5000:5000"
    volumes:
      - ./questarr-data:/app/data
      - ./games:/games
    environment:
      SQLITE_DB_PATH: /app/data/sqlite.db
      TZ: Europe/Madrid

  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    restart: unless-stopped
    ports:
      - "6969:6969"
      - "6969:6969/udp"
    volumes:
      - ./qbit-config:/config
      - ./downloads:/downloads
    environment:
      PUID: 1000
      PGID: 1000
      TZ: Europe/Madrid

  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    restart: unless-stopped
    ports:
      - "9696:9696"
    volumes:
      - ./prowlarr-config:/config
    environment:
      PUID: 1000
      PGID: 1000
      TZ: Europe/Madrid
EOF
mkdir -p questarr-data qbit-config prowlarr-config downloads games
docker compose up -d
```

### Opción 3: Con Caddy reverse proxy HTTPS

```bash
cat >> docker-compose.yml << 'EOF'

  caddy:
    image: caddy:latest
    container_name: questarr-caddy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
      - caddy_config:/config

volumes:
  caddy_data:
  caddy_config:
EOF

cat > Caddyfile << 'EOF'
games.tudominio.com {
  reverse_proxy questarr:5000
}
EOF
```

### Acceder a Questarr

| Método | URL |
|--------|-----|
| Local | `http://localhost:5000` |
| HTTPS remoto (con Caddy) | `https://games.tudominio.com` |

## ⚙️ Configuración

1. **Obtener IGDB API key**: Visita [igdb.com](https://igdb.com) → Register → Dashboard → API → Copiar key
2. **Acceder Web UI**: Abre `http://localhost:5000` → Setup wizard → Ingresa IGDB API key
3. **Configurar indexers**: Settings → "Indexers" → Opción A: Sync Prowlarr (un click) / Opción B: Agregar manualmente URLs Torznab → Save
4. **Configurar downloader**: Settings → "Download Clients" → Selecciona qBittorrent → URL local: `http://qbittorrent:6969` → Test connection → Save
5. **Configurar auto-download**: Juego → "Monitoring" → Enable "Auto-download" → Selecciona plataforma preferida
6. **Configurar notificaciones**: Settings → "Notifications" → Selecciona proveedor (Discord, Telegram, etc) → Ingresa webhook/token
7. **Sync Steam Wishlist (opcional)**: Settings → "Steam Integration" → Ingresa Steam user ID

## 🚀 Primeros pasos

1. **Obtener IGDB API key**: Visita igdb.com → Register → Completa signup simple → Dashboard → API → Copiar key → Guarda la key
2. **Acceder web UI**: Abre `http://localhost:5000` → Interface aparece: setup wizard → Ingresa IGDB API key cuando pida
3. **Configurar indexers**: Settings → "Indexers" → Opción A: Sync Prowlarr (si tienes) - un click / Opción B: Agregar manualmente URLs Torznab → Save
4. **Configurar downloader**: Settings → "Download Clients" → Selecciona qBittorrent → Ingresa URL local: `http://qbittorrent:6969` → Test connection → debe ser verde → Save
5. **Buscar + agregar juego**: Dashboard → "+ Add Game" → Busca juego (ej: "The Witcher 3") → Resultados IGDB aparecen → Click juego → detalles + opciones
6. **Configurar auto-download**: Juego → "Monitoring" → Enable "Auto-download" → Selecciona plataforma preferida (PC, PS5, etc) → Questarr automáticamente buscará + descargará
7. **Ver colección**: Tab "Collection" → Ve todos juegos agregados con estado → Estados: Wanted (buscando), Owned (tengo), Playing, Completed, Shelved → Click juego → editar estado + rating + notas
8. **Monitorear descargas**: Tab "Queue" → Ve descargas en progreso → Status: Searching, Downloading, Completed
9. **Configurar notificaciones**: Settings → "Notifications" → Selecciona proveedor (Discord, Telegram, etc) → Ingresa webhook/token → Questarr notifica cuando descarga completado
10. **Sync Steam Wishlist (opcional)**: Settings → "Steam Integration" → Ingresa Steam user ID → Questarr sincroniza Wishlist automático → Agrega juegos de tu lista deseados

## 💡 Casos de uso

- 🏠 **Gamers homelabs**: Biblioteca juegos autohospedada con auto-descargas y descubrimiento automático (IGDB + RSS feeds)
- 🎮 **Descubrimiento de nuevos lanzamientos**: Encuentra automáticamente nuevos releases de juegos deseados
- 📼 **Retro gaming archiving**: Colección de clásicos con estados playing/completed y notas personales
- 🖥️ **Multi-platform tracking**: PC, PS5, Xbox, Nintendo en un solo gestor unificado
- ⚙️ **DevOps gamer**: Mismo servidor infraestructura, integrado en Unraid/Proxmox, gestión centralizada

## 🔒 Acceso remoto seguro

Para acceso remoto seguro se recomienda:

1. **Reverse proxy con Caddy** (incluido en Opción 3): HTTPS automático con Let's Encrypt
2. **Autenticación adicional**: Authelia, Authentik o Cloudflare Access delante del proxy
3. **VPN**: WireGuard/Tailscale para acceso solo desde red privada
4. **Firewall**: Restringir puerto 5000 solo a IPs de confianza si no usas proxy

## 🛠️ Gestión y mantenimiento

```bash
# Ver logs en tiempo real
docker logs -f questarr

# Restart container
docker compose restart questarr

# Actualizar a versión reciente
docker pull ghcr.io/doezer/questarr:latest
docker compose up -d

# Monitorear consumo
docker stats questarr
# Típicamente: 100-400MB RAM

# Backup colección + config
docker cp questarr:/app/data ./backup-questarr-$(date +%Y%m%d)

# Limpiar base de datos (reset completo)
docker compose down
rm -rf data/
docker compose up -d
```

## 📝 Licencia

Este proyecto está licenciado bajo **GPL v3** - ver archivo [LICENSE](https://github.com/doezer/Questarr/blob/main/LICENSE) para detalles.

---

> 📖 **Guía completa en el blog**: [Cómo instalar Questarr en Docker - Gestor videojuegos estilo *Arr](https://genbyte.blogspot.com/2026/08/como-instalar-questarr-en-docker-gestor.html)