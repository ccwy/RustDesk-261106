# RustDesk 编译时环境变量配置说明

本文档列出所有通过 `option_env!()` 宏在**编译时**注入的配置项。这些环境变量在 GitHub Actions 打包时从构建环境读取，直接嵌入编译后的二进制文件，运行时无法修改。

## 使用方式

### GitHub Actions 中注入

在 `.github/workflows/build.yml` 的 build step 中设置环境变量：

```yaml
- name: Build RustDesk
  run: cargo build --release
  env:
    RUSTDESK_RENDEZVOUS_SERVERS: "hbbs.example.com"
    RUSTDESK_RS_PUB_KEY: "your-base64-public-key"
    RUSTDESK_PRESET_PASSWORD: "your-secure-password"
    RUSTDESK_DISABLE_UPDATE: "1"
```

### GitHub Repository Secrets/Variables 中注入

在 GitHub 仓库 Settings → Secrets and variables → Actions 中添加变量，然后在 workflow 中引用：

```yaml
- name: Build RustDesk
  run: cargo build --release
  env:
    RUSTDESK_RENDEZVOUS_SERVERS: ${{ vars.RUSTDESK_RENDEZVOUS_SERVERS }}
    RUSTDESK_RS_PUB_KEY: ${{ secrets.RUSTDESK_RS_PUB_KEY }}
    RUSTDESK_PRESET_PASSWORD: ${{ secrets.RUSTDESK_PRESET_PASSWORD }}
    RUSTDESK_DISABLE_UPDATE: "1"
```

### 本地编译时注入

```bash
RUSTDESK_RENDEZVOUS_SERVERS=hbbs.example.com \
RUSTDESK_RS_PUB_KEY=your-base64-public-key \
RUSTDESK_DISABLE_UPDATE=1 \
cargo build --release
```

> **注意**: 这些变量在 `cargo build` / `rustc` 编译时读取，写入二进制文件后不再依赖运行时环境。不设置时使用代码中的默认值。

---

## 服务器相关

### RUSTDESK_RENDEZVOUS_SERVERS
- **用途**: 默认 rendezvous（会合）服务器地址列表
- **格式**: 逗号分隔的域名列表，例如 `my-server1.example.com,my-server2.example.com`
- **默认值**: `rs-ny.rustdesk.com`
- **注入位置**: `libs/hbb_common/src/config.rs`

### RUSTDESK_CUSTOM_SERVER
- **用途**: 自定义中继/会合服务器地址，覆盖 `custom-rendezvous-server` 配置项
- **格式**: 域名或 IP 地址，可带端口，例如 `my-server.example.com:21116`
- **默认值**: 无（空则使用配置文件中的值）
- **注入位置**: `src/common.rs` (`get_custom_rendezvous_server`)

### RUSTDESK_API_SERVER
- **用途**: API 服务器地址，覆盖 `api-server` 配置项
- **格式**: URL，例如 `https://api.example.com` 或 `http://192.168.1.100:21114`
- **默认值**: 无（空则从配置或 rendezvous 服务器推导）
- **注入位置**: `src/common.rs` (`get_api_server`)

## 端口相关

### RUSTDESK_RENDEZVOUS_PORT
- **用途**: Rendezvous（会合）服务器端口
- **格式**: 整数端口号
- **默认值**: `21116`
- **注入位置**: `libs/hbb_common/src/config.rs`

### RUSTDESK_RELAY_PORT
- **用途**: Relay（中继）服务器端口
- **格式**: 整数端口号
- **默认值**: `21117`
- **注入位置**: `libs/hbb_common/src/config.rs`

### RUSTDESK_WS_RENDEZVOUS_PORT
- **用途**: WebSocket rendezvous 端口
- **格式**: 整数端口号
- **默认值**: `21118`
- **注入位置**: `libs/hbb_common/src/config.rs`

### RUSTDESK_WS_RELAY_PORT
- **用途**: WebSocket relay 端口
- **格式**: 整数端口号
- **默认值**: `21119`
- **注入位置**: `libs/hbb_common/src/config.rs`

## 密钥相关

### RUSTDESK_RS_PUB_KEY
- **用途**: Rendezvous 服务器的公钥（base64 编码）
- **格式**: base64 编码的公钥字符串
- **默认值**: `OeVuKk5nlHiXp+APNn0Y3pC1Iwpwn44JGqrQCsWqmBw=`（RustDesk 官方公钥）
- **注入位置**: `libs/hbb_common/src/config.rs`

### RUSTDESK_KEY
- **用途**: 连接服务器时使用的 key，覆盖配置文件中的 `key` 选项
- **格式**: base64 编码的公钥字符串
- **默认值**: 无（空则使用 `RUSTDESK_RS_PUB_KEY` 或配置文件中的值）
- **注入位置**: `src/common.rs`

