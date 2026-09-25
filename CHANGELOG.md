# Changelog

## v1.0.1

- Fix: Next.js-Web-Build lief in `JavaScript heap out of memory` (Nodes Default-Heap ~2 GB in der TypeScript-Phase). Build und `opendesign-update` nutzen jetzt `NODE_OPTIONS=--max-old-space-size=6144`.
- Hinweis-Texte zu bestehenden VMs unmissverstaendlich: Es wird nie etwas geloescht oder angefasst.

## v1.0.0

- Initiale Version: OpenCode Web (4096) + OpenDesign nativ (7456) + dufs-Dateiserver (8080) in einer Ubuntu-24.04-LTS-VM auf Proxmox VE (LAN-only).
- Abgeleitet von OpenCode-Proxmox v1.10.1 (HatchetMan111/OpenCode-Proxmox).
