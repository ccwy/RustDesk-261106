# RustDesk-261106

RustDesk v1.5.1 自定义编译版本，支持通过 GitHub Actions Secrets 在编译时注入自定义服务器、密钥、密码等配置。

## 快速开始

### 1. 配置 Secrets

打开 https://github.com/ccwy/RustDesk-261106/settings/secrets/actions

点击 **Add secret**，按需添加以下变量（不添加的用默认值）：

| Secret Name | 示例值 | 说明 |
|---|---|---|
| `RUSTDESK_RENDEZVOUS_SERVERS` | `hbbs.example.com` | 会合服务器地址（多个用逗号分隔） |
| `RUSTDESK_API_SERVER` | `https://api.example.com` | API 服务器地址（独立 HTTPS 端口，无法从会合服务器推导） |
| `RUSTDESK_RENDEZVOUS_PORT` | `21116` | 会合服务器端口 |
| `RUSTDESK_RELAY_PORT` | `21117` | 中继服务器端口 |
| `RUSTDESK_RS_PUB_KEY` | `base64编码的公钥` | 服务器公钥 |
| `RUSTDESK_PRESET_PASSWORD` | `你的密码` | 预设永久密码 |

共 6 个变量。不配置的项使用默认值（官方 RustDesk 服务器）。

### 2. 端口自动推导

只需设置 `RENDEZVOUS_PORT` 和 `RELAY_PORT`，其余端口自动推导：

| 端口 | 计算方式 | 默认值 | 用途 |
|---|---|---|---|
| `RENDEZVOUS_PORT - 2` | 21114 | API 服务器 (hbbs HTTP API) |
| `RENDEZVOUS_PORT - 1` | 21115 | NAT 测试端口 |
| `RENDEZVOUS_PORT` | 21116 | 会合服务器主端口 (hbbs，打洞) |
| `RELAY_PORT` | 21117 | 中继服务器 (hbbt，转发) |
| `RENDEZVOUS_PORT + 2` | 21118 | WebSocket 会合端口 |
| `RENDEZVOUS_PORT + 3` | 21119 | WebSocket 中继端口 |

例如把 `RENDEZVOUS_PORT` 设为 `30016`、`RELAY_PORT` 设为 `30017`：

```
30014 = API
30015 = NAT 测试
30016 = 会合
30017 = 中继
30018 = WS 会合
30019 = WS 中继
```

### 3. 更新检查

更新检查已**永久禁用**，无需配置：
- `check_software_update()` 直接 return，不执行任何检查
- 更新检查 URL 硬编码为 `https://0.0.0.0/no-update`（不存在的地址）

### 4. 触发编译

打开 https://github.com/ccwy/RustDesk-261106/actions

1. 左侧选择 **Custom Build**
2. 点击 **Run workflow**
3. 填入版本号（如 `1.5.1`），选择目标平台
4. 点击绿色 **Run workflow** 按钮

### 5. 下载编译产物

编译完成后，在运行记录的 **Artifacts** 区域下载，或从 GitHub Release 页面下载。

## 环境变量完整说明

详见 [ENV_VARS.md](ENV_VARS.md)

所有变量统一使用 **Actions Secrets** 配置，编译时通过 `option_env!()` 宏注入二进制文件。

## 已移除的变量

以下变量已从代码中移除，不再需要配置：

| 变量 | 原因 |
|---|---|
| `RUSTDESK_DISABLE_UPDATE` | 更新已永久禁用，无需开关 |
| `RUSTDESK_UPDATE_URL` | URL 已硬编码为不存在的地址 |
| `RUSTDESK_KEY` | 与 `RUSTDESK_RS_PUB_KEY` 重复，不设时自动回退 |
| `RUSTDESK_CUSTOM_SERVER` | 与 `RUSTDESK_RENDEZVOUS_SERVERS` 重复 |
| `RUSTDESK_WS_RENDEZVOUS_PORT` | 代码未使用，由 `RENDEZVOUS_PORT + 2` 推导 |
| `RUSTDESK_WS_RELAY_PORT` | 代码未使用，由 `RENDEZVOUS_PORT + 3` 推导 |

## 技术原理

```
GitHub 仓库 Secrets
        ↓
GitHub Actions workflow (custom-build.yml)
        ↓  env: secrets.* 传入
flutter build linux --release
        ↓  option_env!() 编译时读取
编译后的二进制文件（配置已嵌入）
```
