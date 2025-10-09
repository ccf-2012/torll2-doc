# TORCP 设置

TORCP 是 `torll2` 与 `tordb` 服务之间沟通的桥梁，它负责解析种子信息、获取元数据，并根据设定的模板生成最终的目录结构和文件名。此处的配置将直接影响 `rcp` 脚本整理文件后的效果。

**入口**: 设置 -> TORCP 服务设置

---

## 配置项说明

### 1. TorDB URL

- **说明**: `tordb` 服务的 URL 地址。
- **示例**: 如果 `tordb` 和 `torll2` 在同一个 Docker 网络中，应填写 `http://tordb:6009`。

### 2. TorDB API Key

- **说明**: 访问 `tordb` 服务所需的 API 密钥。此密钥必须与 `tordb` 启动时在 `.env` 文件中设置的 `TORDB_API_KEY` 完全一致。

### 3. 文件名括号

- **说明**: 自定义最终文件名中附加的括号内容，通常用于标注制作组或版本信息。
- **示例**: 如果填 `MyGroup`，文件名中可能会出现 `[MyGroup]` 的字样。

### 4. 目录/文件命名模板

这部分是 TORCP 设置的核心，它决定了媒体文件在硬链接或移动后的目录结构和文件名格式。

- **电影路径模板 (`movie_path_template`)**: 用于电影的命名规则。
- **剧集路径模板 (`tv_path_template`)**: 用于电视剧的命名规则。
- **其他路径模板 (`other_path_template`)**: 用于其他未识别类型的媒体。

**可用变量**: 模板中可以使用一系列由 `tordb` 解析出的变量，例如：
- `{title}`: 媒体标题
- `{year}`: 年份
- `{area5}`: 地区（如 `欧美`, `华语` 等）
- `{emby_bracket}`: 包含 IMDB ID 或 TMDb ID 的括号，用于 Emby/Jellyfin 识别，例如 `[imdbid-tt123456]`。

**默认模板示例**:
- `Movie/{area5}/{title} ({year}) {emby_bracket}`
- `TV/{area5}/{title} {emby_bracket}`

### 5. 高级命名选项

- **目录包含地区 (`areadir`)**: 控制是否在生成的目录路径中包含地区信息（如 `area5` 变量）。
- **包含类型 (`genre`)**: 控制是否在文件名或目录中包含媒体类型（如 `动作`, `喜剧`）。
- **类型包含地区 (`genre_with_area`)**: 控制是否在类型信息后附带地区。

## 截图

![TORCP设置](../torll2_screenshots/settings-torcp.png)