# Proxmox OpenDesign + OpenCode

One-command installer for running **OpenCode Web** and **OpenDesign** permanently
inside an isolated Ubuntu 24.04 LTS virtual machine on Proxmox VE — reachable
from every device on your LAN via the browser.

Abgeleitet von [HatchetMan111/OpenCode-Proxmox](https://github.com/HatchetMan111/OpenCode-Proxmox)
(OpenCode-Proxmox v1.10.1), erweitert um [nexu-io/open-design](https://github.com/nexu-io/open-design)
als nativen zweiten Dienst.

## What it does

The installer runs on a Proxmox VE host and automatically:

- creates a dedicated Ubuntu 24.04 LTS VM
- chooses a free VM ID
- downloads the official Ubuntu Cloud Image
- verifies the image with Ubuntu's published SHA256 checksum
- configures Cloud-Init
- installs QEMU Guest Agent
- installs OpenCode using the official installer
- runs `opencode web` as a systemd service
- clones, builds and runs OpenDesign (Node 24 + pnpm, native, no Docker) as a systemd service
- starts OpenCode and OpenDesign automatically after reboots
- creates `/home/opencode/projects`
- installs a dufs file server (upload, photo preview, browser text editor, WebDAV) on port 8080
- protects the OpenCode Web UI with HTTP Basic Authentication
- protects OpenDesign with an API token (`OD_API_TOKEN`)
- enables a VM firewall that only permits OpenCode, OpenDesign and the file server from private IPv4 networks
- prints the VM IP and all Web UI URLs when installation is complete

Ubuntu publishes official 24.04 LTS Cloud Images and SHA256 checksums: [https://cloud-images.ubuntu.com/releases/server/24.04/release/](https://cloud-images.ubuntu.com/releases/server/24.04/release/)

OpenCode documents `opencode web --hostname 0.0.0.0`, port 4096, HTTP Basic Authentication and mDNS: [https://opencode.ai/docs/web/](https://opencode.ai/docs/web/)

OpenDesign is installed natively (`git clone https://github.com/nexu-io/open-design.git`,
`pnpm install`) and runs in production mode like the official Docker image
(`node apps/daemon/dist/cli.js --no-open`): the daemon serves its API plus
the built web UI itself on `0.0.0.0:7456`. It auto-detects the OpenCode CLI
(`/home/opencode/.opencode/bin`) as its agent.

## One-line installation

Run the following on your **Proxmox VE host as root**:

```sh
bash -c "$(curl -fsSL https://raw.githubusercontent.com/HatchetMan111/OpenDesignServer/main/install.sh)"
```

## Result

At the end, the installer prints something like:

```
============================================================
                 OpenDesign + OpenCode sind bereit
============================================================

  🌐  OpenCode Web-UI im Browser öffnen:

       http://192.168.178.123:4096

  📋  OpenCode-Login (beim ersten Screen der Web-UI):

       Benutzer:      opencode
       Web-Passwort:  xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

  🎨  OpenDesign im Browser öffnen:

       http://192.168.178.123:7456

  📋  OpenDesign-Login (nativer Browser-Dialog):

       Benutzer:   open-design
       API-Token:  yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy

  📁  Dateiserver (dufs) im Browser öffnen:

       http://192.168.178.123:8080

       Benutzer:   opencode
       Passwort:   opencode
```

Open the displayed addresses in a browser on your LAN. The OpenDesign
browser dialog expects user `open-design` and the printed `OD_API_TOKEN`
as password. A wrong token answers `401`.

## Model providers

The VM does **not** hard-code a single AI provider.

OpenCode currently supports 75+ LLM providers and local models. Provider credentials can be configured from OpenCode using:

```
/connect
```

Models can then be selected using:

```
/models
```

See the official OpenCode provider documentation:

[https://opencode.ai/docs/providers/](https://opencode.ai/docs/providers/)

and model documentation:

[https://opencode.ai/docs/models/](https://opencode.ai/docs/models/)

OpenDesign uses the OpenCode CLI inside the VM as its local agent runtime,
so both UIs share the same provider setup.

### Important

The installer cannot magically provide paid models without credentials/subscriptions. You still need to authenticate the providers you want to use.

## LAN-only security

The VM listens on:

```
0.0.0.0:4096
0.0.0.0:7456
0.0.0.0:8080
```

because that is required for other devices on the LAN to reach the Web UIs.

The VM's UFW firewall only allows TCP/4096 (OpenCode), TCP/7456 (OpenDesign)
and TCP/8080 (file server) from:

- `10.0.0.0/8`
- `172.16.0.0/12`
- `192.168.0.0/16`

SSH is restricted to the same private IPv4 ranges.

### Router

Do **not** create a port-forward from the Internet to port `4096`, `7456` or `8080`.

For remote access, use a VPN such as WireGuard or Tailscale rather than exposing OpenCode or OpenDesign directly to the Internet.

## VM defaults

| Setting | Default |
|---|---|
| OS | Ubuntu Server 24.04 LTS |
| CPU | 2 vCPU |
| RAM | 8 GB |
| Disk | 32 GB |
| Network | DHCP |
| Bridge | `vmbr0` |
| OpenCode port | `4096` |
| OpenDesign port | `7456` |
| File server port | `8080` |
| VM name | `opendesign-opencode` |
| Project directory | `/home/opencode/projects` |
| File server directory | `/home/opencode/files` |
| OpenDesign directory | `/opt/open-design` |

## OpenCode service

OpenCode runs as:

```
systemctl status opencode
```

Follow logs:

```sh
journalctl -u opencode -f
```

The server is configured with:

```
opencode web --hostname 0.0.0.0 --port 4096
```

## OpenDesign service

OpenDesign runs as:

```
systemctl status opendesign
```

Follow logs:

```sh
journalctl -u opendesign -f
```

The daemon serves its API plus the built web UI on `0.0.0.0:7456` and is
protected by the `OD_API_TOKEN` from `/etc/opendesign/env`.

## Updating

Inside the VM:

```sh
sudo /usr/local/bin/opencode-update
sudo /usr/local/bin/opendesign-update
```

Check versions:

```sh
/usr/local/bin/opencode-version
/usr/local/bin/opendesign-version
```

Rotate the OpenDesign token:

```sh
sudo /usr/local/bin/opendesign-token <neues-token>
```

## File server (dufs)

The VM also runs a [dufs](https://github.com/sigoden/dufs) file server on port `8080`:

- Web UI in the browser: upload via drag & drop, photo preview, text editor, folder download as ZIP
- WebDAV access and `curl` up-/downloads
- files live in `/home/opencode/files` (same place is reachable from OpenCode)

Default login:

```
user:     opencode
password: opencode
```

Change it right after installation (alphanumeric password recommended):

```sh
qm terminal <VMID>
sudo fileserver-password <neues-passwort>
```

Service status and logs:

```sh
systemctl status fileserver
journalctl -u fileserver -f
```

Note: FileBrowser was deliberately **not** used — the project has been archived (read-only since 2026-09-01) with unpatched security issues. dufs is actively maintained and a single static binary with no extra dependencies.

## Accessing the VM

The installer creates the Linux user:

```
opencode
```

The generated password is printed once at the end of the installation.

You can also use the Proxmox console:

```
VM → Console
```

## Backups

The OpenCode state, authentication data, sessions, projects and the
OpenDesign data (SQLite DB + artifacts in `/opt/open-design`) live inside the VM.

For a homelab setup, back up the entire Proxmox VM.

In particular, do not publish:

- OpenCode provider credentials
- API keys
- OAuth tokens
- generated passwords
- the `OD_API_TOKEN`
- `/home/opencode/.local/share/opencode/`
- private project repositories

## Requirements

- Proxmox VE
- x86_64/AMD64 host
- root access on the Proxmox host
- an active network bridge, normally `vmbr0`
- Internet access from the Proxmox host during installation
- enough storage for the Ubuntu image and VM disk

## Repository layout

```
opendesign-opencode/
├── install.sh
├── README.md
├── LICENSE
├── .gitignore
└── CHANGELOG.md
```

## Troubleshooting

Falls ein Dashboard nicht lädt, verbinde dich zur VM:

```sh
qm terminal <VMID>
```

und prüfe:

```sh
a. IP prüfen:  ip a
b. Dienst:     systemctl status opencode
               systemctl status opendesign
               systemctl status fileserver
c. Logs:       journalctl -u opencode -e
               journalctl -u opendesign -e
               journalctl -u fileserver -e
d. Setup-Log:  cat /var/log/opencode-setup.log
               cat /var/log/opendesign-setup.log
```

Die Ersteinrichtung installiert nach dem ersten Boot noch einige Minuten
lang Pakete, OpenCode und OpenDesign (Node 24 + `pnpm install` + Build
dauern beim ersten Mal am längsten).

## Disclaimer

This script creates a VM, installs packages and changes firewall configuration inside that VM.

Review the script before running it on production infrastructure.

Use backups and test it on a non-critical Proxmox host first.

## License

MIT License. See `LICENSE`.

OpenCode, OpenDesign, Ubuntu and Proxmox are trademarks and/or projects of their respective owners.
