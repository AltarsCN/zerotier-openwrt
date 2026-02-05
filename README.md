# ZeroTier OpenWrt Package with Controller Support

This is a custom OpenWrt package feed for ZeroTier with built-in network controller support (`ZT_CONTROLLER=1`).

The official OpenWrt/ImmortalWrt zerotier package does not include controller support by default. This feed provides a replacement package with the controller enabled.

## Pre-built Packages

Pre-built packages are available in [GitHub Releases](https://github.com/AltarsCN/zerotier-openwrt/releases).

### Supported Architectures

| Architecture | Platform | Devices |
|-------------|----------|---------|
| `x86_64` | ImmortalWrt | 软路由、虚拟机 |
| `aarch64_cortex-a53` | ImmortalWrt | Rockchip ARM64 设备 (R2S/R4S/R5S 等) |
| `qualcommax_ipq60xx` | ImmortalWrt | 京东云 AX1800/AX6000、GL-AX1800 等 |
| `qualcommax_ipq807x` | ImmortalWrt | 红米 AX6、Netgear RAX120 等 |

### Quick Install

```bash
# 下载对应架构的包
wget https://github.com/AltarsCN/zerotier-openwrt/releases/latest/download/zerotier_xxx_x86_64.ipk

# 安装
opkg install zerotier_*.ipk
# 或 (ImmortalWrt 24.10+)
apk add zerotier_*.apk
```

## Features

- **Built-in Network Controller**: Create and manage your own ZeroTier networks locally
- **No Cloud Dependency**: Host your own controller without relying on ZeroTier Central
- **Compatible**: Works with luci-app-zerotier controller UI

## Quick Start

### Method 1: Add as Custom Feed (Recommended)

1. Add this repository as a custom feed in your OpenWrt buildroot:

```bash
# Edit feeds.conf.default or feeds.conf
echo "src-git zerotier_custom https://github.com/YOUR_USERNAME/zerotier-openwrt.git" >> feeds.conf
```

2. Update and install the feeds:

```bash
./scripts/feeds update -a
./scripts/feeds install -a -f  # -f to force override the official zerotier
```

3. Configure the build:

```bash
make menuconfig
# Navigate to: Network -> VPN -> zerotier
# Enable: [*] Enable built-in network controller
```

4. Build the package:

```bash
make package/zerotier/compile V=s
```

5. Find the IPK in `bin/packages/<arch>/zerotier_custom/`

### Method 2: Build with ImmortalWrt imagebuilder

1. Download the ImageBuilder for your target from [ImmortalWrt releases](https://downloads.immortalwrt.org/)

2. Add this package to the build:

```bash
# Extract ImageBuilder
tar xf immortalwrt-imagebuilder-*.tar.xz
cd immortalwrt-imagebuilder-*

# Add custom feed
echo "src-git zerotier_custom https://github.com/YOUR_USERNAME/zerotier-openwrt.git" >> feeds.conf

./scripts/feeds update zerotier_custom
./scripts/feeds install -f zerotier

# Build image with the custom zerotier
make image PACKAGES="zerotier luci-app-zerotier"
```

### Method 3: Manual Build in SDK

1. Download the OpenWrt SDK for your target

2. Clone this repository:

```bash
cd sdk/
git clone https://github.com/YOUR_USERNAME/zerotier-openwrt.git package/zerotier-openwrt
```

3. Build:

```bash
make package/zerotier-openwrt/net/zerotier/compile V=s
```

## Configuration Options

In `make menuconfig` under Network -> VPN -> zerotier:

| Option | Description | Default |
|--------|-------------|---------|
| Enable built-in network controller | Enables ZT_CONTROLLER=1 for local network management | Yes |
| Build in debug mode | Enables debug symbols and logging | No |
| Build self-test program | Builds zerotier-selftest diagnostic tool | No |

## Controller API Usage

Once installed, the controller API is available at:

```
http://localhost:9993/controller
```

You need the auth token from `/var/lib/zerotier-one/authtoken.secret`

### Example: Create a Network

```bash
# Get auth token
TOKEN=$(cat /var/lib/zerotier-one/authtoken.secret)
NODE_ID=$(zerotier-cli info | awk '{print $3}')

# Create a new network
curl -X POST -H "X-ZT1-Auth: $TOKEN" \
  http://localhost:9993/controller/network/${NODE_ID}______

# List networks
curl -H "X-ZT1-Auth: $TOKEN" \
  http://localhost:9993/controller/network
```

## Using with luci-app-zerotier

This package is designed to work with [luci-app-zerotier](https://github.com/YOUR_USERNAME/luci-app-zerotier) which provides a web UI for the built-in controller.

After installing both packages:
1. Go to Services → ZeroTier → Network Controller
2. Create and manage networks through the web interface

## Compatibility

Tested with:
- ImmortalWrt 23.05+
- OpenWrt 23.05+
- ZeroTier 1.14.0 - 1.16.0

Supported architectures:
- x86_64
- aarch64 (arm64)
- armv7
- mipsel

## License

This package follows the same license as the original OpenWrt/ImmortalWrt zerotier package (GPL-2.0).

ZeroTier itself is licensed under the Mozilla Public License 2.0.

## Contributing

Issues and pull requests are welcome!

## Related Projects

- [ZeroTier Official](https://github.com/zerotier/ZeroTierOne)
- [OpenWrt Packages](https://github.com/openwrt/packages)
- [ImmortalWrt Packages](https://github.com/immortalwrt/packages)
