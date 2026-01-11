---
title: 如何在linux服务器部署一个属于你自己的QQBOT
published: 2026-01-10
description: 本教程将介绍如何使用 Maibot / astrbot搭建一个QQbot。整个过程需要一定的技术基础，但按照步骤操作即可完成
image: ""
tags:
  - 教程
category: 教程
draft: false
lang: ""
---
## 一、基础环境准备

以下以 **Debian / Ubuntu** 系为例，其他发行版命令请自行等价替换。

### 1. 系统与基础工具

```bash
# 更新系统
sudo apt update && sudo apt upgrade -y

# 安装常用工具
sudo apt install -y git curl wget screen python3-dev 
```
### 2. 确认 / 安装 Python（>= 3.10）

MaiBot 要求 **Python = 3.12**
AstrBot 要求**Python = 3.10**

```bash
python3 --version
```

若版本低于 3.10，可按 MaiBot 文档示例安装（以 3.12 为例）：  

```bash
# 安装 Python 3.12 及 venv
sudo apt install -y python3.12 python3.12-venv

# （可选）将 python3 默认指向 3.12
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.12 1
sudo update-alternatives --config python3
```

### 3. 安装 uv

MaiBot 文档推荐使用 **uv** 作为 Python 包管理器。  

可以选其一方式安装（任选其一即可）：

#### 方式 A：使用 pip 安装（与 MaiBot 文档保持一致）

```bash
# 使用 pip3 从镜像源安装 uv
pip3 install uv --break-system-packages \
  -i https://mirrors.huaweicloud.com/repository/pypi/simple/

# 确保本地 bin 在 PATH 中
grep -qF 'export PATH="$HOME/.local/bin:$PATH"' ~/.bashrc || \
  echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

#### 方式 B：使用官方脚本安装（官方文档推荐）  

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
# 或：
# wget -qO- https://astral.sh/uv/install.sh | sh
```

#### 方式 C：如果发行版已提供 uv 包

部分较新的发行版可能已经内置 uv 包，可以尝试：

```bash
sudo apt install -y uv
```

安装完成后验证：

```bash
uv --version
```

>[!TIP]
>建议在这步顺手给UV换国内源

---
## 二、使用源码部署 AstrBot 并接入 QQ（NapCat）

### 1. 克隆 AstrBot 源码

AstrBot 官方文档推荐直接从 GitHub 克隆源码：  

```bash
# 建议在用户目录下新建一个目录统一存放
mkdir -p ~/bots
cd ~/bots

git clone https://github.com/AstrBotDevs/AstrBot.git
cd AstrBot
```

### 2. 使用 uv 安装依赖

AstrBot 官方推荐使用 uv：  

```bash
# 在 AstrBot 目录下
uv sync
```

`uv sync` 会自动：

- 为该项目创建虚拟环境（通常是 `.venv`）
- 根据项目配置安装依赖

首次执行可能时间较长

### 3. 运行 AstrBot（源码模式）

```bash
# 首次运行（会自动同步依赖）
uv run main.py
```

如果之后你安装了很多插件，再启动时可以加 `--no-sync` 避免每次重新安装依赖：  

```bash
uv run --no-sync main.py
```

成功后，日志中会提示类似：

- 管理面板已启动，地址为 `http://localhost:6185`

如果是在服务器上运行，把 `localhost` 换成服务器 IP，比如：

- `http://<服务器IP>:6185`

默认账号密码：

- 用户名：`astrbot`
- 密码：`astrbot`  

首次登录会强制要求修改密码，密码长度至少 8 位

### 4. 安装 NapCat（AstrBot 侧）

AstrBot 官方建议直接参考 NapCat 文档进行安装：  

