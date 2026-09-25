# Changelog

## v1.0.3

- Fix: OpenDesign lief via `tools-dev run web` (Dev-Server, bindet grundsaetzlich nur 127.0.0.1 - vom LAN nicht erreichbar). Startscript nutzt jetzt den Produktions-Modus wie im offiziellen Docker-Image (`node apps/daemon/dist/cli.js --no-open`, bindet `OD_BIND_HOST=0.0.0.0:OD_PORT`). Unit setzt `NODE_ENV=production` + Heap-Cap 1024 MB fuer den Dauerbetrieb. Setup prueft zusaetzlich, dass der Static-Export `apps/web/out` existiert.

## v1.0.2

- Fix: `opendesign-start` lud `/etc/opendesign/env` (0600 root:root) zur Laufzeit als User `opencode` nach (Permission denied, Restart-Loop). Env kommt jetzt ausschliesslich aus der systemd-Unit per `EnvironmentFile` (wird von systemd als root gelesen).

## v1.0.1

- Fix: Next.js-Web-Build lief in `JavaScript heap out of memory` (Nodes Default-Heap ~2 GB in der TypeScript-Phase). Build und `opendesign-update` nutzen jetzt `NODE_OPTIONS=--max-old-space-size=6144`.
- Hinweis-Texte zu bestehenden VMs unmissverstaendlich: Es wird nie etwas geloescht oder angefasst.

## v1.0.0

- Initiale Version: OpenCode Web (4096) + OpenDesign nativ (7456) + dufs-Dateiserver (8080) in einer Ubuntu-24.04-LTS-VM auf Proxmox VE (LAN-only).
- Abgeleitet von OpenCode-Proxmox v1.10.1 (HatchetMan111/OpenCode-Proxmox).
