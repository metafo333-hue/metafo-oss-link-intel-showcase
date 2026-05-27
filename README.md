# link-intel 实测展示

> 本目录为 link-intel skill（`.claude/skills/link-intel/`）完整流水线实测产出。  
> 4 类链接 → 素材全提取，源资源到成品一览。

---

## 目录结构

```
link-intel-showcase/
├── 01-article/   图文文章  — realpython.com（trafilatura 提取）
├── 02-doc/       文档代码  — GitHub anthropic-sdk-python（raw 直取）
├── 03-video/     抖音视频  — 史铁生《我的梦想》（完整流水线）
└── 04-social/    社交边界  — 说明 xhslink 边界处理
```

---

## 01 · 图文文章

**来源**：`https://realpython.com/python-f-strings/`  
**提取器**：trafilatura  
**产出**：`01-article/产出/文案/正文.md`（40KB 完整正文）

流程：URL → trafilatura.fetch_url + extract → 文案/正文.md + 来源.json

---

## 02 · 文档代码

**来源**：`https://github.com/anthropics/anthropic-sdk-python`  
**提取器**：github-raw（拼接 `/raw/HEAD/README.md` 直接 curl）  
**产出**：`02-doc/产出/文案/正文.md`（README 原文）

流程：URL → curl raw README → 文案/正文.md + 来源.json

---

## 03 · 抖音视频（完整流水线）

**来源**：`https://v.douyin.com/8QTHM0yh-KY/`  
**内容**：史铁生《我与地坛》—— 我的梦想

### 流水线步骤

| 步骤 | 工具 | 产出 |
|------|------|------|
| ① 视频下载 | Playwright 拦截 + curl | `产出/原视频.mp4`（3.4 MB，35s） |
| ② 音轨提取 | ffmpeg -vn pcm_s16le | `产出/音轨/完整音轨.wav`（6.0 MB） |
| ③ 字幕识别 | faster-whisper large-v3 ASR | `产出/字幕/字幕-ASR.txt` |
| ④ BGM分离 | demucs htdemucs two-stems | `产出/BGM分离/htdemucs/完整音轨/vocals.wav` + `no_vocals.wav` |
| ⑤ 关键帧 | ffmpeg fps=1/5 scale=720 | `产出/关键帧/关键帧_001~007.jpg`（7张） |

### 字幕识别结果（ASR 原文）

```
我希望既有一个健美的躯体
又有一个悟了人生意义的灵魂
前者可以期望上帝的恩赐
后者却必须在千难万苦中
靠自己去获取
千万不要说
倘若两者不可间断
你要选哪一个呢
因为人活着
必须有一个最美的梦想
来自史铁生
我的梦想
```

### 关于抖音下载

yt-dlp 2026.03 版存在已知 issue（#12669）——即使有正确 cookie，Douyin API 返回空 200 body。  
**解决方案**：改用 Playwright headless 浏览器拦截 `douyinvod.com` 视频流 URL，bypass API 层直接下载。  
此方案已集成为 `scripts/douyin_playwright.py`（见下方 skill 改进说明）。

---

## 04 · 社交图文（边界说明）

**平台**：小红书（xhslink.com）、微博、X  
**状态**：边界案例，无免费自动提取方案

| 平台 | 原因 | 替代方案 |
|------|------|---------|
| 小红书 | 强登录墙 + 反爬 | MediaCrawler（NON-COMMERCIAL，需独立部署）|
| 微博 | 需账号 Cookie | MediaCrawler |
| X/Twitter | 无免费 API | 手动复制文案 |

extract.py 对 social 类型返回边界说明而非报错，归档目录正常建立。

---

## Skill 路径

`.claude/skills/link-intel/`（项目 metafo-commerce 仓库内）

- `scripts/extract.py`   — A 线 4 类提取器 + A6 归档
- `scripts/subtitle.py`  — yt-dlp 字幕 + faster-whisper ASR
- `scripts/media.py`     — ffmpeg 音轨/关键帧 + demucs 分离
- `references/`          — 提取 SOP + 深探维度 + 报告骨架
- `tests/`               — 6 项单元测试（6/6 通过）

---

> 实测日期：2026-05-23  
> 视频素材版权归原作者，仅用于 skill 功能验证
