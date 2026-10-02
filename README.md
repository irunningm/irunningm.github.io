# 家庭实验室 404

个人博客：<https://irunningm.github.io>。使用 Jekyll、GitHub Pages 和 Chirpy 主题（官方 Starter 结构，主题通过 gem 引入）。

## 本地环境

Ruby 版本见 `.ruby-version`。依赖和 Bundler 版本由 `Gemfile.lock` 固定，gems 安装在本项目的 `vendor/bundle`，不需要 `sudo gem install`。脚本会检查 Ruby 版本，并禁止安装依赖时自动修改锁文件。

在 Mac 上，先准备 Xcode Command Line Tools（`xcode-select -p` 可检查）和 Homebrew，然后运行：

```bash
brew install ruby-build
./scripts/bootstrap-ruby
./scripts/blog setup
```

`ruby-build` 下载并校验 Ruby 源码，把指定版本编译到本仓库的 `.runtime/ruby`。不修改系统 Ruby 或 shell 启动配置；后续直接使用 `./scripts/blog`，无须手动切换 PATH。首次编译需要几分钟。

`setup` 会将项目内过旧的 RubyGems 更新到 3.6.9，修复中文目录下解包 `jekyll-sitemap` 的编码错误，并安装锁文件指定的 Bundler。CI 同样使用 RubyGems 3.6.9；已有版本管理器的 RubyGems 不会被脚本自动修改。

本地脚本还使用 Bundler 官方的 `BUNDLE_DISABLE_EXEC_LOAD` 选项，以普通子进程运行命令，避免旧版 Bundler 在中文 Ruby 路径下的 shebang 编码问题。

在 Linux Mint 22 / Ubuntu 24.04 的 x86_64 电脑上，可以安装项目内的 Ruby：

```bash
./scripts/bootstrap-ruby
./scripts/blog setup
```

安装脚本下载 ruby/ruby-builder 的 Ruby 3.3.11 预编译包并验证 SHA-256，只写入 `.runtime/`。仍需系统有 `curl`、`tar`、`gcc`、`g++` 和 `make`，用于下载和编译 gem 原生扩展。

其他系统也可通过自己的 Ruby 版本管理器安装 `.ruby-version` 中的版本，再运行 `./scripts/blog setup`。项目脚本优先使用 `.runtime/ruby`，没有时使用 PATH 中的 Ruby。

## 写作与预览

```bash
./scripts/blog serve
```

打开 <http://127.0.0.1:4000>。预览包含 `_drafts/`，修改文章后会重新构建，刷新浏览器即可。用 Ctrl+C 停止。

- 已发表文章：`_posts/YYYY-MM-DD-slug.md`
- 草稿：`_drafts/slug.md`
- 图片：`assets/images/`
- 置顶设备分工文章：`_posts/2026-09-26-a-home-for-my-development-environment.md`

草稿的预览日期默认由文件修改时间决定。正式发表时，把文件移入 `_posts/`，加上日期前缀，并填写 front matter 中的 `title`、`date` 和固定 `permalink`，例如 `permalink: /tech/my-post/`。已发表文章的 `permalink` 保持不变；以后修改分类或文件名也不会改变读者访问的网址。确需改网址时，保留旧网址的重定向并验证它。

本地预览使用 `_config.local.yml`，关闭统计脚本和评论。它仅监听本机地址，不会发布到互联网。

## 构建检查

```bash
./scripts/blog preview-build  # 包含草稿，输出到 _site-preview/
./scripts/blog build          # 正式构建，不含草稿，输出到 _site/
./scripts/blog check          # 检查文章网址、站内链接、图片和脚本
```

主题随 gem 安装，构建不再临时下载远程主题。首次安装依赖需要访问 RubyGems。构建前会检查文章元数据和重复网址，构建后还会检查现有文章网址与 canonical 链接；文章头部 YAML 语法错误会使构建失败。

当前页面使用的字体、图标、搜索、日期、目录、灯箱、代码复制、头像等资源随站点托管，来源、固定版本和许可证见 `assets/lib/SOURCES.txt`，文件校验值见 `assets/lib/manifest.json`。统计服务仍独立加载。现有文章未启用数学公式或 Mermaid；以后启用时需另行验证并本地化这两个可选库及其附属资源。

## 发布

本地构建和预览不会上传任何内容。仓库 Settings → Pages → Build and deployment 的 Source 使用 **GitHub Actions**。

日常更新流程：

1. 从最新 `master` 创建工作分支，编辑文章并本地预览。
2. 运行 `./scripts/blog build` 和 `./scripts/blog check`。
3. 推送工作分支并创建 PR，等待 `Blog checks` 通过。
4. 合并到 `master`。工作流会再次构建检查，再部署 Pages；PR 自身不会部署。

部署按队列执行，后续提交不会打断正在进行的部署。若发布后发现问题，在新分支撤销对应改动并通过 PR 恢复，再确认 Pages 部署成功；不要强推覆盖历史。最初的迁移版本为 `a470fd4`，可作为本轮维护前的对照。

不要使用 GitHub Pages 内置的分支 Jekyll 构建：Chirpy 使用自己的 Jekyll 依赖与插件。不要把 `.runtime/`、`vendor/`、`_site/` 或 `_site-preview/` 提交进仓库。

旧文章继续使用 `/tech/文章名/`，归档入口保持 `/posts/`。日常设置优先修改 `_config.yml`，导航页面位于 `_tabs/`。尽量不复制主题布局和样式，以便升级。

评论原配置缺少 Giscus ID，目前保持关闭；Umami 统计已迁移到 Chirpy 原生配置，本地预览关闭。

## 参考

- [Jekyll 本地环境说明](https://jekyllrb.com/docs/installation/ubuntu/)
- [GitHub Pages 本地测试](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll)

- [Chirpy 官方入门与发布说明](https://chirpy.cotes.page/posts/getting-started/)
