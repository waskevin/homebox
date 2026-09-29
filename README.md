# HomeBox for TerraMaster TOS 7

TOS 7 App Center package for [HomeBox](https://homebox.software) — a self-hosted inventory and organization system for your home.

一个把 [HomeBox](https://homebox.software) 打包上架到 TOS 7 应用中心的 Docker 应用（单容器、单数据卷、SQLite，零外部依赖）。

---

## What it does / 这是什么

HomeBox keeps track of everything you own: locations, labels, warranties, purchase prices, photos and attachments — so you know what you have and where it is.

- Web UI on host port `10089` → container `7745`
- Single SQLite database, stored on your NAS
- Runs as a non-root user, no privileged mode, no host networking
- Also exposes a REST API (`/api/`) and a health endpoint (`/api/health`)

## Package contents / 包内文件

```
homebox.tar.gz
├── config.ini            # TOS application metadata
├── homebox.lang          # 14 languages (zh-cn, zh-hk, en-us, ... pt-pt)
├── homebox.svg           # application icon (SVG, transparent)
└── docker-compose.yml    # container definition
```

## docker-compose.yml

```yaml
version: "3.8"

services:
  homebox:
    image: sysadminsmedia/homebox:0.26.2
    restart: unless-stopped
    user: "1000:1000"
    ports:
      - "10089:7745"
    volumes:
      - ./data:/data
    environment:
      - TZ=Asia/Shanghai
    entrypoint: ["/bin/sh", "-c"]
    command:
      - |
        if [ ! -s /data/.pepper ]; then
          head -c 48 /dev/urandom | base64 > /data/.pepper
        fi
        export HBOX_AUTH_API_KEY_PEPPER="$(cat /data/.pepper)"
        exec /app/api /data/config.yml
    healthcheck:
      test: ["CMD-SHELL", "(command -v curl >/dev/null && curl -fsS http://localhost:7745/) || (command -v wget >/dev/null && wget -q -O /dev/null http://localhost:7745/) || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 3

x-app-meta:
  web:
    port: 10089
    protocol: http
```

### Why the entrypoint wrapper / 为什么要这段引导脚本

HomeBox refuses to start unless `HBOX_AUTH_API_KEY_PEPPER` is set (it is used to sign API keys), and a TOS application package may only contain the four files listed above — there is no installer step in which a secret could be generated, and shipping a fixed value inside `docker-compose.yml` would put a credential in the package.

The wrapper therefore generates a **random 48-byte pepper on first start**, stores it in the persistent data volume (`/data/.pepper`, mode 600, owned by the container user) and re-uses it on every later start. Verified on a real TOS 7 device: the pepper survives restarts (`docker restart`), so API keys stay valid, and no secret is ever stored in this repository.

If you prefer to manage it yourself, replace the wrapper with a plain `environment: - HBOX_AUTH_API_KEY_PEPPER=<your 48-byte value>`.

## Data location / 数据位置

The host side of `./data` is resolved by the platform to the application data root:

```
/Volume<N>/DockerAppData/homebox/data
```

It contains `homebox.db` (SQLite) plus `.pepper`. Data survives uninstall as long as "delete data" is not selected, and it also survives reinstall.

## Build

```bash
./build.sh            # produces homebox.tar.gz + homebox.tar.gz.sha256
```

## Release

The package is published as a Release asset (the TOS Developer Platform downloads it from the Release, not from the repository root):

- Tag: `1.0.0`
- Asset: `homebox.tar.gz`
- Checksum: `homebox.tar.gz.sha256`

## Compliance notes / 合规说明

- Image source: Docker Hub only (`sysadminsmedia/homebox`), version tag pinned, never `:latest`.
- Runs as non-root (`user: "1000:1000"`); no `privileged`, no `network_mode: host`.
- **No credentials are stored in this repository**: the API key pepper is generated on the device at first start.
- `container_name` is intentionally omitted, so Compose derives a globally unique name (`<project>_<service>_1`) and the app never fails to install because the device already runs a container with the same name.
- The icon in this repository is original placeholder artwork created for this package; it is **not** the upstream project logo, to avoid trademark issues.
- This repository contains packaging metadata only. The application itself is developed by the HomeBox project and distributed under its own license (AGPL-3.0). All credit belongs to the upstream authors.

## Links

- Upstream project: https://github.com/sysadminsmedia/homebox
- Docker Hub image: https://hub.docker.com/r/sysadminsmedia/homebox
- TOS 7 development guide: https://github.com/terramaster-tos/tos-app-pkg-tools