## 密码相关

### RUSTDESK_PRESET_PASSWORD
- **用途**: 预设永久密码（明文），用于自定义版本的预设密码功能
- **格式**: 明文密码字符串
- **默认值**: 无（空则使用 HARD_SETTINGS 中的编译时预设密码）
- **安全提示**: 此密码以明文形式编译进二进制文件，知道此密码的人可以完全控制设备
- **注入位置**: `libs/hbb_common/src/config.rs` (`get_preset_password_storage_and_salt`)
- **优先级**: 高于 HARD_SETTINGS 中的编译时设置

## 更新检查相关

### RUSTDESK_DISABLE_UPDATE
- **用途**: 禁用软件更新检查
- **格式**: `1` 或 `true`（不区分大小写）表示禁用，不设置表示不禁用
- **默认值**: 未设置（不禁用）
- **注入位置**: `src/common.rs` (`check_software_update`), `src/updater.rs` (`check_update`, `check_update_as_root`)

### RUSTDESK_UPDATE_URL
- **用途**: 软件更新检查的 URL
- **格式**: URL 地址
- **默认值**: `https://api.rustdesk.com/version/latest`
- **注入位置**: `libs/hbb_common/src/lib.rs` (`version_check_request`)

---

## 完整 GitHub Actions 示例

```yaml
name: Build Custom RustDesk

on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Install Rust
        uses: dtolnay/rust-toolchain@stable
      
      - name: Build RustDesk
        run: cargo build --release
        env:
          # 服务器配置
          RUSTDESK_RENDEZVOUS_SERVERS: ${{ vars.RENDEZVOUS_SERVERS }}
          RUSTDESK_CUSTOM_SERVER: ${{ vars.CUSTOM_SERVER }}
          RUSTDESK_API_SERVER: ${{ vars.API_SERVER }}
          
          # 端口配置
          RUSTDESK_RENDEZVOUS_PORT: ${{ vars.RENDEZVOUS_PORT }}
          RUSTDESK_RELAY_PORT: ${{ vars.RELAY_PORT }}
          
          # 密钥配置（使用 Secrets）
          RUSTDESK_RS_PUB_KEY: ${{ secrets.RS_PUB_KEY }}
          RUSTDESK_KEY: ${{ secrets.RS_KEY }}
          RUSTDESK_PRESET_PASSWORD: ${{ secrets.PRESET_PASSWORD }}
          
          # 禁用更新检查
          RUSTDESK_DISABLE_UPDATE: "1"
      
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: rustdesk-custom
          path: target/release/rustdesk
```

## 自建服务器完整配置示例

在 GitHub 仓库 Settings → Secrets and variables → Actions 中设置：

**Variables（非敏感）:**
| Name | Value |
|---|---|
| `RENDEZVOUS_SERVERS` | `hbbs.example.com` |
| `CUSTOM_SERVER` | `hbbs.example.com` |
| `API_SERVER` | `http://hbbs.example.com:21114` |
| `RENDEZVOUS_PORT` | `21116` |
| `RELAY_PORT` | `21117` |

**Secrets（敏感）:**
| Name | Value |
|---|---|
| `RS_PUB_KEY` | `your-server-public-key-base64` |
| `RS_KEY` | `your-server-public-key-base64` |
| `PRESET_PASSWORD` | `your-secure-password` |

## 修改说明

### hbb_common 模块
原项目中 `libs/hbb_common` 是一个 git submodule，指向 `https://github.com/rustdesk/hbb_common`。
已将其改为本地模块，所有代码直接包含在主仓库中，不再依赖外部 git 仓库。

### 常量改为 lazy_static + option_env!
`libs/hbb_common/src/config.rs` 中的以下常量已从 `const` 改为 `lazy_static!` + `option_env!()` 编译时注入：
- `RENDEZVOUS_SERVERS`: `&[&str]` → `Vec<String>`
- `RS_PUB_KEY`: `&str` → `String`
- `RENDEZVOUS_PORT`: `i32` (const) → `i32` (lazy_static)
- `RELAY_PORT`: `i32` (const) → `i32` (lazy_static)
- `WS_RENDEZVOUS_PORT`: `i32` (const) → `i32` (lazy_static)
- `WS_RELAY_PORT`: `i32` (const) → `i32` (lazy_static)

所有调用点已相应更新（添加 `*` 解引用或 `.as_str()` / `.clone()` 调用）。

### option_env! vs std::env::var
- `option_env!("VAR")` — **编译时**读取环境变量，嵌入二进制文件，运行时不可更改
- `std::env::var("VAR")` — 运行时读取环境变量，需要运行环境设置

本项目使用 `option_env!()`，适合 GitHub Actions CI/CD 打包场景。
