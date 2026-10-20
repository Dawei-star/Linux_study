# 开发环境下载与安装指南

> 版本：0.0.1（参考清单）
> 日期：2026-10-20
> 目标机器：Lenovo Legion Y7000P IRH8 / Ubuntu 26.10 / i7-13700H / 16GB
> 文档状态：按需安装，未批量执行
> 关联文档：[系统升级方案](../../操作系统/系统升级方案/0.0.1.md)
> 首批执行方案：[第一批方案 0.0.1](第一批方案/0.0.1.md)
> 核心原则：官方来源、版本可固定、安装可回滚、稳定优先

---

## 1. 文档说明

本文档用于记录本机开发环境的下载来源、安装位置、安装方式、稳定性与性能表现，作为后续装机和迁移的参考清单。

收录原则：

1. 只收录官方渠道或官方维护的发行版，不使用破解版、绿色版和来路不明的“一键安装包”；
2. 优先选择可固定版本、可无残留卸载的方式；
3. “性能强”不等于“适合当前机器”，同时标注资源占用；
4. 本机为 Ubuntu 26.10 非 LTS，内核和桌面较新，稳定性判断以此为前提。

### 1.1 本机现状

数据来自 2026-10-20 的只读检查。

| 项目 | 现状 |
|---|---|
| 系统 | Ubuntu 26.10 / GNOME 51 / Wayland / Kernel 7.3.0-9 |
| CPU / 内存 | i7-13700H（14 核 20 线程）/ 16GB |
| 根分区 | `/dev/sda3`，183GB，已用 16GB，可用 158GB |
| 数据分区 | 595GB，**尚未挂载到 `/data`** |
| 已装运行时 | Git 2.55.0、Node.js 24.19.0、npm 11.16.0、Python 3.14.8 |
| 已装应用 | Qoder（`/opt/Qoder`）、OpenCode（`/opt/OpenCode`） |
| 尚未安装 | Java、Go、Rust、Docker、Podman、PostgreSQL、MySQL、Redis、CMake、tmux |
| 内存压力 | Qoder 约 2.6GB、Trae CN 约 2.2GB、Firefox 约 0.7GB（历史观测） |

> 在挂载 `/data` 之前，避免安装 Android SDK、大型容器镜像或大量数据集，根分区 158GB 看似充足，但 IDE、SDK 和镜像很容易占用数十 GB。

---

## 2. 评级口径

### 2.1 稳定性

| 评级 | 含义 |
|---|---|
| 高 | 官方长期维护，有 LTS 或稳定发布节奏，升级路径清晰，生态兼容性好 |
| 中高 | 官方维护，但发布节奏较快，偶有破坏性变更 |
| 中 | 新项目或快速迭代产品，接口/配置变化频繁，需要关注版本 |
| 低 | 不建议用于主要开发环境 |

### 2.2 性能与资源

| 指标 | 含义 |
|---|---|
| 性能：强 | 原生编译或高效运行时，CPU/IO 效率高，适合构建、服务端和高负载场景 |
| 性能：中 | 满足日常开发，存在解释执行、JIT 预热或 Electron 开销 |
| 性能：弱 | 启动慢或执行效率明显偏低，仅在特定场景使用 |
| 资源：轻 | 常驻内存通常低于 300MB，磁盘占用小 |
| 资源：中 | 常驻内存约 300MB–1GB，磁盘数百 MB 到数 GB |
| 资源：重 | 常驻内存 1GB 以上，磁盘数 GB 到数十 GB |

> IDE 的“性能”更多取决于索引速度和大项目响应，不能只看跑分；资源占用单独列出，避免在 16GB 机器上同时开多个重型 IDE。

---

## 3. 存储位置

### 3.1 路径约定

