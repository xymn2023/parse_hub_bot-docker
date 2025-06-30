# ParseHubBot Docker 部署指南

ParseHubBot是一个开箱即用的高性能异步[抖音](https://www.douyin.com/)|[TikTok](https://www.tiktok.com/)|[Bilibili](https://www.bilibili.com/)数据爬取工具，支持API调用，在线批量解析及下载。

Github源地址：https://github.com/xymn2023/parse_hub_bot

---

## 部署流程

### 第 1 步：创建配置文件 (`.env`)

此文件用于存放您的所有配置，包括 API 密钥等敏感信息。

在您的服务器上，创建一个名为 `.env` 的文件，并将以下内容复制进去。

```env
##### Bot 配置 #####
API_HASH=
API_ID=
BOT_TOKEN=

# BOT_PROXY=http://127.0.0.1:7890
# PARSER_PROXY=http://127.0.0.1:7890
# DOWNLOADER_PROXY=http://127.0.0.1:7890

##### API 配置 #####
# DOUYIN_API=http://127.0.0.1:80

##### AI总结配置 #####
AI_SUMMARY=True
API_KEY=
BASE_URL=https://apic.ohmygpt.com/v1
MODEL=gpt-4o
```

**重要提示：** 请务必将 `API_HASH`, `API_ID`, `BOT_TOKEN` 和 `API_KEY` (如果启用 AI 总结) 的值替换为您自己的信息。

### 第 2 步：确认 `config/config.py` (创建config文件夹，创建config.py粘贴以下内容)

此文件是程序用来读取 `.env` 中配置的逻辑代码，**您不需要创建或修改它**，只需确保它存在于项目中即可。它的主要作用是从容器的环境变量中加载配置。

其内容如下：
```python
from urllib.parse import urlparse

from dotenv import load_dotenv
from os import getenv

load_dotenv()


class BotConfig:
    def __init__(self):
        self.bot_token = getenv("BOT_TOKEN")
        self.api_id = getenv("API_ID")
        self.api_hash = getenv("API_HASH")
        self.bot_proxy: None | BotConfig._Proxy = self._Proxy(getenv("BOT_PROXY", None))
        self.parser_proxy: None | str = getenv("PARSER_PROXY", None)
        self.downloader_proxy: None | str = getenv("DOWNLOADER_PROXY", None)

        self.cache_time = int(ct) if (ct := getenv("CACHE_TIME")) else 600
        self.ai_summary = bool(getenv("AI_SUMMARY").lower() == "true")
        self.douyin_api = getenv("DOUYIN_API", None)

    class _Proxy:
        def __init__(self, url: str):
            self._url = urlparse(url) if url else None
            self.url = self._url.geturl() if self._url else None

        @property
        def dict_format(self):
            if not self._url:
                return None
            return {
                "scheme": self._url.scheme,
                "hostname": self._url.hostname,
                "port": self._url.port,
                "username": self._url.username,
                "password": self._url.password,
            }


bot_cfg = BotConfig()
```

### 第 3 步：创建 `docker-compose.yml`

这是部署的核心文件，它会告诉 Docker 如何运行您的机器人。

在与 `.env` 文件相同的目录下，创建一个名为 `docker-compose.yml` 的文件，并填入以下内容：

```yaml
version: '3.8'
services:
  bot:
    image: smhw3565/parse-hub-bot
    container_name: parse-hub-bot
    restart: always
    env_file:
      - .env
```

### 第 4 步：启动机器人

完成了以上步骤后，在 `docker-compose.yml` 所在的目录中打开终端，运行以下命令：

```bash
docker compose up -d
```

Docker 将会自动拉取 `smhw3565/parse-hub-bot` 镜像，并使用您的 `.env` 配置来启动机器人容器。

---

## 日常管理

- **查看日志**：
  ```bash
  docker compose logs -f
  ```

- **停止机器人**：
  ```bash
  docker compose down
  ```

- **更新机器人** (当有新版镜像时)：
  ```bash
  docker compose pull
  docker compose up -d
  ```

- **重启机器人**：
  ```bash
  docker compose restart
  ```

---

## 项目来源

本项目基于 [z-mio/parse_hub_bot](https://github.com/z-mio/parse_hub_bot) 进行修改和封装，感谢原作者的贡献。
