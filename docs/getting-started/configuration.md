# 设置与使用

本部分将指导你在 Docker 服务成功启动后，如何配置 `torll2` 的各项功能，使其成为一个全自动的媒体管理系统。

## 核心概念

-   **torll2**: 主应用，负责任务调度、RSS解析、连接下载器和媒体库管理。
-   **tordb**: 辅助服务，提供电影、剧集等元数据信息。`torll2` 通过查询它来获取媒体信息。
-   **qBittorrent**: 下载客户端。`torll2` 会将下载任务发送给它。
-   **[rcp 脚本](https://github.com/ccf-2012/rcp)**: 一个在下载机上运行的“信使”。有两种模式，一是运行 rcp_agent，由torll2 发起控制； 另一种是配置 rcp 脚本，当 qBittorrent 下载完成后，会调用此脚本，由它向 `torll2` 请求数据，然后执行后续的整理（重命名、硬链接等）操作。
-   **[torfilter 油猴脚本](https://greasyfork.org/zh-CN/scripts/451748)**: 在站点网页上发起过滤、查重和下载的脚本。
---

## 配置步骤

请按照以下顺序，在 `torll2` 的 Web UI ([http://localhost:6006](http://localhost:6006)) 中进行配置。

### 步骤 1: 连接 Torll2 与 Tordb

这是让系统能够识别媒体信息的关键一步。

1.  在 `torll2` 界面中，导航至 **设置** -> **TORCP 服务设置**。
2.  填写以下信息：
    -   **TorDB URL**: `http://tordb:6009`
        > **说明**: `tordb` 是 Docker 网络内部的服务名，`torll2` 通过这个地址访问 `tordb` 服务。
    -   **TorDB API Key**: 填写你在 `.env` 文件中为 `TORDB_API_KEY` 设置的值。
3.  点击保存。

### 步骤 2: 配置下载器

1.  导航至 **下载** -> **下载客户端**。
2.  点击 **添加下载器**，并填入你的 qBittorrent 客户端信息（WebUI 地址、用户名、密码）。
3.  这里有一个远端映射路径 `Local Map Path` 此路径是 torll2 所在主机访问媒体文件的根目录，用于后续的文件管理（如删除、读取等）。在查找媒体文件时，是由此路径与媒体库中存储的相对路径拼合而成的。比如可以通过本地网络 nfs mount 过来，或上传网盘后rclone(等) mount过来，或者生成 strm 实现访问。
4.  处理模式，有 3 种，分别为 local, agent, legacy:
    1.  由 torll2 直接控制本地选 local，这通常需要 torll2 直接运行，在 Docker 中运行使用此模式较麻烦；
    2.  torll2 在 Docker 中直接控制下载器，或者下载器在远程，选 agent
    3.  由 qbittorrent 完成后调用脚本发起硬链，选 legacy，这个模式对于远端下载器或Docker外下载器，无法修改和删除硬链
    4.  详见 [下载器处理模式](../features/downloader-modes.md)



### 步骤 3.1: 在下载器所在机器上配置  agent 
参见：[下载器处理模式](https://ccf-2012.github.io/torll2-doc/features/downloader-modes/)
如果是 Docker 部署的，则以 agent 模式控制较简单，在下载器所在机器上：
1. 下载 rcp
```sh
git clone https://github.com/ccf-2012/rcp
cd rcp
```

2. 编辑一个 config.ini, 内容为：
```ini
[torll]
# torll2服务的URL地址，地址按自己的设，后面路径不动
url = http://<your.torll2.host:6006>/api/v1/torcp/info
# torll2服务的API Key，由torll2 启动时得到
api_key = <api key get from torll2>
# qbit 的名字，与在 torll2 中配置的下载器名字对上
qbitname = <qb name set in torll2>

[emby]
# Emby/Jellyfin媒体库的根目录
root_path = <your media path >
```

3. 启动 `rcp_agent`

```sh
# 启一个 screen 
python rcp_agent.py
```


### 步骤 3.2: legacy 模式，在下载器所在机器上配置 rcp 脚本

这一步是为了实现下载完成后，与 `torll2` 通信获取信息后，按要求对文件进行重命名和分类。

1.  **下载脚本**: 从 [rcp 脚本仓库](https://github.com/ccf-2012/rcp) 下载到你**运行 qBittorrent 的机器**上（例如，你的 NAS）。
2.  **修改 `rcp.sh`**:
    -   用文本编辑器打开 `rcp` 目录下的 `rcp.sh` 文件。
    -   修改 `cd` 后面的路径，使其指向 `rcp` 目录在你下载机上的**绝对路径**。
    -   确认执行 `rcp.py` 的 `python` 命令路径是否正确。很多设备的默认 Python 版本较低，请确保使用 Python 3.10+ 的解释器。

```sh
#!/usr/bin/bash
# 脚本所在的绝对路径
cd /path/to/your/rcp 
# 使用正确的 Python 解释器路径执行 rcp.py
/opt/bin/python rcp.py $1 -t $2 -u $3 -n $4 >> rcp.log 2>> rcp2e.log
```

3.  **修改 `config.ini`**:
    -   在 `rcp` 目录中，将 `config.ini.template` 复制为 `config.ini`。
    -   修改 `config.ini` 文件，填入 `torll2` 的 `url` 和 `api_key`。
        -   `url`: `torll2` 服务的地址，例如 `http://192.168.1.100:6006`。
        -   `api_key`: `torll2` 自动生成的 API Key（请从 `docker compose logs torll2` 日志中获取）。

4.  **配置 qBittorrent**:
    -   在 qBittorrent 的 **设置** -> **下载** -> **“Torrent 完成时运行外部程序”** 中，填入以下命令（请使用 `rcp.sh` 的绝对路径）：

```sh
sh /path/to/your/rcp/rcp.sh "%F" "%I" "%L" "%N"
```

### 步骤 4: 添加索引站点

1.  导航至 **索引** -> **站点设置**。
2.  点击 **添加站点**，从预设列表中选择你的 PT 站点，并配置:
  * 你的站点 `Cookie` 
  * `速览URL` - 这是用于浏览站点时的起始 url，在点选过滤按钮时会基于此 url 进行拼接

### 步骤 5: 添加 RSS 订阅

1.  导航至 **RSS** -> **RSS源**。
2.  点击 **添加FEED**，填入从站点获取的 RSS 订阅链接。
3.  根据需要配置 **Filter** (过滤器)，可以配置多个过滤规则，只有全部 filter 通过才收录，通过 JSON 格式的规则实现精准下载。详见 [过滤器规则说明](../features/rss.md)


### 步骤 6 (可选): 配置通知服务

在 **设置** -> **通知** 中，你可以根据需要配置 Telegram 或 Emby 通知，以便在下载完成或出错时收到提醒。

### 步骤 7 (可选): 安装油猴脚本

详见 [torfilter 油猴脚本](https://greasyfork.org/zh-CN/scripts/451748)
