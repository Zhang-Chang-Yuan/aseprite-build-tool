# 🛠️ aseprite-build-tool

## 🇨🇳 中文

### 📖 项目简介

本项目通过 **GitHub Actions** 从 [aseprite/aseprite](https://github.com/aseprite/aseprite) 官方源码自动编译 Aseprite，并将构建产物发布为本仓库的 **Draft Release（草稿发布）**，以规避 Aseprite EULA 对公开分发二进制文件的限制。

- 支持手动触发构建指定版本 / 指定平台
- 支持每日自动检测官方新稳定版本并全平台构建
- 产物仅以草稿形式发布，仅仓库所有者可见

### ✨ 徽章

> 请将下方链接中的用户名/仓库名替换为你的实际仓库。

![Build and Release](https://github.com/Zhang-Chang-Yuan/aseprite-build-tool/actions/workflows/build_and_release.yml/badge.svg)

### 🚀 快速开始

#### 手动触发

1. 进入仓库主页，点击 **Actions** 标签页；
2. 在左侧工作流列表中选择 **Build and Release Aseprite**；
3. 点击右侧 **Run workflow** 按钮；
4. 填写参数：
   - **Aseprite 版本 tag**（可选）：如 `v1.3.10.2`。⚠️ 由于 GitHub 原生不支持动态下拉列表，需手动输入版本 tag；**留空则自动构建最新稳定版**；
   - **平台**：下拉选择 `windows-latest` / `macos-latest` / `ubuntu-latest`，默认 `windows-latest`；
5. 点击 **Run workflow** 开始构建。

#### 自动构建

工作流每天 **UTC 2:00**（北京时间 10:00）自动运行，通过 GitHub API 检测 [aseprite/aseprite](https://github.com/aseprite/aseprite) 官方新增的稳定版本 tag（自动跳过 alpha / beta / rc / dev / SNAPSHOT 等预发布版本），与本仓库已有 Release（含草稿）对比后，对缺失的新版本自动构建全部三个平台并发布草稿。

### ⚠️ 注意事项

- 📝 **Draft Release 仅仓库所有者/协作者可见**，其他人无法访问；
- ⚖️ 根据 [Aseprite EULA](https://github.com/aseprite/aseprite/blob/main/EULA.txt)，构建产物**仅限个人使用**，请勿公开分发；
- ⏱️ 注意 GitHub Actions 的分钟数限制（免费额度公共仓库无限，私有仓库有限额）；
- 🕐 单次构建耗时约 **15–60 分钟**（视平台而定）；
- 🔄 定时任务可能因 GitHub 负载出现几分钟到几十分钟的延迟，属正常现象。

### ❓ FAQ

**Q: 构建失败了怎么办？**
A: 可在 Actions 页面对应运行中点击 **Re-run failed jobs** 重试失败的任务；也可直接手动重新触发一次（同名 Release 会先删除再重建）。

**Q: 为什么版本号需要手动输入，而不是下拉选择？**
A: GitHub Actions 原生不支持在 `workflow_dispatch` 输入中提供动态下拉列表（choice 选项必须写死），因此版本 tag 采用字符串输入，留空即自动使用最新稳定版。

**Q: Windows 版提示缺少 DLL 怎么办？**
A: 压缩包内已包含 `build/bin` 下的全部内容（含运行所需 DLL 与 data 目录）。若仍提示缺少 DLL，请确认解压后未删除任何文件，并安装 [Visual C++ Redistributable](https://aka.ms/vs/17/release/vc_redist.x64.exe)。

**Q: 如何下载草稿（Draft）Release？**
A: 登录 GitHub 后，进入仓库主页右侧的 **Releases** 栏，或直接访问 `https://github.com/<用户名>/aseprite-build-tool/releases` 页面，草稿条目标记为 *Draft*，点击进入后可下载 Assets 中的产物，也可手动将其发布为正式 Release。

### 📄 许可证说明

本仓库**仅包含 CI 脚本与说明文档**，不包含、不分发任何 Aseprite 源码或二进制文件。Aseprite 源码版权归其原作者所有，构建产物遵循 [Aseprite EULA](https://github.com/aseprite/aseprite/blob/main/EULA.txt)。

---

## 🇬🇧 English

### 📖 About

This project uses **GitHub Actions** to automatically build Aseprite from the official [aseprite/aseprite](https://github.com/aseprite/aseprite) source code and publish the artifacts as **Draft Releases** in this repository, working around the Aseprite EULA restriction on public redistribution.

- Manual trigger with custom version / platform
- Daily automatic detection of new stable official tags, building all three platforms
- Artifacts are published as drafts, visible only to the repository owner

### ✨ Badge

> Replace the username/repository in the URLs below with your own.

![Build and Release](https://github.com/Zhang-Chang-Yuan/aseprite-build-tool/actions/workflows/build_and_release.yml/badge.svg)

### 🚀 Quick Start

#### Manual Trigger

1. Go to the repository's **Actions** tab;
2. Select **Build and Release Aseprite** from the workflow list;
3. Click **Run workflow**;
4. Fill in the inputs:
   - **Aseprite version tag** (optional): e.g. `v1.3.10.2`. ⚠️ GitHub does not natively support dynamic dropdowns, so type the tag manually; **leave empty to build the latest stable release**;
   - **Platform**: choose `windows-latest` / `macos-latest` / `ubuntu-latest` (default: `windows-latest`);
5. Click **Run workflow** to start.

#### Automatic Builds

The workflow runs daily at **UTC 2:00**, checks the official [aseprite/aseprite](https://github.com/aseprite/aseprite) tags via the GitHub API (skipping pre-release tags such as alpha / beta / rc / dev / SNAPSHOT), compares them with existing Releases (including drafts) in this repository, and automatically builds any missing versions on all three platforms.

### ⚠️ Notes

- 📝 **Draft Releases are visible only to the repository owner/collaborators**;
- ⚖️ Per the [Aseprite EULA](https://github.com/aseprite/aseprite/blob/main/EULA.txt), built artifacts are **for personal use only** — do not redistribute publicly;
- ⏱️ Be aware of GitHub Actions usage minute limits;
- 🕐 A single build takes about **15–60 minutes** depending on the platform;
- 🔄 Scheduled runs may be delayed by a few minutes under GitHub load — this is normal.

### ❓ FAQ

**Q: What if a build fails?**
A: On the Actions run page, click **Re-run failed jobs** to retry, or simply trigger a manual run again (an existing Release with the same tag is deleted and recreated).

**Q: Why do I have to type the version instead of picking it from a dropdown?**
A: GitHub Actions does not support dynamic option lists in `workflow_dispatch` inputs (choices must be static), so the version is a string input. Leaving it empty builds the latest stable version automatically.

**Q: Windows reports missing DLLs. What should I do?**
A: The archive contains the full `build/bin` directory (including required DLLs and the data folder). If DLLs are still reported missing, make sure no files were deleted after extraction, and install the [Visual C++ Redistributable](https://aka.ms/vs/17/release/vc_redist.x64.exe).

**Q: How do I download a Draft Release?**
A: Sign in to GitHub, then open the **Releases** section on the repository page or visit `https://github.com/<user>/aseprite-build-tool/releases`. Draft entries are labeled *Draft*; open one to download the assets, or publish it as a full release manually.

### 📄 License Note

This repository **contains only CI scripts and documentation** — no Aseprite source code or binaries are included or distributed. Aseprite source code is copyrighted by its authors; built artifacts are governed by the [Aseprite EULA](https://github.com/aseprite/aseprite/blob/main/EULA.txt).
