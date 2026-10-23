# DokoRail（どこレール）本地静态镜像

[dokorail.jp](https://dokorail.jp/) 的完整本地副本 —— 日本全国列车实时位置地图。
全部页面、脚本、数据、图标、底图、铁路线瓦片与 **3D 列车模型**均已落到本地磁盘，
**运行时不向原站发起任何请求**。

抓取时间：2026-10-23　体积：解压后约 1.1 GB，15,418 个文件

---

## 下载与解压

因为 GitHub 单文件限制 100 MB，压缩包被切成 **5 个分卷**，下载后必须合并解压。

### 方法 1：7-Zip / WinZip / WinRAR（推荐，Windows / macOS）

把 5 个分卷放在**同一个文件夹**里，**只右键解压 `dokorail-local.zip`**，
软件会自动读取 `.z01`～`.z04`。不要单独解压 `.z01`。

### 方法 2：命令行

```bash
# 7-Zip 可直接识别分卷
7z x dokorail-local.zip

# 或先合并再解压
zip -s 0 dokorail-local.zip --out full.zip
unzip full.zip
```

解压得到一个 `dokorail-local/` 目录。

| 分卷 | 大小 |
|---|---|
| `dokorail-local.z01` | 90 MB |
| `dokorail-local.z02` | 90 MB |
| `dokorail-local.z03` | 90 MB |
| `dokorail-local.z04` | 90 MB |
| `dokorail-local.zip` | 86 MB |
| **合计** | **447 MB**（解压后 1.1 GB） |

---

## 运行

```bash
cd dokorail-local
python3 server.py            # 纯离线模式
# 浏览器打开 http://127.0.0.1:8765/
```

| 模式 | 命令 | 说明 |
|------|------|------|
| 纯离线（默认） | `python3 server.py` | 只读本地磁盘 |
| 混合 | `python3 server.py --live` | 本地没有的瓦片/接口回源原站并缓存 |

也提供 `./start.sh`。**必须用 HTTP 方式打开**，不能直接双击 `index.html`
（页面是 ES Module + Worker + fetch 架构，`file://` 会被同源策略拦截）。

---

## 内容清单

| 类别 | 内容 |
|---|---|
| 页面 | 5 个入口（`/`、`/app/`、`/about/`、`/en/about/`、`/privacy`、`/en/privacy`） |
| 脚本 | 43 个 JS/CSS 分块、126 个字体子集 |
| 线路数据 | 176 家运营商 / 599 条线路几何、8,255 条概览记录、77 个特殊线路几何 |
| 时刻表 | 6,888 个资产 + 257 个按日数据包（sha256 全部校验通过） |
| 铁路线瓦片 | RTAR 归档 124 MB，125,249 块瓦片（全日本线网） |
| 底图 | 样式 / 精灵 / 字形全量 + 日本 z3–z8、九大都市圈 z8–z11 |
| 图片资源 | 194 个运营商标识、135 张车站照片图集 |
| **3D 模型** | **131 种车型、1,834 个 glb、2,758 个 ktx2 贴图 + Basis 转码器** |
| 接口快照 | region、status feed、train feed、trains |

---

## 本地化的关键处理

1. **RTAR 归档靠 HTTP Range 随机读取** —— 服务器实现 `206 Partial Content`，否则地图画不出来
2. **`.bin.gz` 不能带 `Content-Encoding: gzip`** —— 前端用 `DecompressionStream` 自行解压
3. **时刻表 URL 是精确字符串匹配** —— 必须是 `timetables/vN/sha256/<hash>.json`
4. **MapLibre 要求 sprite / glyphs 是绝对 URL** —— 服务器按请求 `Host` 动态注入
5. **外部引用全量改写** —— `tiles.dokorail.jp`、`tiles.openfreemap.org` 改为本地路径；
   PostHog / Sentry / Google 统计指向 `/__noop__`；Cloudflare 统计脚本已移除

---

## 已知限制

- **实时列车位置是快照**（2026-10-23 约 23:40 JST），不是实时的；用 `--live` 可回源
- 抓取时段为日本夜间，顶部会显示 `Train feed unavailable`，原站同时访问也是同样提示
- 底图瓦片有覆盖范围，预播种之外需 `--live` 回源一次
- 110 个 `.webp` 可选回退贴图缺失（原站本身即 404，实际使用 `.ktx2`）
- `/api/translate` 依赖服务端翻译，离线返回空

---

## 版权

数据来源与版权归 [DokoRail](https://dokorail.jp/)（@waxdt）所有。
底图 © [OpenFreeMap](https://openfreemap.org/) / [OpenMapTiles](https://openmaptiles.org/) / OpenStreetMap 贡献者。
本副本仅供本地离线使用与存档。
