任务目标
在当前工作区文件夹下，利用其文件夹名称作为仓库名，在 GitHub 上新建一个仓库。为该仓库配置 GitHub Actions 自动化工作流，实现从 aseprite/aseprite 官方源码编译构建，并发布到 Releases。参考项目：

https://github.com/a1393323447/aseprite-builder

https://github.com/mmozeiko/aseprite-bin

功能需求
手动触发构建：工作流支持 workflow_dispatch，提供两个下拉选择框：

版本选择：动态获取 Aseprite 官方所有 tags，下拉列表供用户选择（也可输入自定义版本）。

平台选择：下拉选项为 Windows、macOS、Linux (Ubuntu)，实际对应 windows-latest、macos-latest、ubuntu-latest。

自动定时构建：每天定时（例如 UTC 2:00）自动检查 Aseprite 官方新发布的 tags。如果发现本仓库尚未构建过的新版本，则自动触发构建 所有三个平台 的产物，并发布到 Releases。

重复构建覆盖：如果同一版本已存在 Release，再次构建（手动或自动）时，先删除旧 Release 和对应 Tag，再创建新的 Draft Release 并上传产物，确保始终为最新构建。

产物管理：构建后的压缩包（.zip 或 .tar.gz）上传至 Draft Release（草稿），仅仓库所有者可见（规避 Aseprite EULA 分发限制）。

说明文档：在仓库根目录创建 README.md，包含中英文双语的操作步骤、注意事项和常见问题。

执行步骤
1. 获取仓库名称
获取当前工作区文件夹的绝对路径，取其最后一级目录名作为新仓库名称。例如，若工作区为 /home/user/my-aseprite-builder，则仓库名为 my-aseprite-builder。

2. 在 GitHub 上创建仓库
使用 GitHub CLI (gh) 或 GitHub API（需配置 GITHUB_TOKEN 环境变量）创建公有仓库（或私有，可根据需要）。确保新仓库为空（不初始化 README、.gitignore 等），后续由 Agent 推送本地代码。

3. 初始化本地 Git 仓库
在工作区执行：

bash
git init
git remote add origin https://github.com/<你的用户名>/<仓库名>.git
4. 创建工作流文件
在 .github/workflows/build_and_release.yml 中写入以下完整 YAML 内容（见下文“工作流代码”）。

5. 创建 README.md
在仓库根目录创建 README.md，内容需包含中英文说明（见下文“说明文档模板”）。

6. 提交并推送
bash
git add .
git commit -m "Initial commit: Aseprite auto-builder workflow"
git push -u origin main
7. 启用 Actions
推送后，GitHub Actions 将自动启用。由于工作流包含 schedule 和 workflow_dispatch，无需额外设置。

工作流代码（完整 YAML）
将此内容保存为 .github/workflows/build_and_release.yml：

yaml
name: Build and Release Aseprite

on:
  workflow_dispatch:
    inputs:
      aseprite_version:
        description: 'Aseprite 版本 (如 v1.3.18，留空则构建最新稳定版)'
        required: false
        default: ''
        type: string
      platform:
        description: '选择构建平台'
        required: true
        default: 'windows-latest'
        type: choice
        options:
          - label: 'Windows'
            value: 'windows-latest'
          - label: 'macOS'
            value: 'macos-latest'
          - label: 'Linux (Ubuntu)'
            value: 'ubuntu-latest'

  schedule:
    # 每天 UTC 2:00 (北京时间 10:00) 运行
    - cron: '0 2 * * *'

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: false