| 路径 | 用途 | 当前状态 |
|---|---|---|
| `/usr/bin`、`/usr/lib` | `apt` 安装的系统级工具与库 | 已使用 |
| `/opt/<name>` | 手动安装的大型应用（IDE、独立客户端） | 已有 Qoder、OpenCode |
| `/usr/local` | 官方 tar 包、源码编译安装（如 Go） | 预留 |
| `~/.local/bin` | 用户级 CLI 工具 | 预留 |
| `~/.local/share/<name>` | 用户级应用数据 | 预留 |
| `~/.config/<name>` | 应用配置 | 已使用 |
| `~/.cache/<name>` | 构建缓存、包缓存 | 已使用 |
| `~/Downloads/software/` | 安装包下载缓存（建议新建） | 尚未建立 |
| `~/.nvm` | Node 多版本 | 未安装 |
| `~/.sdkman` | Java 多版本 | 未安装 |
| `~/.rustup`、`~/.cargo` | Rust 工具链与包缓存 | 未安装 |
| `~/.local/share/uv`、`~/.cache/uv` | uv 管理的 Python 与缓存 | 未安装 |
| `/usr/local/go`、`~/go` | Go 工具链与模块缓存 | 未安装 |
| `/var/lib/docker` | Docker 镜像、容器、卷 | 未安装 |
| `/var/lib/postgresql`、`/var/lib/mysql` | 数据库数据目录 | 未安装 |
| `~/Android/Sdk` | Android SDK | 未安装 |
| `/data` | 大文件、镜像、备份、数据集 | **未挂载** |
| 项目内 `node_modules`、`.venv` | 项目依赖 | 随项目创建 |

### 3.2 下载缓存建议

安装包不要散落在 `~/Downloads` 根目录，建议按工具和版本归档：

```text
~/Downloads/software/
├── node/
│   └── v24.19.0/
├── vscode/
│   └── code_<版本>_amd64.deb
├── go/
│   └── go<版本>.linux-amd64.tar.gz
├── jetbrains-toolbox/
└── SHA256SUMS.txt
```

约定：

- 每个安装包保留对应版本的官方校验值；
- 大版本升级成功后，旧安装包保留一个版本即可；
- 容器镜像、Android SDK、模型文件不放在根分区，挂载 `/data` 后再安装；
- 下载目录不存放密钥、许可证文件和账号凭据。

### 3.3 空间分配建议

| 用途 | 建议空间 |
|---|---|
| 系统与 apt 软件 | 30–40GB |
| IDE 与工具链 | 10–20GB |
| 容器镜像与卷 | 20–50GB |
| 数据库数据 | 10GB 起，按项目增长 |
| Android SDK / 模拟器 | 20–40GB |
| 项目代码与依赖 | 20GB 起 |

根分区至少保留 20% 空闲，避免 ext4 碎片化和构建失败。

---

## 4. 安装方式对比

| 方式 | 典型工具 | 升级 | 隔离与回滚 | 稳定性 | 性能 | 说明 |
|---|---|---|---|---|---|---|
| Ubuntu 官方源 `apt` | Git、CMake、SQLite、tmux | `apt upgrade` | 卸载干净，系统集成好 | 高 | 原生 | 首选基础工具 |
| 软件官方 `apt` 源 | Docker、PostgreSQL、Temurin | `apt upgrade` | 较干净，需信任第三方密钥 | 高 | 原生 | 版本比 Ubuntu 源新 |
| 官方 `.deb` | VS Code、Bruno | 随 apt 升级 | 卸载方便 | 高 | 原生 | 桌面应用首选 |
| 官方 tar / 二进制 | Go、Android Studio、JetBrains | 手动替换目录 | 删目录即可回滚 | 高 | 原生 | 适合 `/opt`、`~/.local` |
| 版本管理器 | nvm、uv、SDKMAN、rustup、mise | 管理器切换 | 项目级隔离最好 | 高 | 接近原生 | 多版本、多项目必备 |
| 容器 | PostgreSQL、Redis、MySQL | 拉取新镜像 | 隔离最好，数据卷独立 | 高 | 接近原生（1–3% 开销） | 数据库和中间件推荐 |
| Snap / Flatpak | 部分 IDE、终端 | 自动更新 | 沙箱隔离 | 中高 | 启动偏慢、磁盘偏大 | 不作为开发工具首选 |
| 源码编译 | 特殊需求的库和工具 | 手动 | 可控但维护成本高 | 取决于上游 | 可优化 | 仅在必要时使用 |

