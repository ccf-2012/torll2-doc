# 过滤器规则说明 (Filter Rules)

系统中所有的过滤功能（包括 RSS 订阅和追剧）都使用一套统一的、基于 JSON 的规则引擎。这套引擎非常灵活，可以通过组合不同的规则来实现精确的资源筛选。

### 规则基本结构

一个完整的过滤器由一个 `mode` 和一个 `rules` 列表组成。

```json
{
  "mode": "all",
  "rules": [
    { "field": "title", "operator": "contains", "value": "1080p" },
    { "field": "size_gb", "operator": "gt", "value": 5 }
  ]
}
```

*   `mode`: 定义 `rules` 列表中多条规则的组合方式。
    *   `"all"`: **与 (AND)** 逻辑，所有规则都必须匹配。
    *   `"any"`: **或 (OR)** 逻辑，只需匹配任意一条规则即可。
*   `rules`: 一个规则对象的列表，每个对象定义了一条独立的匹配逻辑。

### 规则对象

每个规则对象包含三个部分：`field`, `operator`, 和 `value`。

```json
{ "field": "...", "operator": "...", "value": "..." }
```

#### `field` (字段)

指定要对项目的哪个信息进行匹配。

*   **可用字段**:
    *   `title`: 种子主标题
    *   `subtitle`: 种子副标题
    *   `extitle`: (来自tordb的)扩展标题
    *   `media_title`: (来自tordb的)媒体标题
    *   `site`: 站点标识 (例如 `hdsky`)
    *   `size_gb`: 种子大小 (单位: GB)
    *   `tags`: 种子的标签 (字符串)
    *   `category`: 种子的分类 (字符串)
*   **多字段匹配**: `field` 的值可以是一个列表，表示对列表中的多个字段进行匹配，满足**任意一个**即可。
    *   `"field": ["title", "subtitle"]`

#### `operator` (操作符)

定义如何进行比较。

| 操作符 | 说明 | 示例 `value` |
| :--- | :--- | :--- |
| `contains` | 包含 (不区分大小写) | `"keyword"` |
| `not_contains` | 不包含 (不区分大小写) | `"keyword"` |
| `regex` | 正则表达式匹配 | `"^Movie.*2023$"` |
| `not_regex` | 正则表达式不匹配 | `"S\d+E\d+"` |
| `gt` | 大于 (Greater Than) | `10` |
| `lt` | 小于 (Less Than) | `50` |
| `eq` | 等于 (Equal to) | `25` |
| `in` | 存在于列表中 | `["siteA", "siteB"]` |
| `not_in` | 不存在于列表中 | `["siteC", "siteD"]` |

#### `value` (值)

用于与字段内容进行比较的值，其类型需与操作符的要求相匹配。

### 应用示例

#### 示例 1: RSS 过滤器

需求：匹配所有**标题**包含 `1080p` 但不包含 `x265`，且**大小**在 5GB 到 20GB 之间的种子。

```json
{
  "mode": "all",
  "rules": [
    { "field": "title", "operator": "contains", "value": "1080p" },
    { "field": "title", "operator": "not_contains", "value": "x265" },
    { "field": "size_gb", "operator": "gt", "value": 5 },
    { "field": "size_gb", "operator": "lt", "value": 20 }
  ]
}
```
*注意：对于 RSS，其配置本身是一个过滤器列表，列表中的过滤器是 "OR" 关系。上面的示例是列表中的一个元素。*

#### 示例 1.5: 旧格式转换示例 (Legacy Format Conversion)

为了帮助理解，这里是一个旧版 RSS 过滤器格式转换为新格式的直接示例。

*   **旧格式**:
    ```json
    [
      {
        "size_gb_max": 55,
        "title_not_regex": "S\\d+E\\d+|720p",
        "subtitle_not_regex": "第\\d+\s*集"
      }
    ]
    ```

*   **等效的新格式**:
    ```json
    [
      {
        "mode": "all",
        "rules": [
          { "field": "size_gb", "operator": "lt", "value": 55 },
          { "field": "title", "operator": "not_regex", "value": "S\\\\d+E\\\\d+|720p" },
          { "field": "subtitle", "operator": "not_regex", "value": "第\\\\d+\s*集" }
        ]
      }
    ]
    ```


#### 示例 2: 追剧过滤器

需求：订阅美剧《人生切割术》(Severance)，只想要 `HDSky` 或 `PterClub` 站的 `2160p` 资源，但不要 `REMUX` 版本。

在追剧模块中，**订阅名称本身会作为一个正则匹配规则自动加入**。我们只需要配置额外的规则即可。

*   **订阅名称 (Name)**: `Severance`
*   **规则 (Rules) JSON**:
```json
{
  "mode": "all",
  "rules": [
    { "field": "site", "operator": "in", "value": ["hdsky", "pterclub"] },
    { "field": "title", "operator": "contains", "value": "2160p" },
    { "field": "title", "operator": "not_contains", "value": "REMUX" }
  ]
}
```
系统在匹配时，会自动将订阅名 `Severance` 也作为一个匹配条件（`"field": ["title", "extitle", "media_title"], "operator": "regex", "value": "severance"`）加入到 `rules` 列表中。
```