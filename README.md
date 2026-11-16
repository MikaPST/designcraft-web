<p align="center">
  <img src="https://raw.githubusercontent.com/storytold/designcraft/main/assets/app-icon/hicolor/256x256/apps/ai.storyteller.designcraft.png" width="128" height="128" alt="DesignCraft application icon">
</p>

<h1 align="center">DesignCraft Web — Docker Image</h1>
<p align="center">
  A community-maintained Docker image for
  <a href="https://github.com/storytold/designcraft">DesignCraft</a>,
  bringing its Rust/WebAssembly-based page layout and publishing studio to a self-hosted web interface.
  <br>
  Automated builds and image publication to GitHub Container Registry (GHCR).
</p>

<p align="center">
  <a href="https://github.com/MikaPST/designcraft-web/actions/workflows/publish-image.yml">
    <img src="https://github.com/MikaPST/designcraft-web/actions/workflows/publish-image.yml/badge.svg" alt="Build and publish DesignCraft Web">
  </a>
  <a href="https://github.com/MikaPST/designcraft-web/releases/latest">
    <img src="https://img.shields.io/github/v/release/MikaPST/designcraft-web?label=Release&logo=github&color=blue" alt="Latest Release">
  </a>
  <a href="https://github.com/MikaPST/designcraft-web/pkgs/container/designcraft-web">
    <img src="https://ghcr-badge.egpl.dev/mikapst/designcraft-web/latest_tag?trim=major&label=GHCR" alt="GHCR image tag">
  </a>
  <a href="docker-compose.yml">
    <img src="https://img.shields.io/badge/Docker%20Compose-ready-2496ED?logo=docker&logoColor=white" alt="Docker Compose configuration">
  </a>
</p>

---

## Why this repository?

The official DesignCraft project provides a Rust-based page layout and publishing application with a WebAssembly version that runs in a browser. This repository builds the official upstream web application and packages its generated static files into a lightweight NGINX Docker image for straightforward self-hosting.

The goals are to:

