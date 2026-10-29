# Privacy Policy - HomeBox for TerraMaster TOS 7

**Application**: HomeBox (`app_id: homebox`)
**Publisher**: waskevin
**Policy version**: 1.0
**Effective date**: 2026-10-29
**Contact**: https://github.com/waskevin/homebox/issues

This policy covers the TOS 7 application package distributed by waskevin
(the `homebox.tar.gz` archive and the `config.ini` / `homebox.lang` / `homebox.svg` /
`docker-compose.yml` files inside it) and the container it starts on your
device.  It does not cover the HomeBox project's own website or any service you
choose to connect to it.

> 本政策适用于 waskevin 发布的 TOS 7 应用包及其在你的设备上启动的容器。所有数据默认只保存在你自己的 TNAS 上。

## 1. Who is responsible

The publisher of this TOS 7 package is **waskevin** (GitHub: @waskevin).  The
application running inside the container is the open-source **HomeBox**
project (https://github.com/sysadminsmedia/homebox); when you use that software, its own project is the
controller of the software, while **you**, the NAS owner, are the controller
of the data you put into it.

## 2. What data is processed

- **Content you create**: inventory items, locations, labels, photos and attachments, account data, entered through the web UI.
- **Account data**: a local administrator account (username, password hash)
  created on first use.
- **Technical data**: application logs and the SQLite database produced by the
  application itself.


The package itself collects nothing: it contains only configuration files
and an icon.

## 3. Where the data is stored

Everything is stored locally on your TNAS, in the application data root:

```
/Volume<N>/DockerAppData/homebox/data   ->  /data (inside the container)
```

`N` is the volume you selected during installation.  Nothing is written to
`/etc`, `/root` or `/boot`.  The container runs as the unprivileged user
`1000:1000` and can only write to that mounted directory.

## 4. Retention period

- Data is retained **indefinitely on your device**, until you delete it.
- There is no automatic expiry, no scheduled cleanup and no rotation.
- Uninstalling the application from App Center **keeps** the data directory by
  default; it is removed only if you explicitly choose to delete the data, or
  if you delete `/Volume<N>/DockerAppData/homebox/` yourself.
- The publisher holds no copy of your data and therefore cannot retain it.

## 5. Network behaviour and data destinations

- **No telemetry, no analytics, no crash reporting, no usage statistics.**
- **No data is uploaded to the publisher or to any third party** by this
  package or by its default configuration.
- The only outbound request made by the package itself happens **once, at
  install/update time**: the TOS platform pulls the container image
  `sysadminsmedia/homebox:0.27.1` from **Docker Hub** (`hub.docker.com`, operated by Docker, Inc.).
  Docker Hub sees your public IP address and the requested image name, as with
  any HTTP request.  See <https://www.docker.com/legal/privacy/>.
- At runtime the container serves the web UI over plain HTTP on your LAN at
  port `10089`.  It does not call home.
- **Optional, off by default**: email and webhook notifiers let you forward events to an SMTP server, a webhook URL or a chat service you configure yourself.  If you enable one, the data you choose to send (for example an item name or a notification body) leaves the device to that destination, which you select and control.  The package itself configures none.

## 6. Third-party services

| Service | When | Data leaving the device |
|---|---|---|
| Docker Hub (`hub.docker.com`) | install / update only | public IP address, requested image name |
| Your SMTP / webhook / chat endpoint (optional, user-configured) | only if you enable a notifier | the notification content you choose to send |

No other third party is contacted.  There is no publisher-operated server, no analytics endpoint and no cloud sync.


## 7. Security measures

- The container runs as a non-root user (`user: "1000:1000"`).
- `privileged` mode is not used; `network_mode: host` is not used.  Only the
  single web port is published.
- No credentials, tokens, API keys or database connection strings are shipped
  in the package.  Secrets that the application needs are generated **on your
  device** at first start.
- Data is stored in a single directory that ordinary NAS users cannot reach;
  it is only accessible to the system account that runs Docker Engine, and to
  administrators.
- The web UI is **plain HTTP** by default.  If you expose it beyond a trusted
  LAN, terminate TLS in front of it (reverse proxy or VPN) - the package does
  not provide transport encryption.

## 8. Your rights

Because the publisher never receives your data, you exercise your rights
directly on your device:

- **Access / portability**: the data directory above is a plain, documented
  layout that you can copy or back up at any time.
- **Rectification**: edit or delete records in the application's web UI.
- **Erasure**: uninstall the application and choose to delete its data, or
  delete `/Volume<N>/DockerAppData/homebox/` from the NAS.  This is immediate and
  irreversible.
- **Restriction / objection**: stop the application in App Center, or disable
  it; no further processing takes place.

If you are in the EEA/UK and believe the *packaging* itself (not the
application) breaches data-protection law, contact the publisher via
https://github.com/waskevin/homebox/issues.

## 9. Children

The application is a general-purpose self-hosted tool.  It is not directed at
children and the publisher does not knowingly process any data about children.

## 10. Changes to this policy

This document lives in the public repository
<https://github.com/waskevin/homebox>.  Any material change is published there
and in the next package version, with the policy version and effective date
updated above.  The policy shipped with the version you installed is the one
that applies to it.

## 11. Contact

Issues and questions: https://github.com/waskevin/homebox/issues
