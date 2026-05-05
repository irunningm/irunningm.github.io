---
title: "我让 AI 帮我在群晖 NAS 上搭了一套完整的媒体服务器"
date: 2026-05-04
excerpt_separator: "<!--more-->"
categories:
  - Tech
tags:
  - NAS
  - Docker
  - AI
  - Jellyfin
  - 自媒体
---

从零开始，用对话的方式，搭起了 Prowlarr + Radarr + Sonarr + qBittorrent + MoviePilot + Jellyfin 的全套架构。中间踩了不少坑，但整个过程比我预想的顺畅得多。

<!--more-->

## 起因

我有一台群晖 DS918+，用了好几年了，之前只装了 Jellyfin 看片。平时下载电影电视剧的流程是：找种子 → 手动下载 → 手动整理文件夹 → 手动命名 → 刷新 Jellyfin 库。一套下来挺繁琐的，一直想搞自动化，但看了看 arr 全家桶的文档，各种配置项看得头大，就一直搁着。

最近在试小米的 MiMo-v2.5-pro 模型，想着正好拿这个练练手——让 AI 来帮我搞定这些配置，我负责提需求和验收就行。

## 第一步：搭 arr 全家桶

arr 全家桶是媒体自动化的核心：

- **Prowlarr**：索引器管理，聚合多个种子站
- **Radarr**：电影自动搜索+下载
- **Sonarr**：剧集自动搜索+下载
- **Bazarr**：字幕自动下载
- **qBittorrent**：实际干活的 BT 下载客户端

我用 docker-compose 把这些服务一股脑丢到 NAS 上。AI 帮我写好了 compose 文件，配好了环境变量、端口映射、数据卷挂载。一键 `docker compose up -d`，六个容器全部起来。

代理是个关键问题。种子站基本都在海外，得走代理。但 qBittorrent 不能走代理——BT 流量走代理会被封。AI 帮我在每个服务的环境变量里单独配了 `HTTP_PROXY`，qBittorrent 则不设代理，完美解决。

## 第二步：接入 Jellyfin

Jellyfin 之前就有了，这次主要是让它和 arr 全家桶配合起来。关键是目录规划：

```
/volume1/video/
├── download/       ← qBittorrent 下载到这里
│   ├── movies/
│   ├── tv/
│   └── anime/
├── link/           ← Jellyfin 读取这里
│   ├── movies/
│   ├── tv/
│   └── anime/
```

下载目录和媒体目录在同一块硬盘上，用**硬链接**连接——文件只占一份空间，但两个路径都能访问。下载完成后删掉 download 里的源文件，link 里的照常可用。

## 第三步：踩坑 MoviePilot

MoviePilot 是整个链路里最折腾的一个。它的作用是监控 download 目录，新文件来了自动重命名、刮削元数据（海报、简介、NFO 文件），然后硬链接到媒体目录。

坑在哪呢？

### 1. API 设置目录有 bug

MoviePilot 的 Web UI 里设置监控目录，保存后数据结构会双层嵌套，导致程序读不到配置。AI 试了好几种 API 调用方式都不行，最后发现得直接写 SQLite 数据库：

```sql
UPDATE systemconfig SET value = '...' WHERE key = 'Directories';
```

这种绕过 UI 直接改数据库的操作，如果没 AI 帮忙排查，我自己估计得折腾一晚上。

### 2. 配置项名字不直觉

- `transfer_type` 必须填 `"link"`，不是 `"hardlink"`
- `monitor_type` 必须填 `"monitor"`，不是 `"downloader"`
- `renaming` 必须设为 `true`，不然不会重命名

任何一个填错都不会报错，只是功能不生效。这种隐性配置错误，AI 通过读源码帮我定位到了。

### 3. inotify 监控的坑

MoviePilot 用 inotify 监控新文件，但 `touch` 一个文件不会触发事件，必须 `mv` 才行。这意味着你不能靠"碰一下"来触发整理。

## 第四步：批量导入已有资源

我的 NAS 上已经有 337 部电影、50 多个电视剧、十几个动漫。手动一个个导入不现实。

电影部分靠 Radarr 的"Library Import"功能搞定——扫描目录，自动匹配 TMDB。

电视剧和动漫就麻烦了，文件命名五花八门：有的是 `[SubGroup]Title - 01.mkv`，有的是 `S01E01.mkv`，有的干脆就是数字编号。AI 帮我写了个 `tv_import.sh` 脚本，用正则表达式匹配各种命名模式，批量创建硬链接：

```bash
bash tv_import.sh        # 执行
bash tv_import.sh dry    # 预览模式，先看看对不对
```

最终导入了 669 个电视剧硬链接、267 个动漫硬链接。

## 第五步：误识别和清理

MoviePilot 太"勤快"了，什么都想整理。字幕文件（.srt）被当成电影、sample.mkv 被当成正片、NFO 文本文件也被识别成什么"更多电影请访问"。最后生成了 786 条失败记录和 80 多条错误映射。

AI 通过 SQLite 查询找到了所有误识别的条目，按模式分类清理：

- 字幕文件（.srt, .ass）：398 条
- Sample 文件：10 条
- Featurettes 和 NFO：剩余的

然后通过 API 和直接操作数据库，把错误记录全部清掉。

## 第六步：设置 4K 画质

一切就绪后，我跟 AI 说"帮我设置都是 4K 画质"。它通过 Sonarr 的 API，把 Quality Profile 改成了只允许 2160p 的资源：

- Bluray-2160p Remux
- Bluray-2160p
- WEBDL-2160p / WEBRip-2160p
- HDTV-2160p

低于 4K 的种子直接忽略。以后追剧不用担心下到 720p 了。

## 最终架构

```
用户在 Radarr/Sonarr 添加电影/剧集
        ↓
Prowlarr 从种子站搜索资源
        ↓
找到合适的 4K 种子，推给 qBittorrent
        ↓
qBittorrent 下载完成
        ↓
MoviePilot 自动重命名 + 刮削元数据 + 硬链接到媒体目录
        ↓
Jellyfin 自动发现新内容，随时可看
```

从添加一部电影到能在电视上看到，全程零人工干预。

## 我的感受

整个过程下来，最大的感受是：**AI 真正的价值不在于帮你写代码，而在于帮你排错。**

Docker compose、环境变量这些，其实看看文档我也能写。但 MoviePilot 的 API bug、配置项命名的坑、inotify 的行为特性、SQLite 直接操作——这些东西没有源码级的排查能力，靠自己摸索得花好几天。

AI 的工作方式很像一个经验丰富的运维同事：你描述问题，它读源码、查日志、试方案，最后给你一个能用的解决方案。你不需要理解每一个细节，但你知道它在帮你干活。

当然，也有局限。比如 UI 里的 checkbox 点击，因为是 React 组件，AI 的自动化工具反复操作都失败，最后只能绕过 UI 直接调 API。这说明 AI 操控前端界面的能力还有待提高。

## 硬件配置参考

| 项目 | 配置 |
|------|------|
| NAS | 群晖 DS918+ (Intel J3455) |
| 存储 | 18TB RAID5 + DX517 扩展柜 |
| Docker 容器数 | 8 个（arr 全家桶 + Jellyfin + MoviePilot） |
| 代理 | 路由器层面，HTTP 1081 / SOCKS5 1082 |

---

*本文由 MiMo-v2.5-pro 协助完成。文中提到的所有 Docker 配置、脚本编写、问题排查均由 AI 在对话中完成。*