jobs:
  # 自动检测新版本（仅定时任务执行）
  check-and-trigger:
    runs-on: ubuntu-latest
    if: github.event_name == 'schedule'
    outputs:
      missing_versions: ${{ steps.get-missing.outputs.missing }}
    steps:
      - name: 获取本仓库已有 Releases
        id: existing
        run: |
          EXISTING=$(curl -s -H "Authorization: token ${{ secrets.GITHUB_TOKEN }}" \
            https://api.github.com/repos/${{ github.repository }}/releases | jq -r '.[].tag_name')
          echo "existing=$EXISTING" >> $GITHUB_OUTPUT

      - name: 获取 Aseprite 官方所有 Tags
        id: all_tags
        run: |
          TAGS=$(curl -s https://api.github.com/repos/aseprite/aseprite/tags?per_page=100 | jq -r '.[].name')
          echo "tags=$TAGS" >> $GITHUB_OUTPUT

      - name: 计算缺失版本
        id: get-missing
        run: |
          EXISTING="${{ steps.existing.outputs.existing }}"
          ALL_TAGS="${{ steps.all_tags.outputs.tags }}"
          MISSING=""
          for tag in $ALL_TAGS; do
            # 跳过非稳定版（可选，若包含 alpha/beta/rc 则跳过）
            # if [[ $tag == *"alpha"* ]] || [[ $tag == *"beta"* ]] || [[ $tag == *"rc"* ]]; then continue; fi
            if ! echo "$EXISTING" | grep -q "$tag"; then
              MISSING="${MISSING} ${tag}"
            fi
          done
          MISSING_TRIMMED=$(echo $MISSING | xargs)
          if [ -z "$MISSING_TRIMMED" ]; then
            echo "missing=[]" >> $GITHUB_OUTPUT
          else
            JSON_ARRAY=$(echo $MISSING_TRIMMED | sed 's/ /","/g' | sed 's/^/["/' | sed 's/$/"]/')
            echo "missing=$JSON_ARRAY" >> $GITHUB_OUTPUT
          fi

  # 构建任务（手动触发 或 定时发现的缺失版本）
  build:
    if: |
      github.event_name == 'workflow_dispatch' || 
      (github.event_name == 'schedule' && needs.check-and-trigger.outputs.missing_versions != '[]')
    runs-on: ${{ matrix.platform }}
    needs: [check-and-trigger]
    strategy:
      matrix:
        version: ${{ 
          github.event_name == 'workflow_dispatch' && 
          fromJSON(format('["{0}"]', github.event.inputs.aseprite_version || 'latest')) || 
          fromJSON(needs.check-and-trigger.outputs.missing_versions) 
        }}
        platform: ${{ 
          github.event_name == 'workflow_dispatch' && 
          fromJSON(format('["{0}"]', github.event.inputs.platform)) || 
          fromJSON('["windows-latest", "ubuntu-latest", "macOS-latest"]') 
        }}
    env:
      SKIA_VERSION: m124

    steps:
      - name: 解析最终版本号
        id: resolve_version
        run: |
          if [ "${{ matrix.version }}" == "latest" ]; then
            VERSION=$(curl -s https://api.github.com/repos/aseprite/aseprite/releases/latest | jq -r '.tag_name')
          else
            VERSION="${{ matrix.version }}"
          fi
          echo "VERSION=$VERSION" >> $GITHUB_ENV
          echo "version=$VERSION" >> $GITHUB_OUTPUT

      - name: 克隆 Aseprite 源码
        run: |
          git clone --recursive --branch ${{ env.VERSION }} https://github.com/aseprite/aseprite.git aseprite

      - name: 安装依赖 (Windows)
        if: runner.os == 'Windows'
        run: choco install cmake ninja openssl -y

      - name: 安装依赖 (Ubuntu)
        if: runner.os == 'Linux'
        run: |
          sudo apt-get update
          sudo apt-get install -y g++ clang cmake ninja-build libx11-dev \
            libxcursor-dev libxi-dev libxrandr-dev libgl1-mesa-dev \
            libfontconfig1-dev unzip

      - name: 安装依赖 (macOS)
        if: runner.os == 'macOS'
        run: brew install cmake ninja

      - name: 下载 Skia
        run: |
          OS_LOWER=$(echo "${{ runner.os }}" | tr '[:upper:]' '[:lower:]')
          if [ "${{ runner.os }}" == "Windows" ]; then
            curl -L -o skia.zip https://github.com/aseprite/skia/releases/download/${{ env.SKIA_VERSION }}/Skia-${{ env.SKIA_VERSION }}-windows.zip
          else
            curl -L -o skia.zip https://github.com/aseprite/skia/releases/download/${{ env.SKIA_VERSION }}/Skia-${{ env.SKIA_VERSION }}-${OS_LOWER}.zip
          fi
          unzip skia.zip -d skia

      - name: CMake 配置与 Ninja 构建
        run: |
          mkdir build && cd build
          cmake ../aseprite -G Ninja -DCMAKE_BUILD_TYPE=RelWithDebInfo \
            -DLAF_BACKEND=skia \
            -DSKIA_DIR=$PWD/../skia \
            -DSKIA_LIBRARY_DIR=$PWD/../skia/lib \
            -DSKIA_LIBRARY=$PWD/../skia/lib/libskia.a
          ninja

      - name: 打包 (Windows)
        if: runner.os == 'Windows'
        run: |
          mkdir artifact
          copy build\bin\aseprite.exe artifact\
          # 复制 OpenSSL DLL（假设安装在默认路径）
          copy "C:\Program Files\OpenSSL-v3-Win64\bin\*.dll" artifact\ || echo "跳过OpenSSL复制"
          7z a -r aseprite-${{ env.VERSION }}-win64.zip .\artifact\*

      - name: 打包 (Ubuntu)
        if: runner.os == 'Linux'
        run: |
          mkdir artifact
          cp build/bin/aseprite artifact/
          ldd build/bin/aseprite | grep "=> /" | awk '{print $3}' | xargs -I '{}' cp '{}' artifact/
          tar -czvf aseprite-${{ env.VERSION }}-linux-x64.tar.gz -C artifact .

      - name: 打包 (macOS)
        if: runner.os == 'macOS'
        run: |
          mkdir artifact
          cp build/bin/aseprite artifact/
          cd artifact && zip -r ../aseprite-${{ env.VERSION }}-macos-x64.zip . && cd ..

      - name: 发布或替换 Release
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          VERSION="${{ env.VERSION }}"
          # 查找产物文件
          FILE_PATH=$(ls aseprite-$VERSION-*.zip 2>/dev/null || ls aseprite-$VERSION-*.tar.gz 2>/dev/null)
          if [ -z "$FILE_PATH" ]; then
            echo "未找到产物文件"
            exit 1
          fi
          # 如果 Release 已存在则删除
          if gh release view $VERSION --repo ${{ github.repository }} &>/dev/null; then
            echo "Release $VERSION 已存在，正在删除..."
            gh release delete $VERSION --repo ${{ github.repository }} --yes
            git push --delete origin $VERSION || true
          fi
          # 创建新 Draft Release
          gh release create $VERSION \
            --repo ${{ github.repository }} \
            --title "Aseprite $VERSION (Auto Build)" \
            --draft \
            --notes "自动构建于 $(date +'%Y-%m-%d %H:%M:%S')"
          # 上传产物
          gh release upload $VERSION $FILE_PATH --repo ${{ github.repository }}
说明文档模板（README.md）
创建 README.md，内容如下（中英文双语）：

markdown
# Aseprite 自动构建工具 / Aseprite Auto Builder

[![Build and Release Aseprite](https://github.com/你的用户名/仓库名/actions/workflows/build_and_release.yml/badge.svg)](https://github.com/你的用户名/仓库名/actions/workflows/build_and_release.yml)

## 中文说明

### 📖 简介
本项目通过 GitHub Actions 自动编译 Aseprite 源码，无需本地搭建编译环境。支持手动选择版本和平台，也支持每日定时检测官方新版本并自动构建全平台产物。

### 🚀 快速开始

1. **Fork 本仓库**（若你已拥有，则直接使用）
2. **启用 Actions**：进入仓库 `Actions` 标签页，点击启用。
3. **手动触发构建**：
   - 点击左侧 `Build and Release Aseprite` 工作流
   - 点击 `Run workflow`
   - 选择版本（或留空自动获取最新稳定版）和平台
   - 点击运行，等待约 15-30 分钟
4. **自动构建**：系统每日 UTC 2:00 自动检查 Aseprite 官方新 tags，若发现新版本，会自动构建 Windows、Linux、macOS 三个平台，并发布为 Draft Release。
5. **下载产物**：构建成功后，前往仓库 `Releases` 页面，下载对应压缩包。

> ⚠️ **重要提示**：根据 Aseprite EULA，编译产物仅供个人使用，请勿公开分发。本仓库所有 Release 均设置为 **Draft（草稿）**，仅仓库所有者可见。

### ❓ 常见问题

**Q: 构建失败怎么办？**  
A: 可点击 `Re-run jobs` 重试，或检查依赖下载是否因网络问题中断。

**Q: Windows 运行提示缺少 DLL？**  
A: 请安装 OpenSSL v3（非 Light 版本），安装时选择复制 DLL 到系统目录。

---

## English Instructions

### 📖 Introduction
This project uses GitHub Actions to automatically compile Aseprite from source code. You can manually select a version and platform, or rely on the daily scheduled task to detect new official releases and build for all platforms automatically.

### 🚀 Quick Start

1. **Fork this repository** (or use it directly if you own it).
2. **Enable Actions**: Go to the `Actions` tab and enable workflows.
3. **Manual trigger**:
   - Click the `Build and Release Aseprite` workflow.
   - Click `Run workflow`.
   - Select version (leave blank for latest stable) and platform.
   - Wait about 15-30 minutes for completion.
4. **Automatic builds**: Daily at 2:00 UTC, the system checks for new Aseprite tags. If new versions are found, it automatically builds all three platforms and publishes them as Draft Releases.
5. **Download artifacts**: Go to the `Releases` page and download the archive.

> ⚠️ **Important**: According to the Aseprite EULA, compiled binaries are for personal use only. Do not distribute publicly. All releases here are set as **Draft**, visible only to the repository owner.

### ❓ FAQ

**Q: Build fails?**  
A: Retry by clicking `Re-run jobs`, or check for network issues while downloading dependencies.

**Q: Missing DLL error on Windows?**  
A: Install OpenSSL v3 (non-Light version) and select the option to copy DLLs to system directory.

---

## 许可证 / License
本仓库仅提供构建脚本，不包含 Aseprite 源码或二进制文件。Aseprite 本身遵循其专有 EULA。
注意事项
确保 GitHub 仓库中已设置 GITHUB_TOKEN（默认已存在），用于 API 操作。

若使用私有仓库，需调整 gh release create 的可见性，但 Draft 已足够。

构建时间较长，免费版 GitHub Actions 每月有分钟数限制，建议合理使用。

