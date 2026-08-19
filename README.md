# 🎮 Questarr Docker - Gestor de Videojuegos Estilo *Arr

[![GitHub](https://img.shields.io/badge/GitHub-Doezer%2FQuestarr-181717?logo=github)](https://github.com/Doezer/Questarr)
[![Docker](https://img.shields.io/badge/Docker-ghcr.io%2Fdoezer%2Fquestarr-2496ED?logo=docker)](https://github.com/Doezer/Questarr/pkgs/container/questarr)
[![License](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://github.com/Doezer/Questarr/blob/main/LICENSE)

## 📋 Descripción general

**Questarr** es un gestor de videojuegos autohospedado inspirado en el ecosistema *Arr (Sonarr, Radarr, Prowlarr) que permite descubrir, rastrear y descargar automáticamente los videojuegos que deseas. Incluye metadatos profesionales de IGDB, integración nativa con Prowlarr para indexers centralizados, soporte multi-plataforma, auto-búsqueda y auto-descarga, post-procesamiento automático, y gestión completa de colección con estados de posesión y juego.

Desarrollado en **Node.js + React** con base de datos **SQLite** autocontenida, Questarr es la solución definitiva para gamers que quieren su propia biblioteca de juegos autohospedada con automatización completa.

## ✨ Características principales

- **Descubrimiento inteligente**: IGDB API (metadata, ratings, covers), RSS feeds (xREL.to), sincronización Steam Wishlist
- **Estados de colección avanzados**: Wanted, Owned, Playing, Completed, Shelved + ratings personales + notas
- **Auto-search + Auto-download**: Búsqueda automática de releases hasta encontrar + descarga automática al detectar disponibilidad
- **Integración Prowlarr nativa**: Sincronización de indexers centralizada (Torznab compatible) - configuración única
- **Multi-downloader**: qBittorrent, Transmission, rTorrent, SABnzbd, NZBGet
- **Búsqueda avanzada**: Filtrado por plataforma (PC, PS5, Xbox, Nintendo), género (RPG, FPS, etc), palabras clave
- **Preferencias de release**: Grupos preferidos, blacklist de grupos/plataformas específicas
- **Post-procesamiento automático**: Mueve descargas completadas a librería + importación automática
- **Notificaciones Apprise**: 100+ proveedores (Discord, Telegram, Slack, Gotify, Email, etc)
- **Dashboard estadísticas**: Estado de cola, visión general colección, tracking de releases
- **UI React moderna**: Responsive, multi-idioma, dashboard interactivo
- **Unraid Community App**: Instalación un-click en Unraid
- **GPL v3 Open Source**: Código abierto, community-driven, transparente

## 📋 Requisitos del sistema

- **Docker & Docker Compose v2+**
- **1-2 GB RAM** mínimo
- **500 MB - 10+ GB** espacio disco (DB + cache metadata + juegos)
- **Puerto 5000 TCP** (Web UI)
- **Acceso a internet persistente** (IGDB API, feeds RSS, descargas)
- **IGDB API Key** (gratuita, registro simple en [igdb.com](https://igdb.com))
- **Downloader configurado** (qBittorrent, Transmission, rTorrent, SABnzbd, o NZBGet)
- **Opcional**: Prowlarr para indexers centralizados

> **Nota sobre IGDB API**: Requiere key gratuita (5 minutos setup). Proporciona metadata de juegos profesional (covers, ratings, géneros, plataformas, fechas lanzamiento).

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

### Opción 3: Con Caddy reverse proxy (HTTPS automático)

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

docker compose up -d
```

### Acceso a Questarr

| Método | URL |
|--------|-----|
| Local | `http://localhost:5000` |
| Remoto HTTPS | `https://games.tudominio.com` (si Caddy) |

## ⚙️ Configuración

1. **IGDB API Key**: Obtén tu key gratuita en [igdb.com](https://igdb.com) → Register → Dashboard → API
2. **Indexers**: Settings → "Indexers" → Sync Prowlarr (un click) O agregar manualmente URLs Torznab
3. **Download Client**: Settings → "Download Clients" → qBittorrent → URL: `http://qbittorrent:6969` → Test → Save
4. **Auto-download**: Por juego → "Monitoring" → Enable "Auto-download" → Seleccionar plataforma preferida
5. **Notificaciones**: Settings → "Notifications" → Proveedor (Discord, Telegram, Gotify, etc) → Webhook/Token
6. **Steam Wishlist** (opcional): Settings → "Steam Integration" → Steam User ID → Sync automático

## 🚀 Primeros pasos

1. **Obtener IGDB API Key**: Visita [igdb.com](https://igdb.com) → Register → Completa signup → Dashboard → API → Copiar key
2. **Acceder Web UI**: Abre `http://localhost:5000` → Aparece setup wizard → Ingresa IGDB API key
3. **Configurar indexers**: Settings → "Indexers" → Opción A: Sync Prowlarr / Opción B: Agregar Torznab manual → Save
4. **Configurar downloader**: Settings → "Download Clients" → qBittorrent → URL `http://qbittorrent:6969` → Test → Save
5. **Buscar + agregar juego**: Dashboard → "+ Add Game" → Busca (ej: "The Witcher 3") → Click juego → Detalles + opciones
6. **Configurar auto-download**: Juego → "Monitoring" → Enable "Auto-download" → Plataforma preferida (PC, PS5, etc)
7. **Ver colección**: Tab "Collection" → Todos los juegos con estado (Wanted, Owned, Playing, Completed, Shelved) → Click → Editar estado/rating/notas
8. **Monitorear descargas**: Tab "Queue" → Descargas en progreso (Searching, Downloading, Completed)
9. **Configurar notificaciones**: Settings → "Notifications" → Proveedor → Webhook/Token → Notifica al completar
10. **Sync Steam Wishlist** (opcional): Settings → "Steam Integration" → Steam User ID → Sincroniza automático

## 💡 Casos de uso

- **Gamers homelabs**: Biblioteca de juegos autohospedada con auto-descargas y descubrimiento automático (IGDB + RSS)
- **Descubrimiento de lanzamientos**: Encuentra nuevos releases automáticamente via IGDB metadata + xREL.to feeds
- **Retro gaming archiving**: Colección de clásicos con estados Playing/Completed + ratings + notas personales
- **Multi-platform tracking**: PC, PS5, Xbox, Nintendo en un solo gestor unificado
- **DevOps gamer**: Misma infraestructura servidor (Unraid/Proxmox) + stack *Arr completo integrado

## 🔒 Acceso remoto seguro

Para exposición segura a internet, se recomienda:

- **Caddy reverse proxy** (incluido en Opción 3): HTTPS automático con Let's Encrypt
- **Tailscale / WireGuard**: VPN mesh para acceso privado sin puertos abiertos
- **Authelia / Authentik**: Autenticación SSO + 2FA delante del proxy
- **Cloudflare Tunnel**: Acceso sin abrir puertos + WAF + DDoS protection

## 🛠️ Gestión y mantenimiento

```bash
# Ver logs en tiempo real
docker logs -f questarr

# Reiniciar contenedor
docker compose restart questarr

# Actualizar a versión reciente
docker pull ghcr.io/doezer/questarr:latest
docker compose up -d

# Monitorear consumo recursos
docker stats questarr
# Típicamente: 100-400MB RAM, CPU baja

# Backup colección + config
docker cp questarr:/app/data ./backup-questarr-$(date +%Y%m%d)

# Limpiar base de datos (reset completo)
docker compose down
rm -rf data/
docker compose up -d
```

## 📝 Licencia

**GPL v3** - Código abierto, community-driven, transparente.
Repositorio oficial: [Doezer/Questarr](https://github.com/Doezer/Questarr)

---

> 📖 **Guía completa original**: [Cómo instalar Questarr en Docker - Gestor videojuegos estilo *Arr](https://genbyte.blogspot.com/2026/08/como-instalar-questarr-en-docker-gestor.html)
> 
> 🎥 **Canal YouTube**: [Genbyte](https://youtube.com/@genbyte) | 💬 **Telegram**: [@genbyte](https://t.me/genbyte) | ☕ **Ko-fi**: [Invítame un café](https://ko-fi.com/genbyte)