# RustDesk-261106

RustDesk v1.5.1 自定义编译版本，支持通过 GitHub 仓库环境变量在编译时注入自定义服务器、密钥、密码等配置。

## 快速开始

### 1. 配置环境变量

进入仓库 **Settings → Secrets and variables → Actions**：

#### Variables（非敏感配置）

点击 **Variables → Add variable**，添加以下变量（按需配置，不添加则使用默认值）：

| Name | 示例值 | 说明 |
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

#### Secrets（敏感配置）

点击 **Secrets → Add secret**，添加以下密钥：

| Name | 示例值 | 说明 |
|---|---|---|
| `RUSTDESK_RS_PUB_KEY` | `base64编码的公钥` | 服务器公钥 |
| `RUSTDESK_KEY` | `base64编码的公钥` | 连接密钥 |
| `RUSTDESK_PRESET_PASSWORD` | `你的密码` | 预设永久密码 |

### 2. 触发编译

有两种方式触发编译：

#### 方式一：手动触发
1. 进入仓库 **Actions** 页面
2. 选择 **Custom Build** workflow
3. 点击 **Run workflow**
4. 选择编译目标（linux-x64 / windows-x64 / all）
5. 点击 **Run workflow** 按钮

#### 方式二：推送 Tag 自动触发
```bash
git tag v1.5.1
git push origin v1.5.1
```

### 3. 下载编译产物

- **手动触发**：在 Actions 运行结果页的 **Artifacts** 区域下载
- **Tag 触发**：在 Releases 页面下载（自动上传到 Release）

## 环境变量完整说明

详见 [ENV_VARS.md](ENV_VARS.md)

| 环境变量 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `RUSTDESK_RENDEZVOUS_SERVERS` | Variable | `rs-ny.rustdesk.com` | 会合服务器地址 |
| `RUSTDESK_CUSTOM_SERVER` | Variable | 无 | 自定义中继服务器 |
| `RUSTDESK_API_SERVER` | Variable | 从配置推导 | API 服务器地址 |
| `RUSTDESK_RENDEZVOUS_PORT` | Variable | `21116` | 会合端口 |
| `RUSTDESK_RELAY_PORT` | Variable | `21117` | 中继端口 |
| `RUSTDESK_WS_RENDEZVOUS_PORT` | Variable | `21118` | WS 会合端口 |
| `RUSTDESK_WS_RELAY_PORT` | Variable | `21119` | WS 中继端口 |
| `RUSTDESK_DISABLE_UPDATE` | Variable | 不禁用 | 禁用更新检查 |
| `RUSTDESK_UPDATE_URL` | Variable | `https://api.rustdesk.com/version/latest` | 更新 URL |
| `RUSTDESK_RS_PUB_KEY` | Secret | 官方公钥 | 服务器公钥 |
| `RUSTDESK_KEY` | Secret | 同 RS_PUB_KEY | 连接密钥 |
| `RUSTDESK_PRESET_PASSWORD` | Secret | 无 | 预设密码 |

## 技术原理

所有环境变量通过 Rust 的 `option_env!()` 宏在**编译时**读取并嵌入二进制文件：

```rust
// 编译时读取，不是运行时
option_env!("RUSTDESK_RENDEZVOUS_SERVERS")
    .unwrap_or("rs-ny.rustdesk.com")
```

这意味着：
- ✅ 编译后的程序不依赖运行时环境变量
- ✅ 配置值直接嵌入二进制文件，用户无法提取修改
- ✅ 适合通过 GitHub Actions CI/CD 自动化编译分发

## 与原版 RustDesk 的区别

1. `libs/hbb_common` 从 git submodule 改为本地模块
2. 12 个硬编码常量改为 `option_env!()` 编译时注入
3. 新增 `custom-build.yml` workflow 支持从仓库设置读取配置