- [NapCat GitHub](https://github.com/NapNeko/NapCatQQ/)
- [NapCat.Installer - Linux 一键使用脚本(支持Ubuntu 20+/Debian 10+/Centos9)](https://napneko.github.io/guide/boot/Shell)

安装完成后，NapCat 会提供一个 WebUI（通常为 6099 端口）和登录二维码。

一般步骤是：

1. 启动 NapCat（具体命令按 NapCat 文档）
2. 查看日志获取 WebUI 地址和Token（docker 时常用 `docker logs napcat`）  
3. 在浏览器打开 WebUI，用你要作为机器人的 QQ 扫描二维码登录

### 5. 在 AstrBot 中配置 OneBot v11（NapCat）

参考 AstrBot 官方 “使用 NapCat” 文档：  

1. 登录 AstrBot 管理面板
2. 左侧点击「机器人」
3. 右上角「+ 创建机器人」
4. 选择 **OneBot v11**

在弹出的配置中，按需填写：

- **ID**：任取，用来区分不同消息平台实例，如 `qq-main`
- **启用**：勾选
- **反向 WebSocket 主机地址**：`0.0.0.0`（表示监听所有网卡）
- **反向 WebSocket 端口**：如 `6199`（默认值）
- **反向 WebSocket Token**：如果你在 NapCat 中配置了 token，这里要保持一致，否则留空

配置完后点击「保存」。

### 6. 在 NapCat 中添加 WebSocket 客户端（连接 AstrBot）

在 NapCat WebUI 中按 AstrBot 文档操作：  

1. 打开 NapCat WebUI
2. 进入「网络配置」→「新建」→ 选择 **WebSockets 客户端**
3. 关键字段示例：

   - 启用：勾选
   - URL：`ws://<AstrBot 所在机器 IP>:6199/ws`  
     - 如果 NapCat 与 AstrBot 在同一台主机，可用 `ws://127.0.0.1:6199/ws`
   - 消息格式：`Array`
   - 心跳间隔：`5000`（毫秒）
   - 重连间隔：`5000`（毫秒）

**注意：**

- URL 结尾一定要有 `/ws`
- IP 不能填 `0.0.0.0`，应是具体 IP 或 `localhost`  

保存后，回到 AstrBot WebUI 的「控制台」，若出现类似：

- `aiocqhttp(OneBot v11) 适配器已连接。`（蓝色日志）

说明 AstrBot 已与 NapCat 成功连通。  

此时可以在 QQ 私聊机器人发送 `/help` 测试。

---

## 三、使用源码部署 MaiBot + MaiBot-Napcat-Adapter + NapCat

### 1. 克隆 MaiBot 及 Napcat Adapter

```bash
mkdir -p ~/bots
cd ~/bots

# 建议单独建一层目录
mkdir maimai
cd maimai

git clone https://github.com/MaiM-with-u/MaiBot.git
git clone https://github.com/MaiM-with-u/MaiBot-Napcat-Adapter.git
```

最终大致结构：

```text
maimai
├── MaiBot
└── MaiBot-Napcat-Adapter
```


### 2. 使用 uv 为 MaiBot 与 Adapter 创建环境并安装依赖

#### 2.1 为 MaiBot 创建虚拟环境并安装依赖

```bash
cd ~/bots/maimai/MaiBot

# 创建虚拟环境（默认 .venv）
uv venv

# 这里可以用两种方式安装依赖

# 使用镜像源安装依赖
uv pip install -r requirements.txt \
  -i https://mirrors.aliyun.com/pypi/simple --upgrade


# 或同步所有包
uv sync
```

#### 2.2 为 MaiBot-Napcat-Adapter 创建环境并安装依赖

```bash
cd ~/bots/maimai/MaiBot-Napcat-Adapter
#创建虚拟环境
uv venv

#这里可以用两种方式安装依赖

#使用镜像源安装依赖
uv pip install -r requirements.txt \
  -i https://mirrors.aliyun.com/pypi/simple --upgrade

#或直接同步所有包  
uv sync 
```

---

### 3. 配置 MaiBot 主程序

```bash
cd ~/bots/maimai/MaiBot

# 创建配置目录
mkdir -p config

# 拷贝模板配置
cp template/bot_config_template.toml   config/bot_config.toml
cp template/model_config_template.toml config/model_config.toml
cp template/template.env               .env
```

然后编辑 `.env`：

```bash
nano .env
```

内容应该如下:

```ini
HOST=127.0.0.1
PORT=8000
# WebUI 独立服务器配置
WEBUI_ENABLED=true
WEBUI_MODE=production   # 模式: development(开发) 或 production(生产)
WEBUI_HOST=0.0.0.0      # WebUI 服务器监听地址
WEBUI_PORT=8001         # WebUI 服务器端口
```

- `HOST`：建议本机先用 `127.0.0.1`
- `PORT`：Linux 文档示例为 `8000`，后面 Napcat Adapter 需要保持一致  
- `WEBUI_ENABLED` : 建议开启，可以打开自带的webUI
- `WEBUI_PORT` : WebUI端口，一般不需要改

其他文件(model_config.toml、bot_config.toml)可以后续用Webui细调
 >[!TIP]
用不了WebUI的也可以阅读以下文档:
[bot_config配置教程](https://docs.mai-mai.org/manual/configuration/configuration_standard.html) 
[ model_config配置教程](https://docs.mai-mai.org/manual/configuration/configuration_model_standard.html)

---

### 4. 配置 MaiBot-Napcat-Adapter

#### 4.1 复制配置模板

Linux 部署文档示例：  

```bash
cd ~/bots/maimai/MaiBot-Napcat-Adapter

# 根据你本地 template 目录中的文件名选择一个：
ls template
# 可能看到：
#   template_config.toml  或  config_template.toml

# 若存在 template_config.toml（新文档用法）
cp template/template_config.toml config.toml

# 若只有 config_template.toml（旧文档写法）
# cp template/config_template.toml config.toml
```
#### 4.2 编辑 config.toml

打开配置文件：

```bash
nano config.toml
```

mmc和napcat连接的ws配置：  

```toml
[inner]
version = "0.1.3" # 版本号
# 请勿修改版本号，除非你知道自己在做什么

[nickname] # 现在没用
nickname = ""

[napcat_server] # Napcat连接的ws服务设置
host = "localhost"      # Napcat设定的主机地址
port = 8095             # Napcat设定的端口 
token = ""              # Napcat设定的访问令牌，若无则留空
heartbeat_interval = 30 # 与Napcat设置的心跳相同（按秒计）

[maibot_server] # 连接麦麦的ws服务设置
host = "localhost" # 麦麦在.env文件中设置的主机地址，即HOST字段
port = 8000        # 麦麦在.env文件中设置的端口，即PORT字段
```

可选的聊天控制配置（黑白名单等）：  

```toml
[chat] # 黑白名单功能
group_list_type = "whitelist" # 群组名单类型，可选为：whitelist, blacklist
group_list = [] # 群组名单
# 当group_list_type为whitelist时，只有群组名单中的群组可以聊天
# 当group_list_type为blacklist时，群组名单中的任何群组无法聊天
private_list_type = "blacklist" # 私聊名单类型，可选为：whitelist, blacklist
private_list = [] # 私聊名单
# 当private_list_type为whitelist时，只有私聊名单中的用户可以聊天
# 当private_list_type为blacklist时，私聊名单中的任何用户无法聊天
ban_user_id = []   # 全局禁止名单（全局禁止名单中的用户无法进行任何聊天）
ban_qq_bot = true # 是否屏蔽QQ官方机器人
enable_poke = true # 是否启用戳一戳功能
```

语音、日志等级等：  

```toml
[Voice]
use_tts = false  # 若后续配置了 TTS adapter，可改为 true

[Debug]
level = "INFO"   # DEBUG / INFO / WARNING / ERROR
```

修改以下字段即可:

- heartbeat_interval`  心跳间隔
- `group_list` 群组列表
- `private_list` 私聊列表

---

### 5. 安装并配置 NapCat（MaiBot 侧）

Linux 部署文档建议直接参考 NapCat QQ 文档（Shell 版 / Framework 版，二选一）：  

安装方式可与 AstrBot 部分相同，不再重复。

**关键是 NapCat 的 WebSocket 客户端配置：**

1. 在 NapCat WebUI 中新建 **WebSocket 客户端**（NapCat 文档中通常叫 “反向 WebSocket 客户端”）
2. 将 URL 设置为：

   ```text
   ws://localhost:8095/
   ```

   或者：

   ```text
   ws://<MaiBot 服务器 IP>:8095/
   ```

3. 心跳间隔与 config.toml 中 `heartbeat` 转化一致  
   - 例如 NapCat 中填 `30000 ms`，则 `heartbeat = 30`
3. 消息格式使用 NapCat 推荐的 Array / 默认方式

---

### 6. 启动 MaiBot 与 Napcat Adapter（使用 uv）

#### 6.1 前台运行（便于调试）

**启动 MaiBot 核心：**

```bash
cd ~/bots/maimai/MaiBot
uv run python3 bot.py
```

首次运行时，会出现 EULA 检查提示，需要输入：

- `同意` 或 `confirmed` 表示已阅读并同意协议。  

**在另一个终端窗口中启动 Napcat Adapter：**

```bash
cd ~/bots/maimai/MaiBot-Napcat-Adapter
uv run python3 main.py
```

若两边都正常启动，并且 NapCat WebSocket 客户端配置无误，此时 MaiBot 就已经通过 NapCat 接入 QQ

#### 6.2 使用 screen 后台运行

如果希望关闭终端后程序继续运行，可以用 screen（官方文档也推荐）：  

**MaiBot 核心：**

```bash
cd ~/bots/maimai/MaiBot
screen -S mmc              # 新建名为 mmc 的会话
uv run python3 bot.py
# 等待 EULA 提示并输入 同意 / confirmed
# 然后按 Ctrl + a，再按 d，退出会话但程序继续运行
```

**Napcat Adapter：**

```bash
cd ~/bots/maimai/MaiBot-Napcat-Adapter
screen -S mmc-adapter
uv run python3 main.py
# Ctrl + a, d 退出
```

常用 screen 操作：  

```bash
screen -ls        # 查看所有会话
screen -r mmc     # 重新进入 mmc
screen -r mmc-adapter
```

---

## 四、同时源码部署 AstrBot 与 MaiBot 时，如何用 uv 管理和快速切换版本

假设你希望：

- AstrBot 使用 Python 3.10
- MaiBot 使用 Python 3.12

### 1. 让 uv 安装并管理多个 Python 版本

```bash
# 安装多个版本（如 3.10 和 3.12）
uv python install 3.10 3.12
```

uv 会在自己的目录中管理这些 Python 版本，并提供对应解释器。  

你可以查看可用版本：

```bash
uv python list
```

> 图片建议：`uv python list` 显示多版本 Python 的截图。

### 2. 为不同项目固定（pin）不同的 Python 版本

利用 `.python-version` 文件 + `uv python pin`，在项目内固定默认 Python 版本：  

**为 AstrBot 固定 3.10：**

```bash
cd ~/bots/AstrBot
uv python pin 3.10
```

此时 AstrBot 目录下会生成 `.python-version`，内容为 `3.11`。

**为 MaiBot 固定 3.12：**

```bash
cd ~/bots/maimai/MaiBot
uv python pin 3.12
```

同理，在 `MaiBot` 目录下生成 `.python-version` 文件。

以后在各自目录下执行 `uv run`、`uv venv` 等命令时，uv 会自动选用对应版本的 Python。  

### 3. 切换项目时只需要切目录 + uv run

这样一来，你并不需要“切换全局 Python 版本”，而是：

- 想操作 AstrBot：

  ```bash
  cd ~/bots/AstrBot
  uv run --no-sync main.py
  ```

- 想操作 MaiBot：

  ```bash
  cd ~/bots/maimai/MaiBot
  uv run python3 bot.py
  ```

- 想重启 MaiBot Adapter：

  ```bash
  cd ~/bots/maimai/MaiBot-Napcat-Adapter
  uv run python3 main.py
  ```

uv 会根据各项目目录中的 `.python-version` 自动使用对应版本。

## 五、常见问题与排查思路

### 1. QQ 收不到消息 / 不响应

按顺序检查：

1. NapCat 是否成功登录 QQ（WebUI 是否显示在线）
2. NapCat WebSocket 客户端是否连接成功（连接状态、错误日志）
3. AstrBot 或 MaiBot 日志中是否有连接成功/失败的提示
4. 端口是否被防火墙拦截（例如 6199、8095、8000 等）

### 2. uv 相关问题

- 查看缓存或清理缓存：

  ```bash
  uv cache clean
  ```

- 更新 uv（若使用脚本安装）  

  ```bash
  uv self update
  ```

- 安装新 Python 版本后，记得重新 `uv python pin`，避免 `.python-version` 与项目要求冲突。  

### 3. EULA 未同意导致 MaiBot 退出

- 首次运行 `uv run bot.py` 时，一定要在提示处输入 `同意` 或 `confirmed`，否则进程会直接退出。  