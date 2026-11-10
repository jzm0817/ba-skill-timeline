# Windows 开发环境安装清单

适用于 Windows x64。以下命令在 PowerShell 中运行。两台电脑各自安装系统工具并重建项目依赖。

## 1. 系统工具

| 工具 | 用途 | 安装来源 |
|---|---|---|
| WinGet | 下载并校验 Windows 安装包 | Windows App Installer |
| Git for Windows | 代码和文档版本管理 | `Git.Git` |
| CPython 3.13 x64 | Python 后端，附带 pip、venv 和 SQLite 接口 | `Python.Python.3.13` |
| Node.js LTS x64 | 前端运行环境，附带 npm | `OpenJS.NodeJS.LTS` |
| FFmpeg Essentials | 媒体处理和检查，包含 `ffmpeg` 和 `ffprobe` | `Gyan.FFmpeg.Essentials` |

```powershell
winget --version
winget install --exact --id Git.Git --source winget
winget install --exact --id Python.Python.3.13 --source winget
winget install --exact --id OpenJS.NodeJS.LTS --source winget
winget install --exact --id Gyan.FFmpeg.Essentials --source winget
```

安装后重新打开 PowerShell。已运行的 Codex 进程可能仍持有安装前的 PATH，必要时重启 Codex。PowerShell 中使用 `npm.cmd` 和 `npx.cmd` 可直接运行 Windows 命令入口。

本机已通过版本命令验证 Git 2.56.0.windows.4、Python 3.13.17、pip 26.2.2、Node.js 26.13.0、npm 11.20.0、FFmpeg 和 ffprobe 9.0.1。版本来自本次安装，其他电脑的 WinGet 源可能提供更新版本。

## 2. 项目依赖

| 环境 | 软件包 | 用途 |
|---|---|---|
| Python `.venv` | NumPy、OpenCV Headless、PyAV | 数值计算、ROI、视频解码 |
| Python `.venv` | FastAPI、Uvicorn | API 和本地 Web 服务 |
| Python `.venv` | pytest、HTTPX | 后端单元测试与 API 测试 |
| 前端 `node_modules` | React、React DOM、TypeScript、Vite、React 插件与类型声明、Rolldown Windows x64 原生包 | 前端开发和构建 |
| 前端 `node_modules` | Playwright Test 和 Microsoft Edge | 界面端到端测试。可使用 Windows 上已安装的 Edge。 |

从项目根目录执行。本机的 Playwright 已验证可用 Edge 通道运行。其他电脑应先确认已安装 Edge，也可选用 Playwright 管理的独立 Chromium。

```powershell
python --version
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install --only-binary=:all: -r requirements-dev.txt
.\.venv\Scripts\python.exe -m pip check
Push-Location frontend
try {
    npm.cmd ci
} finally {
    Pop-Location
}
```

先确认 `python --version` 返回 3.13.x。`py -3.13` 在本机普通 PowerShell 中验证通过，Codex 沙箱中可直接使用 `python.exe` 或 `.venv` 中的解释器。Python 依赖的精确版本见 [requirements-dev.txt](../requirements-dev.txt)，前端依赖以 `frontend/package-lock.json` 为准。

在 Playwright 项目配置中可用 `use: { channel: "msedge" }` 选择 Edge。若要用 Playwright 管理的独立 Chromium，在 `frontend` 执行 `npx.cmd playwright install chromium`。仅需无头版时可执行 `npx.cmd playwright install --only-shell chromium`。本机首次下载独立 Chromium 时出现连接中断和 DNS 错误，重试完整包和无头版均未完成。Edge 已验证可用。

SQLite 通过 Python 的 `sqlite3` 使用。OCR 引擎、模型、GPU 工具和浏览器品牌支持范围在实现阶段按验证结果确定。真实视频和移动硬盘不是环境冒烟检查的前置条件。需要处理真实视频时，将 `BA_DATA_DIR` 设为本机数据根目录。

## 3. 环境检查

- Python 的 `pip check` 通过，`python -m pytest --version` 返回 9.2.0。使用 `pytest -q -s -p no:cacheprovider` 对临时冒烟用例执行测试，1 项通过。
- 使用 FFmpeg 在内存中生成两帧 MP4，用 PyAV 解码并检查 PTS，再用 NumPy 和 OpenCV 验证图像转换。
- SQLite 3.50.4 内存查询通过。FastAPI 通过 HTTPX 的 ASGI 传输执行进程内请求，Uvicorn 0.54.0 可导入。
- `npm ls --depth=0`、TypeScript 7.0.2 和 Vite 8.3.7 命令检查通过。首次安装跳过了 Rolldown 的 Windows x64 原生包，已按匹配版本 `1.2.18` 显式锁定。补装后，Vite 成功在内存中构建 React/TSX 入口。当前前端锁定配置面向 Windows x64。
- Playwright 1.65.0 通过 `chromium.launch({ channel: "msedge", headless: true })` 启动 Edge 157.0.4341.42，读取本地内联页面标题成功。

## 4. 上游安装说明

- [WinGet 安装命令](https://learn.microsoft.com/windows/package-manager/winget/install)
- [Python 在 Windows 上的安装与使用](https://docs.python.org/3.13/using/windows.html)
- [Node.js 下载](https://nodejs.org/en/download)
- [FFmpeg 下载和 Windows 构建来源](https://ffmpeg.org/download.html)
- [Vite 运行要求](https://vite.dev/guide/)和 [Playwright 浏览器安装](https://playwright.dev/docs/browsers)
- [Rolldown 原生包故障排查](https://rolldown.rs/guide/troubleshooting)