> Linux 上不要安装 Docker Desktop：它通过虚拟机运行，额外占用内存和 IO；直接使用 Docker Engine 或 Podman。

---

## 5. 工具总览

### 5.1 基础工具

| 工具 | 官方下载 | 存储位置 | 安装方式 | 稳定性 | 性能 | 资源 | 建议 |
|---|---|---|---|---|---|---|---|
| Git | [git-scm.com](https://git-scm.com) | `/usr/bin/git` | `apt` | 高 | 强 | 轻 | 已装 2.55 |
| build-essential | Ubuntu 源 | `/usr` | `apt` | 高 | 强 | 轻 | 编译基础，必装 |
| CMake | [cmake.org](https://cmake.org) | `/usr/bin` | `apt` | 高 | 强 | 轻 | C/C++ 构建 |
| Ninja | [ninja-build.org](https://ninja-build.org) | `/usr/bin` | `apt` | 高 | 强 | 轻 | 比 Make 更快 |
| ripgrep | [GitHub Releases](https://github.com/BurntSushi/ripgrep) | `/usr/bin/rg` | `apt` | 高 | 强 | 轻 | 代码搜索 |
| fd | [GitHub Releases](https://github.com/sharkdp/fd) | `/usr/bin/fdfind` | `apt` | 高 | 强 | 轻 | 文件名搜索 |
| fzf | [GitHub Releases](https://github.com/junegunn/fzf) | `/usr/bin/fzf` | `apt` | 高 | 强 | 轻 | 模糊查找 |
| tmux | [GitHub Releases](https://github.com/tmux/tmux) | `/usr/bin/tmux` | `apt` | 高 | 强 | 轻 | 终端复用 |
| jq | [jqlang.github.io](https://jqlang.github.io/jq/) | `/usr/bin/jq` | `apt` | 高 | 强 | 轻 | JSON 处理 |
| Zsh | [zsh.org](https://www.zsh.org) | `/usr/bin/zsh` | `apt` | 高 | 强 | 轻 | 可选，配置成本较高 |

### 5.2 语言与运行时

| 工具 | 官方下载 | 存储位置 | 安装方式 | 稳定性 | 性能 | 资源 | 建议 |
|---|---|---|---|---|---|---|---|
| Node.js | [nodejs.org](https://nodejs.org) | `~/.nvm` 或 `/usr` | nvm / fnm / apt | 高 | 强 | 中 | 已装 24.19；多版本用 nvm |
| Python | [python.org](https://www.python.org) | `~/.local/share/uv` + 项目 `.venv` | uv / 系统包 | 高 | 中 | 中 | 已装 3.14.8；不要全局 pip |
| Java（Temurin） | [adoptium.net](https://adoptium.net) | `/usr/lib/jvm` 或 `~/.sdkman` | apt / SDKMAN | 高 | 强 | 重 | 选 21 或 25 LTS |
| Go | [go.dev/dl](https://go.dev/dl/) | `/usr/local/go`、`~/go` | 官方 tar | 高 | 强 | 轻 | 单二进制部署友好 |
| Rust | [rustup.rs](https://rustup.rs) | `~/.rustup`、`~/.cargo` | rustup | 高 | 强 | 中 | 编译慢，运行快 |
| mise | [mise.jdx.dev](https://mise.jdx.dev) | `~/.local/share/mise` | 官方脚本 / apt 源 | 中高 | 强 | 轻 | 统一管理多语言版本，较新 |

### 5.3 编辑器与 IDE

| 工具 | 官方下载 | 存储位置 | 安装方式 | 稳定性 | 性能 | 资源 | 建议 |
|---|---|---|---|---|---|---|---|
| VS Code | [code.visualstudio.com](https://code.visualstudio.com) | `/usr/share/code`、`~/.config/Code` | 官方 `.deb` | 高 | 中 | 中 | 轻量全能，默认推荐 |
| Qoder | 厂商官网 / 应用内更新 | `/opt/Qoder` | 官方安装包 | 中高 | 强（AI） | 重 | 已装，内存约 2.6GB |
| JetBrains Toolbox | [jetbrains.com/toolbox-app](https://www.jetbrains.com/toolbox-app/) | `~/.local/share/JetBrains` | 官方 tar | 高 | 强（索引/重构） | 重 | Java/Kotlin 大型项目 |
| Android Studio | [developer.android.com/studio](https://developer.android.com/studio) | `/opt/android-studio`、`~/Android/Sdk` | 官方 tar | 高 | 强 | 重 | 磁盘 10–30GB，建议放 `/data` |
| OpenCode | 厂商官网 | `/opt/OpenCode` | 官方安装包 | 中 | 中 | 中 | 已装 |
| Trae CN | 厂商官网 | `/opt` 或用户目录 | 官方安装包 | 中 | 中 | 重 | 当前未在 PATH，按需安装 |
| Neovim | [neovim.io](https://neovim.io) | `/usr/bin/nvim` | `apt` / AppImage | 高 | 强 | 轻 | 终端党，配置成本高 |

> 16GB 内存下不要同时运行 Qoder、Trae CN 和 JetBrains IDE；三者叠加很容易超过 8–10GB，再开浏览器和容器就会触发交换。

### 5.4 容器与虚拟化

| 工具 | 官方下载 | 存储位置 | 安装方式 | 稳定性 | 性能 | 资源 | 建议 |
|---|---|---|---|---|---|---|---|
| Docker Engine | [docs.docker.com](https://docs.docker.com/engine/install/ubuntu/) | `/var/lib/docker` | 官方 apt 源 | 高 | 强 | 中 | Linux 首选，不装 Desktop |
| Podman | [podman.io](https://podman.io) | `~/.local/share/containers` | `apt` | 高 | 强 | 中 | rootless，无守护进程 |
| kind | [kind.sigs.k8s.io](https://kind.sigs.k8s.io) | 容器内 | 官方二进制 | 高 | 中 | 中 | 本地 Kubernetes |
| k3d | [k3d.io](https://k3d.io) | 容器内 | 官方脚本 | 中高 | 中 | 中 | 轻量 K3s 集群 |
| Multipass | [multipass.run](https://multipass.run) | `/var/snap/multipass` | Snap | 高 | 中 | 重 | 需要完整 Ubuntu VM 时使用 |

### 5.5 数据库与缓存

| 工具 | 官方下载 | 存储位置 | 安装方式 | 稳定性 | 性能 | 资源 | 建议 |
|---|---|---|---|---|---|---|---|
| PostgreSQL | [postgresql.org](https://www.postgresql.org/download/linux/ubuntu/) | `/var/lib/postgresql` 或数据卷 | PGDG apt / 容器 | 高 | 强 | 中 | 首选关系型数据库 |
| MariaDB | [mariadb.org](https://mariadb.org/download/) | `/var/lib/mysql` 或数据卷 | apt / 容器 | 高 | 强 | 中 | MySQL 兼容，轻量 |
| MySQL | [dev.mysql.com](https://dev.mysql.com/downloads/) | `/var/lib/mysql` 或数据卷 | 官方 apt / 容器 | 高 | 强 | 中 | 生态广，注意许可证 |
| Redis | [redis.io](https://redis.io/downloads/) | `/var/lib/redis` 或数据卷 | apt / 容器 | 高 | 强 | 轻 | 缓存首选，注意许可证变更 |
| Valkey | [valkey.io](https://valkey.io) | `/var/lib/valkey` 或数据卷 | apt / 容器 | 高 | 强 | 轻 | Redis 的 BSD 分支，新项目可选 |
| SQLite | [sqlite.org](https://sqlite.org/download.html) | 单个 `.db` 文件 | `apt` | 高 | 强（读） | 轻 | 嵌入式、本地测试 |

### 5.6 API 调试与终端

| 工具 | 官方下载 | 存储位置 | 安装方式 | 稳定性 | 性能 | 资源 | 建议 |
|---|---|---|---|---|---|---|---|
| Bruno | [usebruno.com](https://www.usebruno.com) | `/opt/Bruno` 或 `~/Applications` | `.deb` / AppImage | 高 | 中 | 轻 | Git 友好、免费、无需账号 |
| Postman | [postman.com/downloads](https://www.postman.com/downloads/) | `/opt/Postman` | tar / AppImage | 高 | 中 | 重 | 功能全，Electron 占内存 |
| Insomnia | [insomnia.rest/download](https://insomnia.rest/download) | `/opt/Insomnia` | `.deb` / AppImage | 中 | 中 | 重 | 设计/调试，账号体系有变化 |
| curl | [curl.se](https://curl.se) | `/usr/bin/curl` | `apt` | 高 | 强 | 轻 | 脚本化首选 |
| HTTPie | [httpie.io](https://httpie.io) | `/usr/bin/http` | `apt` | 高 | 中 | 轻 | 可读性好的 CLI |
| Kitty | [sw.kovidgoyal.net/kitty](https://sw.kovidgoyal.net/kitty/) | `/usr/bin/kitty` | `apt` | 高 | 强（GPU） | 轻 | Wayland 支持好 |
| Alacritty | [alacritty.org](https://alacritty.org) | `/usr/bin/alacritty` | `apt` / cargo | 高 | 强（GPU） | 轻 | 极简、配置简单 |

---

## 6. 分类安装说明

### 6.1 基础工具（apt，一次装齐）

```bash
sudo apt update
sudo apt install -y \
  build-essential pkg-config cmake ninja-build \
  git curl wget unzip zip \
  ripgrep fd-find fzf tmux jq \
  sqlite3 httpie
```

验证：

```bash
git --version
cmake --version
rg --version
fdfind --version
sqlite3 --version
```

说明：

- `fd-find` 安装后的命令是 `fdfind`，可加别名 `alias fd=fdfind`；
- 这些工具都在系统路径，升级由 `apt` 统一管理；
- 稳定性高、性能原生、资源轻，建议全部安装。

---

### 6.2 Node.js

本机已有系统级 Node.js 24.19.0 和 npm 11.16.0，单版本项目可以直接使用。

多版本或多项目时，推荐 nvm 或 fnm：

```bash
# nvm 按官方 README 安装后
nvm install --lts
nvm use --lts
node -v
npm -v
```

说明：

- 版本管理器会把自身路径放到 `PATH` 前面，可能覆盖 `/usr/bin/node`，使用前用 `which node` 确认；
- pnpm/yarn 优先通过项目的 `packageManager` 字段和 corepack 管理，不要 `sudo npm install -g`；
- 存储位置：`~/.nvm`（每个版本约 200–500MB）、npm 缓存 `~/.npm`；
- 稳定性高（LTS），性能强（V8），常驻内存约 50–150MB/进程。

---

### 6.3 Python

系统已有 Python 3.14.8。Ubuntu 启用了 PEP 668，**不要**直接 `sudo pip install` 或 `pip install --user` 装项目依赖。

推荐 uv：

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv --version

# 项目内使用
uv venv --python 3.12
source .venv/bin/activate
uv pip install <包名>
# 或使用 pyproject.toml
uv sync
```

说明：

- 存储位置：二进制 `~/.local/bin/uv`，Python 版本 `~/.local/share/uv/python`，缓存 `~/.cache/uv`；
- 项目依赖放在项目内 `.venv`，不要跨项目共享；
- CPython 本身性能中等，数值计算依赖原生扩展；uv 的安装和解析速度明显快于 pip；
- 稳定性高，资源中等。

---

### 6.4 Java

需要 Java 时，优先 LTS 版本（21 或 25，按框架支持选择）。推荐 Temurin + SDKMAN：

```bash
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"

sdk list java
sdk install java 21.0.5-tem   # 示例，安装前用 sdk list java 确认最新 21.x
java -version
```

说明：

- 存储位置：`~/.sdkman/candidates/java/<版本>`，每个 JDK 约 300MB；
- 也可以用 Adoptium 官方 apt 仓库安装到 `/usr/lib/jvm`，适合只需要一个版本的情况；
- 稳定性高，JIT 性能强，但 JVM 常驻内存通常 0.5–2GB；
- 16GB 机器上同时开 JetBrains IDE、JVM 服务和浏览器时，注意堆内存上限。

---

### 6.5 Go

从 [go.dev/dl](https://go.dev/dl/) 下载当前稳定版 `linux-amd64` 包，安装到 `/usr/local`：

```bash
# 以下载的包名为准
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go<版本>.linux-amd64.tar.gz

echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.profile
source ~/.profile
go version
```

说明：

- 存储位置：工具链 `/usr/local/go`（约 250MB），模块缓存 `~/go/pkg/mod`，构建缓存 `~/.cache/go-build`，两者可能增长到数 GB；
- 不建议用 `apt install golang`，Ubuntu 源版本通常落后；
- 稳定性高，运行性能强，资源轻，适合 CLI、服务和云原生工具。

---

### 6.6 Rust

使用官方 rustup：

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
rustup default stable
rustc --version
cargo --version
```

说明：

- 存储位置：工具链 `~/.rustup`（约 1GB），包缓存 `~/.cargo`，项目 `target/` 可能数 GB；
- 运行性能强，编译时间和 CPU 占用较高；
- 稳定性高；如无明确需求，不必为了“性能”而选 Rust，开发成本也要计入。

---

### 6.7 VS Code

从官网下载 `linux-deb-x64` 包：

```bash
sudo apt install ./code_<版本>_amd64.deb
code --version
```

说明：

- 存储位置：程序 `/usr/share/code`，配置 `~/.config/Code`，扩展 `~/.vscode/extensions`；
- 扩展通常占用 1–3GB，建议定期清理不用的扩展；
- 稳定性高，性能中等（Electron），常驻内存约 300–800MB；
- 不建议用 Snap 版本，启动和文件监听在部分环境下有额外问题。

---

### 6.8 JetBrains Toolbox

从官网下载 Toolbox tar 包，解压到 `~/Applications` 或 `/opt`：

```bash
mkdir -p ~/Applications
tar -xzf jetbrains-toolbox-*.tar.gz -C ~/Applications
~/Applications/jetbrains-toolbox-*/jetbrains-toolbox
```

说明：

- Toolbox 自身管理 IDE 版本，IDE 安装在 `~/.local/share/JetBrains/Toolbox/apps`；
- 配置 `~/.config/JetBrains`，缓存 `~/.cache/JetBrains`；
- 稳定性高，大型项目索引和重构能力强，但内存通常 1.5–3GB；
- 16GB 机器建议一次只开一个 JetBrains IDE。

---

### 6.9 Docker Engine

按官方文档添加 GPG 密钥和 apt 仓库后安装：

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings

# GPG key 与仓库地址以官方文档为准，添加后执行：
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin

sudo systemctl enable --now docker
sudo usermod -aG docker $USER
# 重新登录后生效
docker run --rm hello-world
```

说明：

- 存储位置：`/var/lib/docker`，镜像和卷都占根分区，后续可迁移到 `/data`；
- 如果 Ubuntu 26.10 的 codename 尚未进入 Docker 官方仓库，可临时使用 Ubuntu 源：
  `sudo apt install docker.io docker-compose-v2`，版本较旧但稳定；
- 加入 `docker` 组等同于 root 权限，只对可信用户开放；
- 本地开发数据库端口绑定 `127.0.0.1`，不要默认暴露到 `0.0.0.0`；
- 性能接近原生，容器开销通常在 1–3%，不需要 Docker Desktop。

Podman 作为 rootless 替代：

```bash
sudo apt install -y podman
podman run --rm hello-world
```

存储位置为 `~/.local/share/containers` 和 `/var/lib/containers`，无守护进程，适合不希望常驻 root 服务的场景。

---

### 6.10 数据库与缓存

本地开发优先用容器，版本可控、卸载干净、不污染系统：

```bash
docker volume create pgdata
docker run -d --name pg \
  -e POSTGRES_PASSWORD='<强密码>' \
  -p 127.0.0.1:5432:5432 \
  -v pgdata:/var/lib/postgresql/data \
  postgres:17   # 示例，使用当前稳定大版本
```

Redis/Valkey 示例：

```bash
docker run -d --name cache \
  -p 127.0.0.1:6379:6379 \
  -v cache:/data \
  valkey/valkey:8   # 示例，生产环境固定版本
```

原生安装只建议在需要开机自启、长期驻留时使用：

```bash
sudo apt install -y postgresql postgresql-contrib   # Ubuntu 源版本
sudo apt install -y mariadb-server
sudo apt install -y redis-server
sudo apt install -y sqlite3
```

说明：

- PostgreSQL 稳定、功能完整、性能强，内存约 100–300MB，是默认选择；
- MariaDB 比 MySQL 更轻，兼容大部分 MySQL 用法；
- Redis 性能强但需注意许可证变化，新项目可评估 Valkey；
- SQLite 零配置、读性能强，适合本地工具和测试，不适合高并发写入；
- 容器数据务必使用命名卷或绑定挂载，删除容器前确认数据卷；
- 不要在生产环境使用 `latest` 标签，固定大版本和镜像摘要。

---

### 6.11 API 调试与终端

Bruno（推荐，轻量、免费、请求集合可进 Git）：

```bash
# 从官网下载 .deb
sudo apt install ./bruno_*_amd64.deb
```

或使用 AppImage：

```bash
mkdir -p ~/Applications
mv bruno_*.AppImage ~/Applications/
chmod +x ~/Applications/bruno_*.AppImage
```

终端：

```bash
sudo apt install -y kitty
# 或
sudo apt install -y alacritty
```

说明：

- Bruno 资源轻、无需账号，适合团队共享接口集合；
- Postman 功能全但 Electron 资源占用高，账号和云同步策略需评估；
- Kitty/Alacritty 在 Wayland 下支持良好，GPU 渲染，启动快；
- 终端配置放在 `~/.config/kitty/` 或 `~/.config/alacritty/`。

---

## 7. 推荐组合

### 7.1 按场景

| 场景 | 推荐组合 | 预期常驻内存 | 说明 |
|---|---|---|---|
| 轻量 Web 全栈 | Git + Node（nvm）+ Python（uv）+ VS Code + Docker + PostgreSQL/Redis + Bruno | 4–7GB | 最适合 16GB，稳定优先 |
| Java/Kotlin 后端 | Temurin 21/25 + JetBrains IDEA + Docker + PostgreSQL | 6–9GB | 不要同时开 Qoder/Trae |
| Go/Rust 服务端 | Go 或 Rust + VS Code/Neovim + Docker + PostgreSQL | 3–6GB | 工具链磁盘占用会增长 |
| AI 辅助开发 | Qoder 或 OpenCode + Node + Python + Docker | 5–8GB | Qoder 单项约 2.6GB，注意内存 |
| 移动端 | Android Studio + Android SDK + 模拟器 | 6–10GB | SDK/模拟器建议放 `/data` |
| 本地 Kubernetes | Docker/Podman + kind 或 k3d | 5–9GB | 集群和镜像很占内存/磁盘 |

### 7.2 16GB 内存预算

| 项目 | 建议上限 |
|---|---|
| 浏览器 | 2–3GB |
| 编辑器/IDE | 3–5GB（只开一个重型 IDE） |
| 语言服务/构建 | 1–2GB |
| 容器与数据库 | 2–4GB |
| 系统与其他 | 2–3GB |
| 余量 | 至少 2GB |

超出预算时优先关容器和模拟器，不要依赖交换分区；可参考系统升级方案启用 zram。

---

## 8. 下载与校验

1. **只用官方渠道**：优先官网、官方 GitHub Releases、官方 apt 仓库；不使用网盘转载和“精简版”。
2. **校验安装包**：

   ```bash
   sha256sum <安装包>
   # 与官网公布的 SHA256 逐字比对
   ```

3. **校验 apt 仓库**：GPG key 放入 `/etc/apt/keyrings`，不要用 `apt-key`。
4. **谨慎执行远程脚本**：

   ```bash
   curl -fsSL <官方脚本地址> -o install.sh
   less install.sh
   bash install.sh
   ```

   确认来源和内容后再执行，不要直接 `curl | sudo bash`。
5. **固定版本**：团队项目使用 `.nvmrc`、`.python-version`、`go.mod`、`rust-toolchain.toml` 或 mise 配置，避免“我这儿能跑”。
6. **不要用 sudo 装语言包**：`sudo npm -g`、`sudo pip` 会把文件写到系统目录，后续权限和升级问题多。

---

## 9. 卸载与回滚

| 安装方式 | 卸载方式 | 注意事项 |
|---|---|---|
| `apt` 包 | `sudo apt remove --purge <包名>` 后 `sudo apt autoremove` | 配置目录可能保留，按需删除 |
| 官方 `.deb` | 同 `apt` | 例如 `sudo apt remove code` |
| `/opt` 手动安装 | 删除对应目录和 `.desktop` 文件 | 先确认没有软链接指向它 |
| nvm | `nvm uninstall <版本>` | 删除 `~/.nvm` 会移除全部 Node 版本 |
| uv | 删除 `~/.local/bin/uv`、`~/.local/share/uv`、`~/.cache/uv` | 项目 `.venv` 需单独删除 |
| SDKMAN | `sdk uninstall java <版本>` | 删除 `~/.sdkman` 会移除全部 SDK |
| rustup | `rustup self uninstall` | 会同时移除 `~/.cargo` 中的工具，谨慎确认 |
| Docker | `sudo apt purge docker-ce ...` | `/var/lib/docker` 需手动删除，先备份卷 |
| 原生数据库 | `sudo apt purge postgresql*` | 数据目录不会自动删除，先备份 |
| Snap | `sudo snap remove <包名>` | 注意 `--purge` 会删除快照数据 |

---

## 10. 维护清单

```bash
# 系统包
sudo apt update && sudo apt upgrade
sudo apt autoremove --purge

# 缓存
uv cache clean
npm cache verify
go clean -cache
cargo cache -a   # 如已安装 cargo-cache

# 容器
docker system df
docker image prune          # 只删除悬空镜像
# docker system prune -a    # 会删除未使用镜像，执行前确认
```

每月检查：

- [ ] 根分区剩余空间不低于 20%；
- [ ] `docker system df` 无异常增长；
- [ ] 不再使用的语言版本和 IDE 已卸载；
- [ ] 项目依赖锁文件已提交；
- [ ] 重要数据有独立备份，不依赖开发机单盘。

---

## 11. 待确认事项

1. 主要技术栈是 Web 全栈、Java 后端、Go/Rust 服务端，还是移动端？
2. 是否需要多版本 JDK/Node/Python 并存？
3. 容器方案选 Docker Engine 还是 rootless Podman？
4. 是否先按系统升级方案挂载 `/data`，再安装 Android SDK 和大型镜像？
5. AI IDE 是否只保留 Qoder，还是需要与 OpenCode/Trae 并存？
6. 团队项目是否需要固定工具链版本（mise/asdf 或各语言锁文件）？

---

## 12. 版本策略与变更记录

### 12.1 版本策略

- `0.0.x`：参考清单，允许增删工具；
- `0.1.0`：技术栈和安装范围确认；
- `1.0.0`：本机开发环境安装完成并通过验证。

### 12.2 变更记录

| 版本 | 日期 | 说明 |
|---|---|---|
| 0.0.1 | 2026-10-20 | 初始清单：存储约定、安装方式对比、工具总览、分类安装、组合建议与卸载手册 |
