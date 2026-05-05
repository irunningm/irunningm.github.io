---
permalink: /about/
title: "关于"
---

这是 **irunningm** 的家庭实验室。

这里记录我折腾 NAS、智能家居和自动化工具的过程。从一台群晖 DS918+ 开始，逐步构建一个更聪明的居住空间。

## 我的家庭实验室

### 核心设备

| 设备 | 配置 |
|------|------|
| NAS | 群晖 DS918+ (Intel J3455) |
| 存储 | 18TB RAID5 + DX517 扩展柜 |
| 下载机 | qBittorrent + arr 全家桶 |
| 媒体中心 | Jellyfin + MoviePilot |

### 自动化流程

```
用户添加电影/剧集
        ↓
Prowlarr 搜索种子 → Radarr/Sonarr 筛选 → qBittorrent 下载
        ↓
MoviePilot 自动整理 → 硬链接到媒体库
        ↓
Jellyfin 自动刷新，随时可看
```

全程零人工干预，从添加到观看全自动。

## 关于本站

- **主题**: 家庭实验室的技术探索
- **内容**: NAS、Docker、智能家居、自动化脚本
- **工具**: MiMo AI、Docker、Home Assistant（规划中）

## 联系方式

- GitHub: [irunningm](https://github.com/irunningm)

---

*本站使用 Jekyll + Minimal Mistakes 主题构建。*
