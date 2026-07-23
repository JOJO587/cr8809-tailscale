# cr8809-tailscale

Tailscale 1.98.9 精简构建（单二进制），面向 **CR8809（Redmi AX3000）/ IPQ50xx 平台**路由器，运行在 **OpenWrt 21.02.7** 上。

> 通用适用说明：任何 ARMv7（Cortex-A7）的 OpenWrt 路由器理论上都可使用，但本构建仅在我自己的 CR8809 上实测验证过。

## 特性

- **单二进制**：`tailscaled` 内嵌 CLI（`ts_include_cli`），`tailscale` 用软链复用同一文件
- **静态链接**：`CGO_ENABLED=0`，`GOARM=7`，无需外部 libc
- **体积**：解压后约 13.4MB，压缩包 ~5.4MB（适合空间紧张的 overlay 分区）
- **TUN 模式** + **subnet router**（需 `kmod-tun`）
- 构建 featuretags：

  ```
  --min --add=osrouter,iptables,cli,unixsocketidentity,advertiseroutes,useroutes,relayserver,portmapper,ipnbus
  ```

## 支持硬件 / 系统

| 项目 | 说明 |
|---|---|
| 架构 | ARMv7（Cortex-A7，GOARM=7） |
| 实测设备 | CR8809（Redmi AX3000）、IPQ5000 平台 |
| OpenWrt | 21.02.7（iptables / firewall3 时代） |
| 依赖 | `kmod-tun`（TUN 模式必需）、`iptables` |

> 其他 armv7 OpenWrt 设备可尝试，但需自行验证；24.10（nftables）未测试。

## 部署简述

1. 将 `tailscaled` 放到持久化目录（如 `/overlay/root/ts/`），`chmod +x`
2. 建软链 `ln -sf tailscaled tailscale`
3. 用 procd init 脚本托管（开机自启 + 崩溃自愈）
4. `tailscale up --advertise-routes=<你的局域网段> --accept-dns=false` → 浏览器登录
5. **在 Tailscale admin console 手动批准子网路由**后才生效

## 已知问题

- ⚠️ `tailscale status` 可能显示 `offline`，但**引擎实际在线**。判断真实状态以 daemon `--verbose` 日志为准（日志应出现 `Switching ipn state Starting -> Running`、`wgengine: Reconfig ... peers`）。
- 精简构建省略了部分状态上报组件，状态显示不完整属正常现象。
- 子网路由必须经 admin console 批准，否则 tailnet 内其他设备无法访问该局域网。

## 说明

本仓库仅为个人构建分享，与 Tailscale 官方无关。构建由 GitHub Actions 完成，产物见 Releases。
