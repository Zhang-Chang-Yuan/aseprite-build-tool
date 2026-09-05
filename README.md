# 🛠️ aseprite-build-tool

## 🇨🇳 中文

### 📖 项目简介

本项目通过 **GitHub Actions** 从 [aseprite/aseprite](https://github.com/aseprite/aseprite) 官方源码自动编译 Aseprite，并将构建产物发布为本仓库的 **Draft Release（草稿发布）**，以规避 Aseprite EULA 对公开分发二进制文件的限制。

- 支持手动触发构建指定版本 / 指定平台
- 支持一键批量补齐所有缺失的「版本×平台」组合
- **仅构建 v1.3.14 及之后的版本**（原因见下方「版本范围说明」）
- 产物仅以草稿形式发布，仅仓库所有者可见

### ✨ 徽章

> 请将下方链接中的用户名/仓库名替换为你的实际仓库。

![Build and Release](https://github.com/Zhang-Chang-Yuan/aseprite-build-tool/actions/workflows/build_and_release.yml/badge.svg)

### 🎯 版本范围说明（重要）

本项目**仅构建 [v1.3.14](https://github.com/aseprite/aseprite/releases/tag/v1.3.14) 及之后发布的版本**，v1.3.13 及更早的版本一律忽略，原因如下：

1. **旧版本依赖的 Skia 没有预编译包**：从 v1.3.14 起，Aseprite 官方的 `laf` 子模块统一固定使用 [aseprite/skia](https://github.com/aseprite/skia) 的 `m124` 预编译版本（Windows x64 / macOS arm64 / Linux x64 三平台包齐全）。而更早的版本分别依赖 m81 / m84 / m96 / m102 等旧版 Skia，甚至根本没有对应的预编译产物，无法直接在 CI 上编译；
2. **更老的版本没有 Skia 后端**：v1.1.x 及更早版本（2017 年以前）的源码还没有 `laf`/Skia 渲染后端，构建体系完全不同；
3. **成本考虑**：在 CI 上为每个旧版本单独搭建旧工具链成本极高且几乎必然失败，故不做尝试。

官方未来发布的新版本（如 v1.3.19、v1.4.0）只要仍使用可用的预编译 Skia，就会在勾选 build_all 运行时被检测并构建。范围阈值由工作流中的 `MIN_VERSION` 环境变量控制，可自行修改。

### 🚀 快速开始

#### 手动触发

1. 进入仓库主页，点击 **Actions** 标签页；
2. 在左侧工作流列表中选择 **Build and Release Aseprite**；
3. 点击右侧 **Run workflow** 按钮；
4. 填写参数：
   - **Aseprite 版本 tag**（可选）：如 `v1.3.18.3`（仅支持 v1.3.14 及以上）。⚠️ 由于 GitHub 原生不支持动态下拉列表，需手动输入版本 tag；**留空则自动构建最新稳定版**；
   - **平台**：下拉选择 `windows-latest` / `macos-latest` / `ubuntu-latest`，默认 `windows-latest`；
   - **构建全部缺失版本（build_all）**：勾选后等同定时任务逻辑，一次性把所有缺失的版本在三个平台上全部构建；
5. 点击 **Run workflow** 开始构建。

#### 批量构建（build_all）

在手动触发时勾选 **构建全部缺失的版本×平台组合（build_all）**，工作流会对比 [aseprite/aseprite](https://github.com/aseprite/aseprite) 官方 tags（仅 v1.3.14 及之后的稳定版本）与本仓库已有 Release（含草稿），把所有缺失的「版本×平台」组合一次性构建补齐。已构建过的组合不会重复构建；再次构建同一产物时会以覆盖方式上传，保证始终是最新构建。

> ℹ️ 本项目**未启用定时构建**（schedule），一切构建均由手动触发。如需每日自动检测新版本，可在 `.github/workflows/build_and_release.yml` 中自行添加 `schedule` 触发器（与 build_all 逻辑完全兼容）。

### ⚠️ 注意事项

- 📝 **Draft Release 仅仓库所有者/协作者可见**，其他人无法访问；
- ⚖️ 根据 [Aseprite EULA](https://github.com/aseprite/aseprite/blob/main/EULA.txt)，构建产物**仅限个人使用**，请勿公开分发；
- ⏱️ 注意 GitHub Actions 的分钟数限制（免费额度公共仓库无限，私有仓库有限额）；
- 🕐 单次构建耗时约 **15–60 分钟**（视平台而定）；
- 🐧 Linux 版为动态链接（glibc / OpenSSL 3），需要较新的发行版；Windows / macOS 产物已内置所需运行库（DLL / dylib）。

### ❓ FAQ

**Q: 为什么只支持 v1.3.14 及以上版本，旧版本能构建吗？**
A: 不能。v1.3.14 起官方统一使用 aseprite/skia 的 m124 预编译 Skia（三平台齐全）；更早版本依赖的旧版 Skia 没有预编译包，v1.1 及以前甚至没有 Skia 后端，在 CI 上无法构建。详细原因见「版本范围说明」。如确有需要，可修改工作流中的 `MIN_VERSION` 自行尝试。

**Q: 构建失败了怎么办？**
A: 可在 Actions 页面对应运行中点击 **Re-run failed jobs** 重试失败的任务；也可直接手动重新触发一次（同名 Release 会先删除再重建）。

**Q: 为什么版本号需要手动输入，而不是下拉选择？**
A: GitHub Actions 原生不支持在 `workflow_dispatch` 输入中提供动态下拉列表（choice 选项必须写死），因此版本 tag 采用字符串输入，留空即自动使用最新稳定版。

**Q: Windows 版提示缺少 DLL 怎么办？**
A: 当前构建产物已自动打包 `libcrypto-3-x64.dll` / `libssl-3-x64.dll`（OpenSSL 运行库）。若下载的是早期构建的压缩包，可手动从 [Win64 OpenSSL](https://slproweb.com/products/Win32OpenSSL.html) 安装目录复制这两个 DLL 到 aseprite.exe 同级目录，或重新触发一次构建获取新包。若仍提示缺少其它 DLL，请安装 [Visual C++ Redistributable](https://aka.ms/vs/17/release/vc_redist.x64.exe)。

**Q: 如何下载草稿（Draft）Release？**
A: 登录 GitHub 后，进入仓库主页右侧的 **Releases** 栏，或直接访问 `https://github.com/<用户名>/aseprite-build-tool/releases` 页面，草稿条目标记为 *Draft*，点击进入后可下载 Assets 中的产物，也可手动将其发布为正式 Release。

### 📄 许可证说明

本仓库**仅包含 CI 脚本与说明文档**，不包含、不分发任何 Aseprite 源码或二进制文件。Aseprite 源码版权归其原作者所有，构建产物遵循 [Aseprite EULA](https://github.com/aseprite/aseprite/blob/main/EULA.txt)。

---

## 🇬🇧 English

### 📖 About

This project uses **GitHub Actions** to automatically build Aseprite from the official [aseprite/aseprite](https://github.com/aseprite/aseprite) source code and publish the artifacts as **Draft Releases** in this repository, working around the Aseprite EULA restriction on public redistribution.

- Manual trigger with custom version / platform
- One-click batch fill of all missing version × platform combinations
- **Only versions v1.3.14 and later are built** (see "Supported Version Range" below)
- Artifacts are published as drafts, visible only to the repository owner

### ✨ Badge

> Replace the username/repository in the URLs below with your own.

![Build and Release](https://github.com/Zhang-Chang-Yuan/aseprite-build-tool/actions/workflows/build_and_release.yml/badge.svg)

### 🎯 Supported Version Range (Important)

This project **only builds versions [v1.3.14](https://github.com/aseprite/aseprite/releases/tag/v1.3.14) and later**; v1.3.13 and older are ignored, for the following reasons:

1. **Older versions depend on Skia builds that are not available**: since v1.3.14, Aseprite's official `laf` submodule pins the `m124` pre-built Skia from [aseprite/skia](https://github.com/aseprite/skia) (with packages for Windows x64 / macOS arm64 / Linux x64). Earlier versions require older Skia releases (m81 / m84 / m96 / m102, etc.) — or none exist at all — which cannot be built directly on CI;
2. **Much older versions have no Skia backend at all**: v1.1.x and earlier (pre-2017) predate the `laf`/Skia rendering backend and use a completely different build system;
3. **Cost**: setting up per-version legacy toolchains on CI for each old release is expensive and almost guaranteed to fail, so it is not attempted.

Future official releases (e.g. v1.3.19, v1.4.0) are detected and built when running with build_all, as long as a usable pre-built Skia exists. The threshold is controlled by the `MIN_VERSION` environment variable in the workflow file.

### 🚀 Quick Start

#### Manual Trigger

1. Go to the repository's **Actions** tab;
2. Select **Build and Release Aseprite** from the workflow list;
3. Click **Run workflow**;
4. Fill in the inputs:
   - **Aseprite version tag** (optional): e.g. `v1.3.18.3` (v1.3.14 and later only). ⚠️ GitHub does not natively support dynamic dropdowns, so type the tag manually; **leave empty to build the latest stable release**;
   - **Platform**: choose `windows-latest` / `macos-latest` / `ubuntu-latest` (default: `windows-latest`);
   - **Build all missing versions (build_all)**: check this to run the same logic as the scheduled task — build every missing version on all three platforms;
5. Click **Run workflow** to start.

#### Batch Build (build_all)

When triggering manually, check **build_all** and the workflow compares the official [aseprite/aseprite](https://github.com/aseprite/aseprite) tags (stable versions v1.3.14 and later only) with existing Releases (including drafts) in this repository, then builds every missing version × platform combination in one run. Already-built combinations are not rebuilt; re-built artifacts are uploaded with overwrite so they are always the latest build.

> ℹ️ This project does **not** use scheduled builds (`schedule`) — everything is triggered manually. If you want daily automatic detection of new versions, add a `schedule` trigger to `.github/workflows/build_and_release.yml` yourself (fully compatible with the build_all logic).

### ⚠️ Notes

- 📝 **Draft Releases are visible only to the repository owner/collaborators**;
- ⚖️ Per the [Aseprite EULA](https://github.com/aseprite/aseprite/blob/main/EULA.txt), built artifacts are **for personal use only** — do not redistribute publicly;
- ⏱️ Be aware of GitHub Actions usage minute limits;
- 🕐 A single build takes about **15–60 minutes** depending on the platform;
- 🐧 The Linux build is dynamically linked (glibc / OpenSSL 3) and needs a reasonably modern distribution; Windows / macOS artifacts bundle their required runtime libraries (DLLs / dylibs).

### ❓ FAQ

**Q: Why are only v1.3.14+ supported? Can older versions be built?**
A: No. From v1.3.14 on, the official project pins the m124 pre-built Skia from aseprite/skia (complete for all three platforms). Older versions require older Skia releases that have no pre-built packages, and v1.1 and earlier predate the Skia backend entirely — none of them can be built on CI. See "Supported Version Range" above; you can tweak `MIN_VERSION` in the workflow if you really want to try.

**Q: What if a build fails?**
A: On the Actions run page, click **Re-run failed jobs** to retry, or simply trigger a manual run again (an existing Release with the same tag is deleted and recreated).

**Q: Why do I have to type the version instead of picking it from a dropdown?**
A: GitHub Actions does not support dynamic option lists in `workflow_dispatch` inputs (choices must be static), so the version is a string input. Leaving it empty builds the latest stable version automatically.

**Q: Windows reports missing DLLs. What should I do?**
A: Current artifacts automatically bundle `libcrypto-3-x64.dll` / `libssl-3-x64.dll` (the OpenSSL runtime). If you downloaded an earlier build, either copy these two DLLs from a [Win64 OpenSSL](https://slproweb.com/products/Win32OpenSSL.html) installation next to `aseprite.exe`, or re-trigger a build to get a fresh archive. If other DLLs are still reported missing, install the [Visual C++ Redistributable](https://aka.ms/vs/17/release/vc_redist.x64.exe).

**Q: How do I download a Draft Release?**
A: Sign in to GitHub, then open the **Releases** section on the repository page or visit `https://github.com/<user>/aseprite-build-tool/releases`. Draft entries are labeled *Draft*; open one to download the assets, or publish it as a full release manually.

### 📄 License Note

This repository **contains only CI scripts and documentation** — no Aseprite source code or binaries are included or distributed. Aseprite source code is copyrighted by its authors; built artifacts are governed by the [Aseprite EULA](https://github.com/aseprite/aseprite/blob/main/EULA.txt).
