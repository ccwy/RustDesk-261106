# RustDesk 编译时环境变量配置说明

本文档列出通过 `option_env!()` 宏在**编译时**注入的配置项。在 GitHub Actions 打包时从 Secrets 读取，直接嵌入二进制文件。

## 更新检查：已永久禁用

更新检查已在代码中**永久禁用**，无需配置：
- `check_software_update()` 直接 return，不执行任何检查
- `check_update()` / `check_update_as_root()` 直接 return
- 更新检查 URL 硬编码为 `https://0.0.0.0/no-update`（不存在的地址）

## 可配置的环境变量（共 8 个）

在仓库 **Settings → Secrets and variables → Actions → Secrets** 中添加：

### 服务器相关

| Secret Name | 用途 | 默认值 |
|---|---|---|
| `RUSTDESK_RENDEZVOUS_SERVERS` | 会合服务器地址（逗号分隔多个） | `rs-ny.rustdesk.com` |
| `RUSTDESK_API_SERVER` | API 服务器地址 | 从配置推导 |

### 端口相关

| Secret Name | 用途 | 默认值 |
|---|---|---|
| `RUSTDESK_RENDEZVOUS_PORT` | 会合服务器端口 | `21116` |
| `RUSTDESK_RELAY_PORT` | 中继服务器端口 | `21117` |
| `RUSTDESK_WS_RENDEZVOUS_PORT` | WebSocket 会合端口 | `21118` |
| `RUSTDESK_WS_RELAY_PORT` | WebSocket 中继端口 | `21119` |

### 密钥与密码

| Secret Name | 用途 | 默认值 |
|---|---|---|
| `RUSTDESK_RS_PUB_KEY` | 服务器公钥（base64 编码） | RustDesk 官方公钥 |
| `RUSTDESK_PRESET_PASSWORD` | 预设永久密码（明文） | 无 |

## GitHub Actions 使用

在 workflow 的 build step 中设置 env：
```yaml
- name: Build
  run: flutter build linux --release
  env:
    RUSTDESK_RENDEZVOUS_SERVERS: ${{ secrets.RUSTDESK_RENDEZVOUS_SERVERS }}
    RUSTDESK_RS_PUB_KEY: ${{ secrets.RUSTDESK_RS_PUB_KEY }}
    RUSTDESK_PRESET_PASSWORD: ${{ secrets.RUSTDESK_PRESET_PASSWORD }}
```

## 已移除的变量

以下变量已从代码中移除，不再需要配置：

| 变量 | 原因 |
|---|---|
| `RUSTDESK_DISABLE_UPDATE` | 更新已永久禁用，无需开关 |
| `RUSTDESK_UPDATE_URL` | URL 已硬编码为不存在的地址 |
| `RUSTDESK_KEY` | 与 `RUSTDESK_RS_PUB_KEY` 重复，不设时自动回退到 RS_PUB_KEY |
| `RUSTDESK_CUSTOM_SERVER` | 与 `RUSTDESK_RENDEZVOUS_SERVERS` 重复，设了默认服务器即可 |
