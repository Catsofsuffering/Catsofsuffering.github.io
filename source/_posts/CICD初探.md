---
title: CI/CD初探-工具act
tags:
  - CI/CD
  - 备忘录
categories: 备忘录
keywords: act
description: 本文记录了作者在调试CI/CD的workflow时碰见的问题，主要介绍act工具的相关使用方法；
date: 2024-12-25 17:01:00
cover: https://pic.imgdb.cn/item/65b76b5d871b83018ab4a506.jpg
top_img: https://pic.imgdb.cn/item/65b76b5d871b83018ab4a506.jpg
---
此前关注到了 [trickest/cve](https://github.com/trickest/cve) 这个仓库，十分眼馋整理归类的poc，因此想自动化拉取到Github仓库然后搞成静态页面（虽然因为md文档数量太多，因此构建失败）。在寻找解决方案的时候，接触到了Github Action，这才有了这边文章。

##  `nektos/act`介绍

`nektos/act` 是一个开源工具，它模拟 GitHub Actions 的 CI/CD 环境，允许你在本地运行 GitHub Actions 工作流。它可以节省大量的时间，因为你可以在本地验证工作流逻辑，而不必每次都推送代码到 GitHub 仓库并等待远程执行。

## 配置 GitHub Actions 示例工作流

以下是一个示例 GitHub Actions 工作流，笔者将用这个工作流来演示如何通过 `nektos/act` 本地调试 GitHub Actions。

```yaml
name: Trickest CVE Monitor And Deploy

on:
  workflow_dispatch:

jobs:
  sync_with_upstream:
    name: Sync with Upstream
    runs-on: ubuntu-latest
    if: ${{ github.event.repository.fork }}

    steps:
      - name: Checkout target repo
        uses: actions/checkout@v4

      - name: Sync Upstream
        uses: aormsby/Fork-Sync-With-Upstream-action@v3.4
        with:
          target_repo_token: ${{ secrets.GITHUB_TOKEN }}
          upstream_sync_repo: trickest/cve
          upstream_sync_branch: main
          target_sync_branch: main
          test_mode: false

      - name: Check for Failure
        if: failure()
        run: |
          echo "[Error] Due to a change in the workflow file of the upstream repository, GitHub has automatically suspended the scheduled automatic update. You need to manually sync your fork."
          exit 1

      - name: Generate Mkdocs Yaml File
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      
      - run: |
          python -m pip install --upgrade pip
          pip install pyyaml
          python mkdocs_yaml_gen.py

      - name: Conver Markdown to Site
        run: |
          mkdir docs
          rsync -av --include='[0-9][0-9][0-9][0-9]/' --include='[0-9][0-9][0-9][0-9]/*.md' --exclude='*' --exclude="docs/" --exclude="summary_html/"  . docs/
          echo '<!DOCTYPE html><html><head><meta http-equiv="refresh" content="0; url=/docs/README.html" /></head></html>' > index.html

      - name: Deploy docs
        uses: mhausenblas/mkdocs-deploy-gh-pages@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          CONFIG_FILE: ./mkdocs.yml
```
## 使用 `nektos/act` 本地运行 GitHub Actions 的优化版教程

### 配置 GitHub Secrets

在 GitHub Actions 中，敏感信息通常通过 `secrets` 管理。`nektos/act` 提供了通过 `.env` 文件或命令行传递 `secrets` 的便捷方式。
创建 `.env` 文件：在项目根目录新建一个 `.env` 文件，用于存储敏感信息（如 GitHub token）。确保 `.env` 文件不会被提交到版本控制系统（建议在 `.gitignore` 中添加 `.env`）。
```env
GITHUB_TOKEN=your_github_token
```
通过命令行设置 `secrets`：也可以直接通过 `-s` 参数在运行时传递 `secrets`，无需创建 `.env` 文件。
```bash
act -s GITHUB_TOKEN=your_github_token
```
### 本地运行工作流

笔者建议先确保项目中已有 GitHub Actions 工作流文件（通常位于 `.github/workflows` 目录下）。然后使用以下命令本地运行：

```bash
act workflow_dispatch
```

`workflow_dispatch` 事件模拟了 GitHub 上手动触发工作流的功能，相当于在 GitHub 界面点击“Run workflow”按钮。

### 实时查看日志输出

运行后，`act` 会打印每个步骤的详细日志，包括成功或失败信息。如果某个步骤出错，日志中会显示详细的错误提示，有助于快速定位问题。

### 调试技巧

在调试 GitHub Actions 工作流时，笔者建议通过增加调试步骤获取更多环境信息。例如：

```yaml
- name: Debug Environment
  run: |
    echo "Current working directory: $(pwd)"
    echo "Files in directory:"
    ls -la
    echo "Environment variables:"
    env
```

本地运行时，`act` 将输出上述信息，便于分析工作流行为是否符合预期。

### 常见问题及解决方法

1. **镜像问题：** 默认情况下，`act` 会下载一个通用的 Docker 镜像。如果需要自定义镜像（如支持特殊环境的镜像），可以使用 `-P` 参数指定：
```bash
act -P ubuntu-latest=custom-image:tag
```
2. **网络依赖问题：** 如果工作流步骤依赖外部服务（如 API 请求），请确保本地网络环境与 GitHub 环境一致，或者在本地搭建模拟服务。
3. **权限不足：** 如果需要额外权限（如访问私有仓库），请确保使用的 `GITHUB_TOKEN` 拥有足够的权限，并正确配置工作流。


