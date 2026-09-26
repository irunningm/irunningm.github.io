# 家庭实验室 404

个人博客：<https://irunningm.github.io>。使用 Jekyll、GitHub Pages 和 Chirpy 主题（官方 Starter 结构，主题通过 gem 引入）。

## 本地环境

Ruby 版本见 `.ruby-version`。依赖版本由 `Gemfile.lock` 固定，gems 安装在本项目的 `vendor/bundle`，不需要 `sudo gem install`。

在 Linux Mint 22 / Ubuntu 24.04 的 x86_64 电脑上，可以安装项目内的 Ruby：

```bash
./scripts/bootstrap-ruby
./scripts/blog setup
```

安装脚本下载 ruby/ruby-builder 的 Ruby 3.3.11 预编译包并验证 SHA-256，只写入 `.runtime/`。仍需系统有 `curl`、`tar`、`gcc`、`g++` 和 `make`，用于下载和编译 gem 原生扩展。

其他系统（包括 Mac）请通过自己的 Ruby 版本管理器安装 `.ruby-version` 中的版本，再运行 `./scripts/blog setup`。项目脚本优先使用 `.runtime/ruby`，没有时使用 PATH 中的 Ruby。

## 写作与预览

```bash
./scripts/blog serve
```

打开 <http://127.0.0.1:4000>。预览包含 `_drafts/`，修改文章后会重新构建，刷新浏览器即可。用 Ctrl+C 停止。

- 已发表文章：`_posts/YYYY-MM-DD-slug.md`
- 草稿：`_drafts/slug.md`
- 图片：`assets/images/`
- 置顶设备分工文章：`_posts/2026-09-26-a-home-for-my-development-environment.md`（本地已准备，需随站点发布上线）

草稿的预览日期默认由文件修改时间决定。正式发表时，把文件移入 `_posts/`，加上日期前缀和 front matter 中的 `date`。

本地预览使用 `_config.local.yml`，关闭统计脚本和评论。它仅监听本机地址，不会发布到互联网。

## 构建检查

```bash
./scripts/blog preview-build  # 包含草稿，输出到 _site-preview/
./scripts/blog build          # 正式构建，不含草稿，输出到 _site/
./scripts/blog check          # 检查正式构建的站内链接、图片和脚本
```

主题随 gem 安装，构建不再临时下载远程主题。首次安装依赖需要访问 RubyGems；浏览器默认按 Chirpy 配置加载第三方字体和静态库。

## 发布

本地构建和预览不会上传任何内容。正式发布前，在仓库 Settings → Pages → Build and deployment 中将 Source 设为 **GitHub Actions**。然后将确认过的修改推送到 `master`；官方 Starter 的 `.github/workflows/pages-deploy.yml` 会构建、检查站内链接并部署。

不要使用 GitHub Pages 内置的分支 Jekyll 构建：Chirpy 使用自己的 Jekyll 依赖与插件。不要把 `.runtime/`、`vendor/`、`_site/` 或 `_site-preview/` 提交进仓库。

旧文章继续使用 `/tech/文章名/`，归档入口保持 `/posts/`。日常设置优先修改 `_config.yml`，导航页面位于 `_tabs/`。尽量不复制主题布局和样式，以便升级。

评论原配置缺少 Giscus ID，目前保持关闭；Umami 统计已迁移到 Chirpy 原生配置，本地预览关闭。

## 参考

- [Jekyll 本地环境说明](https://jekyllrb.com/docs/installation/ubuntu/)
- [GitHub Pages 本地测试](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll)

- [Chirpy 官方入门与发布说明](https://chirpy.cotes.page/posts/getting-started/)