- Build DesignCraft Web from a selected **official upstream release tag**, not an unversioned development checkout.
- Publish ready-to-run Docker images to GHCR for `linux/amd64` and `linux/arm64`.
- Automatically detect new official releases and support manual or forced rebuilds.
- Publish a matching GitHub Release **only after the Docker image has been pushed successfully**.
- Include the shared font assets from [storytold/craft-fonts](https://github.com/storytold/craft-fonts) during compilation, including support for Japanese text.
- Provide Docker Compose and Unraid Community Applications deployment options.
- Maintain independent, version-controlled `Dockerfile`, `nginx.conf`, `.dockerignore`, and build automation without changing the upstream project.
- Preserve upstream licensing and font attribution notices in the runtime image.

This is an independent packaging and distribution project, **not an official DesignCraft release** and not an upstream fork.

## How it works

DesignCraft is a page layout and publishing application written in Rust. Its web edition runs in the browser using WebAssembly and eframe. The container serves the compiled static application through NGINX; it does **not** provide a remote desktop, shared document server, or server-side editing backend.

- **Publishing studio:** create multi-page layouts with pages and spreads, parent pages, text frames, threaded stories, paragraph and character styles, tables, and text wrap.
- **Typography and layout:** work with columns, baseline grids, tabs, rules, shading, and other print-oriented design tools provided by the upstream application.
- **Local file handling:** Open and Place use the browser's file picker; drag and drop is supported. Save and Export download files through the browser.
- **WebAssembly application:** the editor runs on the user's machine; the container only provides static application files.
- **GPU rendering:** WebGPU is used when available, with a WebGL2 fallback.
- **Self-hosting:** access the editor on your LAN or behind an HTTPS reverse proxy.

The web build **does not expose the native JSON control protocol or MCP server** documented for the desktop version. It is not a remote automation or document-storage service.

> [!IMPORTANT]
> DesignCraft is **pre-alpha software**. Test updates and inspect exported files before relying on them for production publishing work. Keep independent copies of important source documents and exports.

## Docker image

The image is published to GitHub Container Registry:

```text
ghcr.io/mikapst/designcraft-web:latest
```

Official upstream releases are also published under their exact version tags after successful packaging. For example:

```text
ghcr.io/mikapst/designcraft-web:v0.21.0
```

Use an existing version tag to remain on a specific upstream release. The `latest` tag is mutable. A forced rebuild can also replace the image behind a version tag; use an image digest when strict binary reproducibility is needed.

### Option 1 — Docker Compose (recommended)

The repository includes a `docker-compose.yml` file:

```yaml
services:
  designcraft-web:
    image: ghcr.io/mikapst/designcraft-web:latest
    container_name: designcraft-web
    ports:
      - "8080:8080"
    restart: unless-stopped
```

Start the application from the repository root:

```bash
docker compose up -d
```

Open the application in your browser:

```text
http://YOUR_SERVER_IP:8080
```

Replace `YOUR_SERVER_IP` with your server's IP address or hostname. If Docker runs on your computer, use `http://localhost:8080`.

#### Useful Compose commands

```bash
# View container logs
docker compose logs -f

# Pull an updated image and recreate the container
docker compose pull
docker compose up -d

# Stop and remove the container
docker compose down
```

No `.env` file or persistent server-side application volume is required for the default configuration. To change the host port, edit the left-hand side of the mapping. For example, `8081:8080` exposes DesignCraft on host port `8081`.

### Option 2 — Run directly with Docker

```bash
docker pull ghcr.io/mikapst/designcraft-web:latest

docker run -d \
  --name designcraft-web \
  --restart unless-stopped \
  -p 8080:8080 \
  ghcr.io/mikapst/designcraft-web:latest
```

Open `http://YOUR_SERVER_IP:8080`, or `http://localhost:8080` on the Docker host.

Optional Docker resource limits can be added when necessary, for example `--cpus=2 --memory=2g`. The web editor's performance also depends on the client browser and graphics hardware.

### Network access and reverse proxies

NGINX listens on **port `8080`**. For external access, use a reverse proxy such as Traefik to terminate HTTPS and forward requests to the container over HTTP.

The independently maintained `nginx.conf`:

- Serves `.wasm` files as `application/wasm` and `.js` as `text/javascript`.
- Enables gzip compression for WebAssembly, JavaScript, CSS, and other compressible assets.
- Sends `no-cache` for `index.html` and revalidates non-hashed JavaScript/WebAssembly assets.
- Uses long-lived `immutable` caching for content-hashed `.wasm` and `.js` files generated by Trunk.
- Provides a lightweight `GET /healthz` endpoint for container health checks.
- Sends `X-Content-Type-Options: nosniff` and runs NGINX as the **non-root `nginx` user**.

WebGPU and certain clipboard capabilities require a **secure browser context** (`https://` or `http://localhost`). On plain HTTP network addresses, the editor may fall back to WebGL2.

The upstream build uses relative asset URLs (`public_url = "./"`). To serve DesignCraft below a prefix such as `/designcraft/`, configure your reverse proxy to **strip that prefix** before forwarding to the container. Preserve the generated relative asset references.

No special COOP or COEP isolation headers are required by the current upstream web build. Avoid imposing `Cross-Origin-Embedder-Policy: require-corp` globally unless your proxy also supplies compatible resource policies.

Do not expose an unprotected HTTP service directly to the public Internet.

### Web application options

| URL option | Behaviour |
|---|---|
| No option | Launch the normal DesignCraft web editor using WebGPU when available, otherwise WebGL2. |
| `?webgl` | Force WebGL2 instead of WebGPU. |
| `?cpu` | Force the slower, more compatible CPU rendering path. |
| `?sample` | Open the upstream sample magazine. |

For example:

```text
https://YOUR_DOMAIN/?sample
https://YOUR_DOMAIN/?webgl
```

Opening or placing a document uses the browser's file picker or drag and drop. Save and Export use browser downloads, **not** Docker volumes or Unraid appdata.

### Embedding DesignCraft in another website

The upstream application supports being hosted in an iframe. For example:

```html
<iframe
  src="https://YOUR_DOMAIN/"
  title="DesignCraft publishing editor"
  style="width:100%; height:720px; border:0;"
  allow="fullscreen; clipboard-read; clipboard-write"
  allowfullscreen>
</iframe>
```

Cross-origin embedding may be affected by browser storage restrictions. Sandboxed iframes must allow scripts, same-origin behavior, downloads, and any required popups. Ensure reverse proxy `frame-ancestors` or `X-Frame-Options` policies permit your intended host page.

For full iframe requirements and current browser limitations, consult the [official web hosting guide](https://github.com/storytold/designcraft/blob/main/packaging/web/README.md).

## Repository structure

This repository maintains its own Docker packaging files while retrieving the official DesignCraft source during automated builds:

```text
designcraft-web/
├── .github/
│   └── workflows/
│       └── publish-image.yml
├── .dockerignore
├── Dockerfile
├── nginx.conf
├── docker-compose.yml
└── README.md
```

| File | Purpose |
|---|---|
| `Dockerfile` | Defines the lightweight NGINX runtime image with unprivileged execution and a health check. |
| `nginx.conf` | Configures WebAssembly MIME types, caching, compression, security headers, and `/healthz`. |
| `.dockerignore` | Limits the temporary Docker build context to required compiled assets and packaging files. |
| `docker-compose.yml` | Provides ready-to-use Docker Compose deployment. |
| `.github/workflows/publish-image.yml` | Detects upstream releases, compiles Rust/Trunk, builds and pushes multi-platform images, then publishes GitHub Releases. |
| `README.md` | Documents the image, deployment, build architecture, maintenance, and upstream attribution. |

### Docker build architecture

Unlike PhotoCraft's multi-stage Dockerfile, DesignCraft compiles its WebAssembly application **once on the GitHub Actions runner**, then packages the resulting static site into two NGINX image architectures.

```text
GitHub Actions workspace/
├── packaging/              # MikaPST/designcraft-web
│   ├── Dockerfile
│   ├── nginx.conf
│   ├── .dockerignore
│   ├── docker-compose.yml
│   └── README.md
├── upstream/               # storytold/designcraft (official release tag)
├── craft-fonts/            # storytold/craft-fonts (main)
└── docker-context/         # generated temporary Docker build context
    ├── Dockerfile          # copied from packaging/
    ├── nginx.conf          # copied from packaging/
    ├── .dockerignore       # copied from packaging/
    ├── site/               # compiled upstream/dist/web/
    └── legal/              # upstream licenses, notices, font licenses
```

The workflow performs these operations:

1. Check out and validate the version-controlled packaging files.
2. Resolve the latest official DesignCraft release, or a selected `upstream_tag`.
3. Skip an already published release unless a manual force rebuild or monitored packaging change requires another build.
4. Check out the exact upstream release tag and shared `craft-fonts` from its independent `main` branch.
5. Validate the upstream Rust workspace, WebAssembly application, and license notices.
6. Install Rust, `wasm32-unknown-unknown`, Trunk, and a compatible `wasm-bindgen-cli` from the upstream `Cargo.lock`.
7. Run `trunk build --release` in `upstream/apps/designcraft-web` with `CRAFT_FONTS_DIR` pointing to the fonts checkout.
8. Validate `upstream/dist/web/index.html`, JavaScript, WebAssembly, and relative asset references.
9. Create a temporary `docker-context/` with the compiled static site, packaging files, upstream notices, and available font licenses.
10. Use Docker Buildx to publish the same compiled site for `linux/amd64` and `linux/arm64`.
11. Create or update the matching GitHub Release **after** the image push succeeds.

The runtime Dockerfile does not compile Rust or download upstream source code. The `docker-context/`, `site/`, and `legal/` folders are generated during GitHub Actions; **they are not committed** to this repository.

The shared fonts repository is checked out separately because its assets are a required input for complete release builds, including Japanese typography. Its exact commit is recorded in image metadata and GitHub Release notes.

> [!NOTE]
> Future upstream releases may change Rust, Trunk, font requirements, or the generated web file layout. Packaging may require corresponding updates.

## Automated builds

GitHub Actions checks the official [DesignCraft releases](https://github.com/storytold/designcraft/releases), compiles the selected upstream tag, and publishes its multi-platform image to GHCR.

| Trigger | Behaviour |
|---|---|
| Daily schedule (`03:17 UTC`) | Checks for the latest official DesignCraft release and skips it when already packaged. |
| Manual workflow dispatch | Builds the latest official release or a specified `upstream_tag`. |
| Force rebuild | Rebuilds a release already published in this repository with `force_rebuild: true`. |
| Packaging changes on `main` | Rebuilds the current official upstream release when `Dockerfile`, `nginx.conf`, `.dockerignore`, `docker-compose.yml`, `README.md`, or the workflow changes. |

### Published tags

Each successful build publishes these GHCR tags:

- `latest` — the most recently successfully published image.
- `<upstream-version>` — the exact upstream release tag, for example `v0.21.0`.
- `upstream-<commit>` — the first 12 characters of the selected DesignCraft source commit.

Images target `linux/amd64` and `linux/arm64`. WebAssembly compilation happens once before Docker Buildx builds the lightweight runtime images.

The upstream DesignCraft source is pinned to a release tag, but `craft-fonts` follows its own `main` branch. A later forced rebuild of the same upstream tag can therefore include a different fonts revision.

### GitHub Releases

After a successful Docker build and GHCR push, the workflow creates or updates a [GitHub Release](https://github.com/MikaPST/designcraft-web/releases) matching the official upstream release tag.

Each Release includes:

- Official upstream release name, link, version, and publication date.
- The selected upstream source commit.
- The separately checked-out shared fonts commit.
- Published Docker image tags.
- Supported platforms (`linux/amd64` and `linux/arm64`).
- Published Docker manifest digest, when available.
- Official upstream release notes and an independent-packaging disclaimer.

The Release badge at the top of this README reflects the latest published GitHub Release.

Scheduled and ordinary manual runs skip upstream versions already released here. To rebuild, open **Actions → Build and publish DesignCraft Web → Run workflow** and enable `force_rebuild`. A push affecting monitored packaging files also triggers a rebuild.

> [!IMPORTANT]
> GitHub Releases are created or updated **only after** the Docker image has been pushed successfully to GHCR.
>
> The `latest` tag is mutable, and a forced rebuild may also replace versioned image tags. For exact deployed bits, pin the published image digest.

## Unraid Community Applications

DesignCraft Web can be deployed on Unraid using the separately maintained [MikaPST/unraid-community-apps](https://github.com/MikaPST/unraid-community-apps) template.

- **Unraid Community Applications repository:** [MikaPST/unraid-community-apps](https://github.com/MikaPST/unraid-community-apps)
- **DesignCraft Web template:** [designcraft-web.xml](https://github.com/MikaPST/unraid-community-apps/blob/main/templates/designcraft-web.xml)
- **Container image:** `ghcr.io/mikapst/designcraft-web:latest`
- **Container port:** `8080`
- **Default host port:** `8080`

The Unraid XML template is maintained separately from this packaging repository. Documents opened and edited in the browser are not automatically stored in Unraid appdata. Use Save or Export and back up important files yourself.

## Upstream project

DesignCraft is a page layout and publishing application developed by the ArtCraft team and contributors.

- **Official source:** [storytold/designcraft](https://github.com/storytold/designcraft)
- **Official web hosting guide:** [packaging/web/README.md](https://github.com/storytold/designcraft/blob/main/packaging/web/README.md)
- **Official upstream README:** [DesignCraft README](https://github.com/storytold/designcraft/blob/main/README.md)
- **Shared font assets:** [storytold/craft-fonts](https://github.com/storytold/craft-fonts)
- **Official releases:** [DesignCraft releases](https://github.com/storytold/designcraft/releases)

The MikaPST GHCR image and GitHub Release track official upstream versions but represent **independently maintained Docker packaging**, not official distributions.

## License and attribution

DesignCraft's source code is dual-licensed under the **MIT License or Apache License 2.0**, at the user's option. Copyright belongs to the ArtCraft team and DesignCraft contributors.

This packaging workflow preserves the upstream `LICENSE-MIT`, `LICENSE-APACHE`, `NOTICE`, and `ASSETS.md` in the runtime image's `/licenses/` directory. It also includes available third-party license notices from the upstream `licenses/` directory and OFL notices for shared font assets.

- [Upstream MIT License](https://github.com/storytold/designcraft/blob/main/LICENSE-MIT)
- [Upstream Apache License 2.0](https://github.com/storytold/designcraft/blob/main/LICENSE-APACHE)
- [Upstream NOTICE](https://github.com/storytold/designcraft/blob/main/NOTICE)
- [Upstream assets and attribution](https://github.com/storytold/designcraft/blob/main/ASSETS.md)
- [Shared font repository](https://github.com/storytold/craft-fonts)

The upstream application's licenses do not automatically determine the license of the independent packaging files maintained here. Refer to a separately supplied packaging `LICENSE`, if present, for those files.

The DesignCraft name, ArtCraft wordmark, artwork, and logos remain subject to the applicable upstream rights and branding rules.

## Disclaimer

This repository is independently maintained and is not affiliated with, sponsored by, or endorsed by DesignCraft, ArtCraft, or their maintainers.

DesignCraft is **pre-alpha software**. Features, browser compatibility, and export behaviour may change between builds. Test upgrades and keep independent copies of important projects and exported documents.
