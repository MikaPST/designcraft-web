<p align="center">
  <img src="https://raw.githubusercontent.com/storytold/designcraft/main/assets/app-icon/hicolor/256x256/apps/ai.storyteller.designcraft.png" width="128" height="128" alt="DesignCraft application icon">
</p>

<h1 align="center">DesignCraft Web — Docker Image</h1>
<p align="center">
  A community-maintained Docker image for
  <a href="https://github.com/storytold/designcraft">DesignCraft</a>,
  bringing its WebAssembly-based page layout and publishing studio to a self-hosted web interface.
  <br>
  Build automation and image publication to GitHub Container Registry (GHCR).
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

The upstream DesignCraft project provides a WebAssembly build that can be served as a static website. This repository automates building that web application from upstream source and packages the generated files into a lightweight NGINX-based Docker image.

The goals are to:

- Publish a matching GitHub Release only after the Docker build and push succeed.
- Build the WebAssembly application from official release tags in [storytold/designcraft](https://github.com/storytold/designcraft).
- Include the shared font assets from [storytold/craft-fonts](https://github.com/storytold/craft-fonts), as required for complete release builds and Japanese text support.
- Check weekly for new official releases and publish a versioned image to GHCR.
- Provide Docker deployment for Unraid Community Applications.
- Keep this packaging project separate from the upstream DesignCraft source code.

This is an independent community packaging project. It is **not an official DesignCraft release** and is not affiliated with or endorsed by the DesignCraft or ArtCraft maintainers.

## How it works

DesignCraft is a page layout and publishing application written in Rust. Its web application runs in the browser using WebAssembly and eframe. The container serves the compiled static web files through NGINX; it does not run a remote desktop or a server-side document-editing service.

- **Browser-based publishing studio:** create multi-page layouts using pages and spreads, parent pages, text frames, threaded stories, paragraph and character styles, tables, text wrap and more.
- **Typography and layout:** use the upstream text composition engine, columns, baseline grids, tabs, rules, shading and print-oriented layout tools.
- **Local file handling:** Open and Place use the browser's file picker; files can also be dropped into the application. Save and Export download files through the browser.
- **No remote desktop:** no VNC session or Linux graphical desktop is required.
- **Self-hosted delivery:** the container hosts the application files. It does not provide a shared document repository or server-side control channel.

The web build has no control channel. The JSON control protocol and MCP functionality described by the upstream project are not available through this browser build.

DesignCraft is at the **pre-alpha** stage and is under active development. Features, compatibility and behaviour may change between upstream builds. Test updates and verify exported documents before relying on it for important publishing work.

## Docker image

The published image is:

```text
ghcr.io/mikapst/designcraft-web:latest
```

Each official upstream release is also published under its exact version tag, for example:

```text
ghcr.io/mikapst/designcraft-web:vX.Y.Z
```

Use a versioned tag if you want to keep a deployment on a specific release. The `latest` tag updates when an image is published.

### Quick start with Docker Compose

A `docker-compose.yml` file is provided at the repository root:

```yaml
services:
  designcraft-web:
    image: ghcr.io/mikapst/designcraft-web:latest
    container_name: designcraft-web
    restart: unless-stopped
    ports:
      - "8080:8080"
```

Start DesignCraft Web from the repository directory:

```bash
docker compose up -d
```

Then open:

```text
http://YOUR_SERVER_IP:8080
```

Replace `YOUR_SERVER_IP` with the IP address or hostname of your server. On the same computer, use `http://localhost:8080`.

### Run directly with Docker

```bash
docker run -d \
  --name designcraft-web \
  --restart unless-stopped \
  -p 8080:8080 \
  ghcr.io/mikapst/designcraft-web:latest
```

### Useful commands

```bash
# View logs
docker compose logs -f

# Pull the latest image and recreate the container
docker compose pull
docker compose up -d

# Stop and remove the container
docker compose down
```

No `.env` file or persistent Docker volume is required for the default configuration. The host port is `8080`; to use a different host port, change the left-hand side of the port mapping, for example `8081:8080`.

### Reverse proxy and browser requirements

The NGINX configuration:

- serves `.wasm` files with the `application/wasm` MIME type;
- uses the MIME types provided by NGINX for JavaScript, CSS, fonts and images;
- enables gzip compression for WebAssembly, JavaScript, CSS and HTML;
- applies `Cache-Control: no-cache` to `index.html`;
- applies long-lived immutable caching to content-hashed static assets generated by Trunk.

DesignCraft uses WebGPU when available and falls back to WebGL2. The `?webgl` query parameter forces WebGL2. WebGPU generally requires a secure browser context (HTTPS, or localhost). If you expose the application outside your local network, use HTTPS through a reverse proxy such as Traefik and avoid exposing plain HTTP directly to the public Internet.

The web build uses relative asset paths and can be served by a static web server. If you publish it below a URL prefix, configure your reverse proxy to preserve the requested path.

### Web application options

| URL option | Behaviour |
|---|---|
| No option | Open the normal DesignCraft web application |
| `?webgl` | Force the WebGL2 renderer instead of WebGPU |
| `?sample` | Open the sample magazine |

For example:

```text
https://YOUR_DOMAIN/?sample
```

The web application uses the browser's file picker for Open and Place, and supports drag and drop. Save and Export download the resulting files. Documents are not automatically stored in the Docker container.

## Automated builds

GitHub Actions checks the official [DesignCraft releases](https://github.com/storytold/designcraft/releases), builds the WebAssembly application from the selected upstream tag and publishes the image to GHCR.

| Trigger | Behaviour |
|---|---|
| Weekly schedule | Checks the latest official DesignCraft release and builds it if it has not already been published here |
| Manual workflow dispatch | Builds the latest official release, or a specified `upstream_tag` |
| Force rebuild | Rebuilds an already published version with `force_rebuild: true` |
| Changes to packaging files on `main` | Rebuilds the current official upstream release with the updated workflow |

### Published tags

- `latest` — the latest successfully published image (mutable tag).
- `<upstream-version>` — the exact official upstream tag, for example `vX.Y.Z`.
- `upstream-<commit>` — the upstream source commit used for the image (first 12 hexadecimal characters).

The image targets both `linux/amd64` and `linux/arm64`. The workflow checks out **the official upstream release tag**, rather than compiling `main` as the versioned source.

### GitHub Releases

After a successful Docker build and GHCR push, the workflow creates or updates a [GitHub Release](https://github.com/MikaPST/designcraft-web/releases) matching the upstream tag. It includes the upstream release notes, source link and commit, Docker tags, supported architectures and (when available) the published image digest.

The Release badge at the top of this README updates automatically. Scheduled runs skip versions already released here. You can force a rebuild from **Actions → Build and publish DesignCraft Web → Run workflow**.

> [!IMPORTANT]
> A GitHub Release is created only after the Docker image has been successfully published. The `latest` tag may change after a rebuild; pin a versioned image tag for a stable deployment. Auxiliary source repositories (for example `craft-fonts`) may be checked out separately and are not necessarily pinned to the upstream release commit.

### Build pipeline

The workflow:

1. Checks out this repository.
2. Checks out DesignCraft and the shared `craft-fonts` repository.
3. Installs the Rust WebAssembly target and Trunk.
4. Sets `CRAFT_FONTS_DIR` to the checked-out font repository.
5. Runs `trunk build --release` from `upstream/apps/designcraft-web`.
6. Uses the upstream Trunk configuration to generate the static site in `upstream/dist/web`.
7. Checks that the generated `index.html`, WebAssembly and JavaScript assets exist.
8. Generates the Dockerfile and NGINX configuration in a temporary build context.
9. Packages the static site into an NGINX image and publishes it to GHCR.

The upstream project states that release builds include `craft-fonts`; without those fonts, Japanese text falls back to system fonts, which are not available in the browser environment. This packaging workflow checks out the shared font repository and makes it available during compilation.

The workflow generates the Dockerfile and NGINX configuration during the GitHub Actions run. Separate `Dockerfile`, `nginx.conf` and `.dockerignore` files are therefore not required in this packaging repository.

The repository is primarily intended as an automated image-build and publication pipeline; the normal way to consume it is to pull the published image.

## Unraid Community Applications

This image can be deployed on Unraid.

The Unraid Community Applications template is maintained in [MikaPST/unraid-community-apps](https://github.com/MikaPST/unraid-community-apps), separately from this image-build repository.

In the meantime, DesignCraft Web can be deployed on Unraid using the Docker Compose configuration above or by configuring the container manually with:

- **Repository:** `ghcr.io/mikapst/designcraft-web:latest`
- **Container port:** `8080`
- **Host port:** `8080` by default

Documents opened in the browser are not stored in the Unraid appdata directory. Save or export your work and keep independent copies of important source files and documents.
- **Template:** [designcraft-web.xml](https://github.com/MikaPST/unraid-community-apps/blob/main/templates/designcraft-web.xml)

## Upstream project

- **DesignCraft source code:** [storytold/designcraft](https://github.com/storytold/designcraft)
- **DesignCraft web build instructions:** [Web section in README](https://github.com/storytold/designcraft#web)
- **Shared font assets:** [storytold/craft-fonts](https://github.com/storytold/craft-fonts)
- **Official releases:** [DesignCraft releases](https://github.com/storytold/designcraft/releases)

The MikaPST GitHub Release tracks the upstream version but represents independent Docker packaging, not an official DesignCraft distribution.

## License and attribution

DesignCraft is dual-licensed under the MIT License or Apache License 2.0, at the user's option. Required notices are provided in the upstream `NOTICE` file. Bundled fonts, icons, images and other assets retain their own licenses and attribution requirements; consult upstream `ASSETS.md` and the `craft-fonts` repository.

The ArtCraft name, wordmark and logos in the upstream `docs/brand/` directory are trademarks and are not covered by the software license. Their use is subject to the upstream `docs/brand/LICENSE-brand.txt`. The license of the upstream application and its assets does not automatically determine the license of the packaging files maintained in this repository. If you want to grant reuse rights for your own workflow, Compose file and documentation, select and include a separate `LICENSE` for this repository.

## Disclaimer

This repository is maintained independently and is not affiliated with, sponsored by or endorsed by the DesignCraft project or its maintainers.

DesignCraft is pre-alpha software. Keep independent copies of important documents and test updates before relying on the application for important publishing work.
