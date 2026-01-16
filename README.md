# RiDDiX/docker-creality-slicer

[![GitHub Stars](https://img.shields.io/github/stars/RiDDiX/docker-creality-slicer-master.svg?style=for-the-badge&logo=github)](https://github.com/RiDDiX/docker-creality-slicer-master)
[![GitHub Release](https://img.shields.io/github/release/RiDDiX/docker-creality-slicer-master.svg?style=for-the-badge&logo=github)](https://github.com/RiDDiX/docker-creality-slicer-master/releases)
[![GitHub Package](https://img.shields.io/badge/ghcr.io-RiDDiX%2Fcreality--slicer-blue?style=for-the-badge&logo=github)](https://github.com/RiDDiX/docker-creality-slicer-master/pkgs/container/creality-slicer)
[![Docker Build](https://img.shields.io/github/actions/workflow/status/RiDDiX/docker-creality-slicer-master/docker-build.yml?style=for-the-badge&logo=github-actions)](https://github.com/RiDDiX/docker-creality-slicer-master/actions)

> **Docker container for [Creality Print](https://github.com/CrealityOfficial/CrealityPrint)** with Intel/Nvidia GPU support, stability improvements, and **automatic updates on every container restart**.

[Creality Print](https://github.com/CrealityOfficial/CrealityPrint) is Creality's official slicer for FDM 3D printers. This Docker container provides a web-based GUI to run Creality Print in your browser.

[![Creality](https://avatars.githubusercontent.com/u/44064182?s=200&v=4)](https://github.com/CrealityOfficial/CrealityPrint)

## Features

- **Automatic Updates**: Creality Print is automatically updated to the latest version on every container restart
- **Intel iGPU Optimization**: Pre-installed VA-API drivers, Mesa optimizations, and Intel-specific environment variables
- **Nvidia GPU Support**: Full support for Nvidia GPUs with hardware acceleration
- **Stability Watchdog**: Automatic detection and recovery from Creality Print hangs
- **Memory Management**: Prevents freezes during settings changes
- **Shader Cache Management**: Automatic cleanup to prevent memory bloat
- **Web-based GUI**: Access Creality Print from any browser via HTTPS

## Quick Start

### Docker Compose (Recommended)

```yaml
services:
  crealityprint:
    image: ghcr.io/riddix/creality-slicer:latest
    container_name: crealityprint
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Berlin
      - DRINODE=/dev/dri/renderD128
      - DRI_NODE=/dev/dri/renderD128
    volumes:
      - ./config:/config
    ports:
      - 3000:3000
      - 3001:3001
    devices:
      - /dev/dri:/dev/dri
    shm_size: 2gb
    restart: unless-stopped
```

### Docker CLI

```bash
docker run -d \
  --name=crealityprint \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/Berlin \
  -e DRINODE=/dev/dri/renderD128 \
  -e DRI_NODE=/dev/dri/renderD128 \
  -p 3000:3000 \
  -p 3001:3001 \
  -v /path/to/config:/config \
  --device /dev/dri:/dev/dri \
  --shm-size=2g \
  --restart unless-stopped \
  ghcr.io/riddix/creality-slicer:latest
```

### Nvidia GPU Support

```yaml
services:
  crealityprint:
    image: ghcr.io/riddix/creality-slicer:latest
    container_name: crealityprint
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Berlin
      - NVIDIA_VISIBLE_DEVICES=all
      - NVIDIA_DRIVER_CAPABILITIES=all
    volumes:
      - ./config:/config
    ports:
      - 3000:3000
      - 3001:3001
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
    shm_size: 2gb
    restart: unless-stopped
```

## Access

The application can be accessed at:
- **HTTPS (recommended)**: https://yourhost:3001/
- **HTTP**: http://yourhost:3000/

## Auto-Update Feature

This container automatically checks for and installs the latest Creality Print version **on every container restart**. The update process:

1. Checks the latest release on GitHub
2. Compares with installed version
3. Downloads and extracts the new AppImage if an update is available
4. Creality Print starts with the latest version

No manual intervention required - just restart your container to get updates!

## Intel iGPU Optimization

This container is optimized for Intel integrated GPUs. The following optimizations are included:

### Pre-installed Drivers
- `intel-media-va-driver` - Modern Intel VA-API driver (iHD)
- `i965-va-driver` - Legacy Intel VA-API driver
- Mesa Vulkan and OpenGL drivers

### Environment Variables (Auto-configured)
| Variable | Default | Description |
|----------|---------|-------------|
| `MESA_LOADER_DRIVER_OVERRIDE` | `iris` | Use Intel Iris driver |
| `LIBVA_DRIVER_NAME` | `iHD` | VA-API driver selection |
| `mesa_glthread` | `false` | Disable threaded GL for stability |

### Recommended Settings for Intel iGPU

```yaml
environment:
  - DRINODE=/dev/dri/renderD128
  - DRI_NODE=/dev/dri/renderD128
devices:
  - /dev/dri:/dev/dri
shm_size: 2gb
```

### Troubleshooting GPU Issues

| Issue | Solution |
|-------|----------|
| Creality Print hangs on settings change | The watchdog will auto-restart it. Increase `shm_size` to `4gb` |
| Older Intel GPU (pre-Skylake) | Add `-e MESA_LOADER_DRIVER_OVERRIDE=i965` |
| Disable GPU acceleration | Add `-e DISABLE_ZINK=true -e DISABLE_DRI3=true` |

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PUID` | `1000` | User ID for file permissions |
| `PGID` | `1000` | Group ID for file permissions |
| `TZ` | `Etc/UTC` | Timezone |
| `DRINODE` | - | GPU render node for DRI3 (e.g., `/dev/dri/renderD128`) |
| `DRI_NODE` | - | GPU render node for VA-API encoding |
| `CUSTOM_USER` | - | HTTP Basic auth username |
| `PASSWORD` | - | HTTP Basic auth password |
| `LC_ALL` | - | Locale (e.g., `de_DE.UTF-8`) |

## Volumes

| Path | Description |
|------|-------------|
| `/config` | Creality Print configuration and user data |

## Ports

| Port | Description |
|------|-------------|
| `3000` | HTTP (requires proxy for full functionality) |
| `3001` | HTTPS (recommended) |

## Building Locally

```bash
git clone https://github.com/RiDDiX/docker-creality-slicer-master.git
cd docker-creality-slicer-master
docker build -t ghcr.io/riddix/creality-slicer:latest .
```

## Credits

- Original OrcaSlicer container concept by [LinuxServer.io](https://linuxserver.io)
- [Creality Print](https://github.com/CrealityOfficial/CrealityPrint) by Creality
- Base image: [docker-baseimage-selkies](https://github.com/linuxserver/docker-baseimage-selkies)

## License

This project is licensed under the GPL-3.0 License - see the [LICENSE](LICENSE) file for details.

## Versions

- **16.01.26:** - Fork: Convert from OrcaSlicer to Creality Print with auto-update feature
- **15.01.26:** - Add Intel iGPU optimizations, VA-API drivers, stability watchdog, and memory management
- **01.01.26:** - Add wayland init
- **25.11.25:** - Update project repo name
- **15.09.25:** - Rebase to Ubuntu Noble and Selkies
