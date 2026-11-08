# RustDesk-261106

RustDesk v1.5.1 自定义编译版本，支持通过 GitHub Actions Secrets 在编译时注入自定义服务器、密钥、密码等配置。

## 快速开始

### 1. 配置 Secrets

打开 https://github.com/ccwy/RustDesk-261106/settings/secrets/actions

点击 **Add secret**，按需添加以下变量（不添加的用默认值）：

| Secret Name | 示例值 | 说明 |
|---|---|---|
| `RUSTDESK_RENDEZVOUS_SERVERS` | `hbbs.example.com` | 会合服务器地址（多个用逗号分隔） |
| `RUSTDESK_CUSTOM_SERVER` | `hbbs.example.com` | 自定义中继服务器 |
| `RUSTDESK_API_SERVER` | `http://hbbs.example.com:21114` | API 服务器地址 |
| `RUSTDESK_RENDEZVOUS_PORT` | `21116` | 会合服务器端口 |
| `RUSTDESK_RELAY_PORT` | `21117` | 中继服务器端口 |
| `RUSTDESK_WS_RENDEZVOUS_PORT` | `21118` | WebSocket 会合端口 |
| `RUSTDESK_WS_RELAY_PORT` | `21119` | WebSocket 中继端口 |
| `RUSTDESK_DISABLE_UPDATE` | `1` | 禁用更新检查（`1` 或 `true`） |
| `RUSTDESK_UPDATE_URL` | `https://api.example.com/version/latest` | 更新检查 URL |
| `RUSTDESK_RS_PUB_KEY` | `base64编码的公钥` | 服务器公钥 |
| `RUSTDESK_KEY` | `base64编码的公钥` | 连接密钥 |
| `RUSTDESK_PRESET_PASSWORD` | `你的密码` | 预设永久密码 |

### 2. 触发编译

打开 https://github.com/ccwy/RustDesk-261106/actions

1. 左侧选择 **Custom Build**
2. 点击 **Run workflow**
3. 选择目标平台（linux-x64 / windows-x64 / all）
4. 点击绿色 **Run workflow** 按钮

### 3. 下载编译产物

编译完成后，在运行记录的 **Artifacts** 区域下载。

## 环境变量完整说明

详见 [ENV_VARS.md](ENV_VARS.md)

所有变量统一使用 **Actions Secrets** 配置，编译时通过 `option_env!()` 宏注入二进制文件。

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
