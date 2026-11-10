# 项目开发进度与跨电脑交接记录

## 1. 当前状态

| 项目 | 记录 |
|---|---|
| 项目 | `ba-skill-timeline` |
| 最近更新 | 2026-11-10 |
| 当前阶段 | Phase 1 工程基础准备中，业务实现尚未开始 |
| 工作区 | 已有项目文档、依赖锁定配置、后端包目录、测试目录、前端源码目录和 pytest 配置。业务实现尚未开始。 |
| Git | 本地仓库使用 `main`，远端目标为 [jzm0817/ba-skill-timeline](https://github.com/jzm0817/ba-skill-timeline)。 |
| 安装清单 | 见 [Windows 开发环境安装清单](setup.md)。 |

需求以 `requirements.md` 为准，设计和待确认项见 `design.md`。

## 2. 阶段任务

| 阶段 | 工作范围 | 完成依据 | 状态 |
|---|---|---|---|
| Phase 1 | 工程基础、角色配置、FIFO 与候选约束、视频解码及 ROI | 状态机、重放和视频基础自动化测试通过 | 进行中 |
| Phase 2 | Cost、技能费用、槽位与补位识别，双阶段扫描及事件证据 JSON | 多帧事件可复核，冲突和未知保留，JSON 可定位证据 | 未开始 |
| Phase 3 | 恢复量推断、连续分组、Markdown 与 JSON 导出 | 阈值边界、未知断组、精确字节格式及导出拦截测试通过 | 未开始 |
| Phase 4 | FastAPI 与 React 界面、ROI 标定、审核、重算与导出 | 端到端验证从本地视频标定到人工修正并导出 | 未开始 |
| Phase 5 | 授权标签页捕获、录制保存与统一分析 | 支持范围内完成授权、取消、停止及录制文件解码验证 | 未开始 |
| Phase 6 | 真实视频标注、准确率评估与优化 | 建立真实素材评估基线，量化通过线在样本基线后确认 | 未开始 |

## 3. Phase 1 实施清单

按下列顺序交付独立可验证的小任务。

1. **工程入口与配置。** 已安装 Git、Python 3.13 和媒体依赖，初始化本地 Git，建立 `backend/src/ba_skill_timeline/`、`backend/tests/` 和 `frontend/src/`。根目录 `pyproject.toml` 配置 pytest，`.env.example` 预留 `BA_DATA_DIR`。运行时读取与校验数据目录将在视频基础任务实现，前端业务工程接入安排在 Phase 4。
2. **角色与初始状态。** 定义六个稳定编号、可选显示名称、三槽位与三项等待队列。测试角色集合、唯一性、空名回退、单候选已知模式和六候选未知模式。
3. **FIFO 与重放。** 实现已确认事件驱动的状态转换、可靠补位约束、候选一致性推导、冲突与快照。测试只变更被释放槽位、候选收窄、无可靠证据时保留未知、冲突不发布后续身份、人工修正后确定性重放。
4. **视频与 ROI。** 增加只读打开、元数据与 PTS 定位、按时间抽帧及归一化 ROI 裁剪。验证缺失数据目录、缺失文件、越界 ROI、不同尺寸和非固定帧间隔。使用可共享的小型素材或运行时生成的测试素材，另行记录实际移动硬盘和授权视频验证结果。
5. **阶段复核。** 运行适用的 pytest 测试，记录实际命令、结果与未运行项，检查差异和 Git 状态，更新本文件。

下一项最小任务是实现角色配置与 `SkillCycle`，并添加相应的单元测试。视频解码与 ROI 可在后续小任务接入。真实样本、分组口径和录制细节分别在相关阶段确认。

## 4. 本次验证与交接

- `Get-ChildItem -Force`、`rg --files --hidden -g '!.git'` 和文档读取用于核对初始目录及内容。对当前目录及其上级检查 `.git` 文件或目录，未发现。
- `git --version` 返回 2.56.0.windows.4。`git init --initial-branch=main` 成功。首次提交范围为工程骨架、环境配置、依赖清单和项目文档。
- 文档检查通过。使用 PowerShell 的 `ReadAllBytes`、严格 UTF-8 解码和正则检查核对文档编码、行尾空白、七个模块、六组验收条件、技能轴样例与六个阶段状态。
- `.venv\Scripts\python.exe -m pip check` 通过，`.venv\Scripts\python.exe -m pytest --version` 返回 9.2.0。合成 MP4 解码、SQLite 查询和 FastAPI 进程内请求通过。尚无业务测试或真实视频准确率结果。
- `pytest -q -s -p no:cacheprovider` 对临时安装冒烟用例执行 1 项测试并通过。Playwright 通过 `msedge` 通道启动本机 Edge 并成功读取测试页面标题。独立 Chromium 下载遇到连接中断和 DNS 错误。
- `npm ls --depth=0`、`tsc --version` 和 `playwright --version` 通过。补装 Windows x64 的 Rolldown 原生包后，`vite --version` 和 React/TSX 内存构建通过。
- `.venv\Scripts\python.exe -m pytest -q -p no:cacheprovider` 通过 1 项临时冒烟测试，确认 pytest 能发现 `backend/tests/` 并导入后端包。临时测试已移除，尚无业务测试。
- `git check-ignore -v` 验证了虚拟环境、前端依赖、真实 `.env` 和视频路径会被忽略，`.env.example` 可纳入 Git。
- 本次文档任务及 Phase 1 状态机测试无需移动硬盘。真实视频、标注及大型证据尚未接入。两台电脑各自配置 `BA_DATA_DIR`，按 `videos/`、`annotations/`、`evidence/`、`outputs/` 存放数据，引用使用相对路径。

### 下一次 Codex 启动指令

> 阅读 `AGENTS.md`、项目文档和安装清单，复查 Git 状态。使用已建立的后端和 pytest 目录，实现角色配置和 `SkillCycle`，覆盖完整已知、未知队列、补位过滤、冲突与人工修正重放。运行测试并更新 `docs/progress.md`。

## 5. 开发会话历史

### 2026-11-10

- 检查现有工作区和 Git 可用性，确认初始目录只有规范和进度模板。
- 按用户启动要求及项目规范写明功能、验收、架构、阶段和待确认项。业务实现仍在 Phase 1 开始前。
- 用户确认是新环境和新项目。安装 Windows 系统工具与 Python、前端开发依赖，记录版本、复现命令与环境检查。
- 使用锁定的 Playwright 1.65.0 重试 `playwright install chromium`，并尝试 `playwright install --only-shell chromium`。下载均长时间无有效进展，停止后未执行独立 Chromium 启动测试。
- 初始化本地 `main` 仓库，建立工程目录、pytest 配置和数据路径模板，验证后端包导入及 Git 忽略规则。
