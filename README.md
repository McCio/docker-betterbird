# docker-betterbird

[![publish](https://github.com/McCio/docker-betterbird/actions/workflows/publish.yml/badge.svg)](https://github.com/McCio/docker-betterbird/actions/workflows/publish.yml)
[![release](https://img.shields.io/github/v/release/McCio/docker-betterbird?label=betterbird)](https://github.com/McCio/docker-betterbird/releases)
[![license](https://img.shields.io/github/license/McCio/docker-betterbird)](LICENSE)

[Betterbird](https://www.betterbird.eu/) is a fine-tuned version of Mozilla Thunderbird: e-mail client with calendar and contacts, RSS reader, chat and news client. This image runs it in a container, used from the browser.

This is a personal fork of [linuxserver/docker-thunderbird](https://github.com/linuxserver/docker-thunderbird), maintained independently. It is **not** a LinuxServer.io image and is not supported by the LinuxServer.io team: please don't report issues with it to them. It builds on their [baseimage-selkies](https://github.com/linuxserver/docker-baseimage-selkies), so everything the base image offers (PUID/PGID, `FILE__` secrets, umask, Docker Mods, GPU, the [Selkies docs](https://docs.linuxserver.io/selkies/)) applies here too.

Changes from upstream:

* Betterbird instead of Thunderbird, from the official Linux tarballs.
* Optional Betterbird language pack baked in at build time (`BETTERBIRD_LANG`).
* Built and published by GitHub Actions to `ghcr.io/mccio/betterbird`, no version pinned in the repo.

## Images

| Tag | Content |
| --- | --- |
| `latest`, `<version>` | English (en-US) |
| `latest-it`, `<version>-it` | Italian language pack |

`<version>` is the Betterbird version (e.g. `153.7.0esr-bb13`), with one [release](https://github.com/McCio/docker-betterbird/releases) per version. A daily check picks up new Betterbird releases. Every image is also rebuilt weekly for base-image updates, so a tag can move to a newer build of the same version (the build date is in the `build_version` label).

## Supported Architectures

Each tag is a multi-arch manifest: pulling `ghcr.io/mccio/betterbird:latest` retrieves the right image for your arch. Specific arch images are available via tags:

| Architecture | Available | Tag |
| :----: | :----: | ---- |
| x86-64 | ✅ | \<tag\>-amd64 |
| arm64 | ✅ | \<tag\>-arm64 |

## Application Setup

The application can be accessed at:

* https://yourhost:3001/

### Strict reverse proxies

This image uses a self-signed certificate by default. This naturally means the scheme is `https`.
If you are using a reverse proxy which validates certificates, you need to [disable this check for the container](https://docs.linuxserver.io/faq#strict-proxy).

This container is based on [Docker Baseimage Selkies](https://github.com/linuxserver/docker-baseimage-selkies), which streams a desktop or single application to your web browser. Only the essentials are covered below, the full platform documentation lives at [docs.linuxserver.io/selkies](https://docs.linuxserver.io/selkies/).

### Start Minimal

Large run configurations assembled from years of forum posts are the usual source of trouble. These containers are designed to work with almost nothing. Start with the minimal command below, confirm it works, then add your options (volumes, env vars, GPU flags) one at a time:

```bash
docker run --rm -it \
  --shm-size=1gb \
  -p 3001:3001 \
  ghcr.io/mccio/betterbird:latest bash
```

Open **https://localhost:3001** (note the `https`) and accept the self signed certificate warning. The trailing `bash` gives you a shell inside the container, `ctrl+d` exits and cleans everything up. If a bigger setup misbehaves, return to this command to isolate the problem, see the [Quickstart](https://docs.linuxserver.io/selkies/user-guide/quickstart/) and [Troubleshooting](https://docs.linuxserver.io/selkies/user-guide/troubleshooting/) guides.

### GPU Acceleration

A GPU is optional, the CPU pipeline is fast enough for smooth sessions on modest hardware. When a GPU is mounted into the container it is automatically detected and configured for zero copy rendering and encoding, set `AUTO_GPU=false` to disable this behavior.

**Intel / AMD (open source drivers), one flag is all it takes:**

```bash
docker run --rm -it \
  --shm-size=1gb \
  -p 3001:3001 \
  --device /dev/dri \
  ghcr.io/mccio/betterbird:latest bash
```

**Nvidia (proprietary drivers, not available on Alpine based images):**

Requires host driver **580 or newer** installed from the `.run` file downloaded directly from Nvidia. On drivers older than 595.80 you may also need kernel parameters and a dummy plug, see [GPU Acceleration](https://docs.linuxserver.io/selkies/user-guide/gpu/) before opening an issue.

```bash
docker run --rm -it \
  --shm-size=1gb \
  -p 3001:3001 \
  --runtime nvidia \
  --gpus all \
  --device /dev/nvidia-modeset \
  ghcr.io/mccio/betterbird:latest bash
```

Compose examples, multi GPU selection (`DRINODE`/`DRI_NODE`), Unraid notes, and a step by step debugging loop are in the [GPU Acceleration](https://docs.linuxserver.io/selkies/user-guide/gpu/) guide.

### Security

>[!WARNING]
>This container provides privileged access to the host system. Do not expose it to the Internet unless you have secured it properly.

**HTTPS is required.** Modern browser features used for video and audio (WebCodecs) only work in a secure context. Port `3001` serves HTTPS directly, the HTTP port exists only to sit behind a reverse proxy.

By default there is **no authentication**. Setting the `CUSTOM_USER` and `PASSWORD` environment variables enables HTTP basic auth, which is suitable only for a trusted local network. For internet exposure, place the container behind a reverse proxy with robust authentication, such as [SWAG](https://github.com/linuxserver/docker-swag). Keep in mind the web interface includes a terminal with passwordless `sudo`, anyone who can reach the GUI can gain root inside the container.

Authentication options, sharing link behavior, container isolation, and the lockdown variables for kiosk style deployments are covered in [Security and Hardening](https://docs.linuxserver.io/selkies/user-guide/security/).

### Language Support - Internationalization

To launch the session in a different language, set the `LC_ALL` environment variable, e.g. `-e LC_ALL=zh_CN.UTF-8` (Chinese), `ja_JP.UTF-8` (Japanese), `ko_KR.UTF-8` (Korean), `ar_AE.UTF-8` (Arabic), `ru_RU.UTF-8` (Russian), `es_MX.UTF-8` (Spanish Latin America), `de_DE.UTF-8` (German), `fr_FR.UTF-8` (French), `nl_NL.UTF-8` (Dutch), `it_IT.UTF-8` (Italian).

`LC_ALL` sets the desktop session's locale (dates, numbers, currency, file dialogs). Each image sets it to match its language pack (`en_US.UTF-8`, `it_IT.UTF-8` for the `-it` tags); `-e LC_ALL=...` overrides it. Betterbird's own UI language comes from the image's language pack. Other languages: build locally with `BETTERBIRD_LANG` and `BETTERBIRD_LOCALE` (see [Building locally](#building-locally)).

### Existing profile

The Betterbird profile lives in `/config/.config/thunderbird/` (with its `profiles.ini`). To bring an existing Thunderbird or Betterbird profile, stop the container and copy the profile folder there as is. It must come from the same or an older version.

### Beyond Running the App

This image inherits every capability of the Selkies platform, all documented in depth on the docs site:

* [Configuration Reference](https://docs.linuxserver.io/selkies/user-guide/configuration/) - every environment variable: ports, subfolder reverse proxy, custom dashboards, watermarking, and the full set of `SELKIES_*` stream, feature, and UI settings with their `|locked` syntax.
* [Using the Web Client](https://docs.linuxserver.io/selkies/user-guide/web-client/) - file transfer, clipboard sync, audio and microphone, up to four gamepads, session sharing, and mobile touch support.
* [Installing Applications](https://docs.linuxserver.io/selkies/user-guide/installing-apps/) - persistent user installed apps with [proot-apps](https://github.com/linuxserver/proot-apps) (`proot-apps install filezilla`) or native packages at boot with Docker mods.
* [Docker in Docker and host Docker control](https://docs.linuxserver.io/selkies/user-guide/configuration/#optional-run-configurations) - run `--privileged` for an isolated DinD setup, or mount `/var/run/docker.sock` to manage the host.
* [Reverse Proxy](https://docs.linuxserver.io/selkies/user-guide/reverse-proxy/) - proxying websockets and subfolders correctly.
* [SealSkin](https://docs.linuxserver.io/selkies/components/sealskin/) - turnkey multi user VDI orchestration with browser extension and mobile apps, this image is fully compatible.
* [Pelorus](https://docs.linuxserver.io/selkies/components/pelorus/) - expose the session to AI agents via a computer use API and accessibility stack.

## Usage

To help you get started creating a container from this image you can either use docker-compose or the docker cli.

>[!NOTE]
>Unless a parameter is flagged as 'optional', it is *mandatory* and a value must be provided.

### docker-compose (recommended, [click here for more info](https://docs.linuxserver.io/general/docker-compose))

```yaml
---
services:
  betterbird:
    image: ghcr.io/mccio/betterbird:latest
    container_name: betterbird
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - /path/to/config:/config
    ports:
      - 3000:3000
      - 3001:3001
    shm_size: "1gb"
    restart: unless-stopped
```

### docker cli ([click here for more info](https://docs.docker.com/engine/reference/commandline/cli/))

```bash
docker run -d \
  --name=betterbird \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Etc/UTC \
  -p 3000:3000 \
  -p 3001:3001 \
  -v /path/to/config:/config \
  --shm-size="1gb" \
  --restart unless-stopped \
  ghcr.io/mccio/betterbird:latest
```

## Parameters

Containers are configured using parameters passed at runtime (such as those above). These parameters are separated by a colon and indicate `<external>:<internal>` respectively. For example, `-p 8080:80` would expose port `80` from inside the container to be accessible from the host's IP on port `8080` outside the container.

| Parameter | Function |
| :----: | --- |
| `-p 3000:3000` | Betterbird desktop gui HTTP, must be proxied. |
| `-p 3001:3001` | Betterbird desktop gui HTTPS. |
| `-e PUID=1000` | for UserID - see below for explanation |
| `-e PGID=1000` | for GroupID - see below for explanation |
| `-e TZ=Etc/UTC` | specify a timezone to use, see this [list](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List). |
| `-v /config` | Users home directory in the container, stores program settings and files. |
| `--shm-size=` | Recommended for all desktop images. |

## Environment variables from files (Docker secrets)

You can set any environment variable from a file by using a special prepend `FILE__`.

As an example:

```bash
-e FILE__MYVAR=/run/secrets/mysecretvariable
```

Will set the environment variable `MYVAR` based on the contents of the `/run/secrets/mysecretvariable` file.

## Umask for running applications

The base image provides the ability to override the default umask settings for services started within the containers using the optional `-e UMASK=022` setting.
Keep in mind umask is not chmod it subtracts from permissions based on it's value it does not add. Please read up [here](https://en.wikipedia.org/wiki/Umask) before asking for support.

## User / Group Identifiers

When using volumes (`-v` flags), permissions issues can arise between the host OS and the container, the base image avoids this issue by allowing you to specify the user `PUID` and group `PGID`.

Ensure any volume directories on the host are owned by the same user you specify and any permissions issues will vanish like magic.

In this instance `PUID=1000` and `PGID=1000`, to find yours use `id your_user` as below:

```bash
id your_user
```

Example output:

```text
uid=1000(your_user) gid=1000(your_user) groups=1000(your_user)
```

## Docker Mods

LinuxServer.io publishes [Docker Mods](https://github.com/linuxserver/docker-mods) that add functionality to containers built on their base images. The [universal mods](https://mods.linuxserver.io/?mod=universal) work here too; the `thunderbird` image mods target upstream and are untested with this fork.

## Support Info

* Shell access whilst the container is running:

    ```bash
    docker exec -it betterbird /bin/bash
    ```

* To monitor the logs of the container in realtime:

    ```bash
    docker logs -f betterbird
    ```

* Container version number:

    ```bash
    docker inspect -f '{{ index .Config.Labels "build_version" }}' betterbird
    ```

* Image version number:

    ```bash
    docker inspect -f '{{ index .Config.Labels "build_version" }}' ghcr.io/mccio/betterbird:latest
    ```

## Updating Info

The image is static and versioned: updating Betterbird means pulling a new image and recreating the container. Updating the app inside the container is not supported.

Below are the instructions for updating containers:

### Via Docker Compose

* Update images:
    * All images:

        ```bash
        docker-compose pull
        ```

    * Single image:

        ```bash
        docker-compose pull betterbird
        ```

* Update containers:
    * All containers:

        ```bash
        docker-compose up -d
        ```

    * Single container:

        ```bash
        docker-compose up -d betterbird
        ```

* You can also remove the old dangling images:

    ```bash
    docker image prune
    ```

### Via Docker Run

* Update the image:

    ```bash
    docker pull ghcr.io/mccio/betterbird:latest
    ```

* Stop the running container:

    ```bash
    docker stop betterbird
    ```

* Delete the container:

    ```bash
    docker rm betterbird
    ```

* Recreate a new container with the same docker run parameters as instructed above (if mapped correctly to a host folder, your `/config` folder and settings will be preserved)
* You can also remove the old dangling images:

    ```bash
    docker image prune
    ```

### Image Update Notifications - Diun (Docker Image Update Notifier)

>[!TIP]
>[Diun](https://crazymax.dev/diun/) works for update notifications. Tools that update containers unattended are not recommended.

## Building locally

If you want to make local modifications to these images for development purposes or just to customize the logic:

```bash
git clone https://github.com/McCio/docker-betterbird.git
cd docker-betterbird
docker build \
  --no-cache \
  --pull \
  --build-arg BETTERBIRD_VERSION=$(curl -sI 'https://www.betterbird.eu/downloads/get.php?os=linux&lang=en-US&version=release' | sed -n 's|.*/betterbird-\(.*\)\.en-US\.linux-x86_64\.tar\.xz.*|\1|p') \
  --build-arg BETTERBIRD_LANG=en-US \
  -t betterbird:local .
```

`BETTERBIRD_VERSION` is required (the repo pins no version; CI resolves it the same way on each run, see `.github/workflows/publish.yml`). `BETTERBIRD_LANG` takes any Betterbird [language pack](https://www.betterbird.eu/downloads/) code (`it`, `de`, ...); pass the matching `BETTERBIRD_LOCALE` (`de_DE.UTF-8`, default `en_US.UTF-8`).

Use `-f Dockerfile.aarch64` for arm64. Cross-arch builds need QEMU (`docker run --rm --privileged lscr.io/linuxserver/qemu-static --reset`); building amd64 under QEMU on Apple silicon fails in `tar`, so CI builds each arch on a native runner.

## Versions

Betterbird versions: see [releases](https://github.com/McCio/docker-betterbird/releases).

* **19.11.26:** - Fork: Betterbird instead of Thunderbird, language packs, GitHub Actions publishing to GHCR.
* **31.05.26:** - (upstream) Rebase to resolute.
* **04.04.26:** - (upstream) Make Wayland default disable with PIXELFLUX_WAYLAND=false.
* **28.12.25:** - (upstream) Add Wayland init logic.
* **02.12.25:** - (upstream) Initial release.

## License

This repository is GPL-3.0, as upstream [linuxserver/docker-thunderbird](https://github.com/linuxserver/docker-thunderbird): see [LICENSE](LICENSE). The images also contain:

* [baseimage-selkies](https://github.com/linuxserver/docker-baseimage-selkies) (GPL-3.0), unmodified, source at that link.
* Betterbird and its language packs, downloaded unmodified from [betterbird.eu](https://www.betterbird.eu/). Betterbird is a fork of Mozilla Thunderbird, whose source is under the MPL-2.0; Betterbird's source is the Thunderbird ESR source plus the patches and build scripts at [Betterbird/thunderbird-patches](https://github.com/Betterbird/thunderbird-patches).
