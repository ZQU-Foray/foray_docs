# 仓库结构

> 上位机软件的仓库划分、依赖方向、版本锁定与协作规范。
> 配套：[`algorithm_structure.md`](algorithm_structure.md)（算法结构）

---

## 0. 目标仓拓扑

```
                    ┌──────────────────────────────────────────────┐
                    │  foray_ws  （元仓 · 零业务代码）              │
                    │  .repos 锁定 · 集成 CI · 一键拉取/构建/启动   │
                    │  整机 tag = 可复现的「那一台能跑的车」        │
                    └───────────────────┬──────────────────────────┘
                                        │ 装配引用（pin commit，不用分支）
     ┌──────────────┬───────────────────┼───────────────────┬──────────────┐
     ▼              ▼                   ▼                   ▼              ▼
 foray_robots  foray_decision    foray_auto_aim     foray_navigation   (第三方 pin)
 R 装配根        L5 决策三层        L4 自瞄             L4 导航          point_lio 等
 + 地图资产      + L2 横向协同      (哨兵/步兵/英雄共用)
     │              │                   │                   │
     └──────┬───────┴─────────┬─────────┴─────────┬─────────┘
            ▼                 ▼                   ▼
      foray_vision    foray_localization        ...
      L3 目标感知      L3 定位 + 地图管理
      + 态势融合       + 配准残差
            │                 │
            └────────┬────────┘
                     ▼
             foray_platform        L1 平台基础 + L2 下行 HAL
                     │
                     ▼
             foray_interfaces      X2 契约：被所有仓依赖，不依赖任何仓
```

**依赖方向严格自上而下。** `foray_interfaces` 为最底层，零自研依赖。

---

## 1. 切割判据

多仓还是单仓、切多细，由可复述的判据决定。

| # | 判据 | 含义 | 违反的后果 |
|---|---|---|---|
| **C1** | 接口 vs 实现 | 接口必须独立成仓，实现不得把接口私有化 | 跨队/跨兵种无法复用；接口变更引发全仓重编译 |
| **C2** | 变更率 | 变更率差异大的模块不共仓 | 高频仓的提交历史淹没稳定仓；review 负担错配 |
| **C3** | 团队边界（Conway） | 一个仓 = 一个 owner 小组 | 两人同改一个仓 → 频繁冲突、互相阻塞 |
| **C4** | 依赖重量 | 依赖重的（TensorRT/OpenCV vs PCL/Ceres）分开 | 所有人被迫装全套依赖；CI 时间爆炸 |
| **C5** | 复用范围 | 被多兵种复用的下沉；只服务单兵种的上浮 | 复用单元被兵种细节污染 |
| **C6** | 跨队共享需求 | 需共享给步兵/其他队的必须独立且稳定 | 权限无法细分；外部依赖你的私有结构 |

> **粒度结论：一个仓 = 一个团队角色 + 一个变更率档位。** 仓内可含多个 ROS 2 package。

**反模式**
- ❌ 每个 ROS 2 package 建一个仓 → 版本矩阵地狱
- ❌ 把测试、launch、config 拆出独立仓 → 它们属于装配根
- ❌ 为了「看起来整齐」按分层机械切仓 → 会切断团队边界

---

## 2. 起步期：9 仓

一次切 14 个仓会杀死推进力。先立骨架，成熟后再拆。

| # | 仓库 | 层/面 | 内容 | Owner | 变更率 |
|---|---|---|---|---|---|
| 1 | `foray_interfaces` | **X2 契约** | 全部 msg/srv/action：机内接口 + 机间态势协议 + 裁判系统消息 | 架构 | 极低 |
| 2 | `foray_platform` | **L1 + L2-HAL** | 时间时钟 · 数学/滤波 · 参数系统 · 日志录制 · 错误码 + 下行 HAL（底盘/云台/发射/串口/CAN/传感器封装） | 平台 | 低 |
| 3 | `foray_localization` | **L3** | 定位服务（内置地图 + 激光配准）· 地图管理 · 配准残差动态物体检测 | 导航 | 高 |
| 4 | `foray_vision` | **L3** | 目标感知（装甲板检测/跟踪）· 态势融合 · WorldSnapshot 生成 | 视觉 | 高 |
| 5 | `foray_auto_aim` | **L4** | 自瞄（哨兵/步兵/英雄共用）· 多源目标注入 | 视觉 | 高 |
| 6 | `foray_navigation` | **L4** | 静态地图规划 · 动态障碍规避 · 全向控制器 | 导航 | 高 |
| 7 | `foray_decision` | **L5 + L2横向 + X5** | 战术层(规则/RL) · 技能层 · 安全门 · 机间协同 · 对局仿真（临时） | 决策 | 高 |
| 8 | `foray_robots` | **R + 地图资产** | 兵种装配根（`sentry_bringup` / `infantry_bringup`）· 参数 · 场地资产（LFS） | 集成 | 中 |
| 9 | `foray_ws` | **元仓** | `.repos` · 顶层文档 · 集成 CI · 一键脚本。零业务代码 | 架构 | 极低 |

### 2.1 三处取舍

| 取舍 | 理由 |
|---|---|
| **L1 与 L2-HAL 合并为 `foray_platform`** | 都是「被所有人依赖的底层」、变更率都低、owner 同属平台组。符合 C2/C3 |
| **地图资产先放 `foray_robots`（Git LFS）** | 地图随赛季变且是大文件（PCD 可达数十 MB）。起步期放装配根 + LFS 足够 |
| **`foray_decision` 初期临时容纳对局仿真** | 对局仿真是离线工具、变更率高，长期应独立；起步期同一 owner 内，成熟后拆出 |

### 2.2 命名修正（已执行）

原仓名 `foray_sentry_nav` 的语义是「**兵种_功能**」，隐含「哨兵专用」——而**导航恰恰要复用给步兵**，与复用目标直接冲突。

> **按领域命名为 `foray_navigation`**（已改名），兵种差异移到 `foray_robots/sentry_bringup`。
> 判据 C5：复用单元不得被兵种细节污染。

---

## 3. 成熟期：+5 仓

| # | 新增仓 | 从哪拆出 | 触发条件 |
|---|---|---|---|
| 10 | `foray_field` | `foray_robots` 的场地资产 | 地图资产体积/版本频率成为负担 |
| 11 | `foray_comms` | `foray_decision` 的机间协同 | 机间协议迭代频繁，或需开放给其他队 |
| 12 | `foray_battle_sim` | `foray_decision` 的对局仿真 | 红队 RL 正式立项 |
| 13 | `foray_calibration` | `foray_platform` / 各仓零散标定脚本 | 标定需要现场独立运行 |
| 14 | `foray_debug` | 各仓的调试装配根 | 调试根数量膨胀 |

> **拆仓是「因为痛了才拆」，不是「因为整齐才拆」。** 每个新仓必须能回答「不拆会怎样」。

---

## 4. 第三方依赖：fork 与 pin

### 4.1 命名规范

组织既有约定：**`<upstream>-foray-fork`**

| 组织现有 | 本项目需要 |
|---|---|
| `open_vins-foray-fork` | `point_lio-foray-fork` |
| `VINS-Fusion-foray-fork` | `livox_ros_driver2-foray-fork` |
| `ORB_SLAM3-foray-fork` | `small_gicp-foray-fork` |
| `RoboRTS-foray-fork` | `pb_omni_pid_pursuit_controller-foray-fork` |
| | `pb_nav2_plugins-foray-fork` |

### 4.2 处置表

| 类别 | 例子 | 处置 |
|---|---|---|
| 传感器驱动 | `livox_ros_driver2`、`hik_camera_ros2_driver` | fork 到组织下，pin commit |
| 状态估计 | `point_lio`、`small_gicp` | fork，pin 到**指定分支的 commit**（pb2025 用特制分支） |
| 规划控制插件 | `pb_omni_pid_pursuit_controller`、`pb_nav2_plugins` | fork，pin commit |
| ROS 官方 | `nav2_*`、`slam_toolbox`（仅离线建图） | apt 或源码 pin 版本 |

**硬规则**
- 仓库名必须是 `<upstream>-foray-fork`
- **不得依赖分支**（`main`/`dev` 会漂移）——必须 pin commit 或 tag
- **不修改上游代码**；必须改则 fork + 在 `decision.md` 记录 patch 内容与原因，并尽量上游化
- 任何 fork 都要在元仓 `.repos` 里可见，不允许「某台电脑上手动 clone 的版本」

---

## 5. 版本锁定

### 5.1 三层机制

| 层 | 机制 | 说明 |
|---|---|---|
| **仓级** | 语义化版本 + tag | 接口仓尤其严格：`v1.3.0` |
| **引用级** | 元仓 `.repos` **pin commit** | 不用分支，不用「大概最新」 |
| **整机级** | **元仓 meta tag** | 如 `sentry-2026-regional-v1.0.0`，内容 = `.repos` 快照 + 版本清单 |

**规则**
- 实现仓依赖接口仓时，**依赖 tag 而非分支**
- 每次比赛/里程碑打一个整机 tag
- 排障第一步：**确认场上跑的是哪个整机 tag**

### 5.2 用 vcstool 而非 git submodule

| 方案 | 评价 |
|---|---|
| `git submodule` | 原生，但操作繁琐易错（子模块未初始化是新人第一大坑） |
| **`vcstool` + `.repos`** ✅ | SMBU 与 RMOSS 生态均采用（`dependencies.repos`）；对新手友好；一文件描述全部依赖 |

---

## 6. 分支、提交与协作

以组织 [`ZQU-Foray/.github`](https://github.com/ZQU-Foray/.github) 规范为准，本节只做必要追加。

### 6.1 分支

```
main          ← 稳定版本，经过测试与 Review
  └─ dev      ← 开发分支，所有更改在此分支调试
       └─ feat/xxx · fix/xxx · docs/xxx
```

- **代码仓**：所有 PR 的目标分支是 `dev`，不是 `main`。`main` 只承载里程碑级稳定快照
- 功能分支从 `dev` 切出，用 `git rebase dev` 同步上游
- **文档类仓库例外**：以 Markdown / 配置为主、**不含构建产物**的仓（`.github`、`foray_docs`）**不需要三级分支模型**，直接在 `main` 上提交并 PR 到 `main`

> 三级分支的价值在于隔离「可能构建失败的开发中代码」；文档没有构建产物，多一层 `dev` 只是多一次合并。

### 6.2 提交

- **Conventional Commits**：`feat` / `fix` / `docs` / `style` / `refactor` / `perf` / `test` / `chore`
- **scope 按仓库模块划分** → 每个仓定义自己的 scope 词表（见 §6.6）
- 按逻辑单元**小步提交**，不得一次性提交全部代码
- **禁止破坏性操作**：`reset --hard` / `push --force` / 非必要 `rebase`

### 6.3 评审分级

| 仓类型 | 基线 | 追加 |
|---|---|---|
| 所有仓 | ≥1 人 Review | — |
| 视觉 / 电控方向 | 建议**组长** Review | — |
| `foray_interfaces` | ≥1 人 | **≥2 人 + 兼容性影响评估** |
| `foray_ws` 的 `.repos` 变更 | ≥1 人 | **必须说明换版本理由** |

> 追加理由：接口仓与元仓的改动**影响面跨仓**，通用规范的「≥1 人」是为单仓开发设计的。

### 6.4 CI

复用组织 `workflows/ci-template.yml`（`lint-python` ruff+black · `lint-c` clang-format · `test-python` pytest · **`build-ros2`** colcon build/test）。

| 仓 | CI 内容 |
|---|---|
| 所有仓 | 复制组织模板 → `.github/workflows/ci.yml` |
| 所有 ROS 2 仓 | `colcon build` / `colcon test` |
| `foray_interfaces` | + 变更影响面检查（哪些仓依赖被改动的消息） |
| `foray_ws` | **集成 CI**：按 `.repos` 拉全量 → 全量 build → 集成测试 |

### 6.5 依赖方向校验

每仓在仓根声明所属层级：

```
# .foray-layer
layer: L3
depends_on_layers: [L1, L2]
```

CI 校验：
- `L(n)` **不得**依赖 `L(>n)`
- **不得依赖装配根仓**（`foray_robots`）
- `foray_interfaces` 不得依赖任何自研仓

> 这是「功能解耦」从文档变成事实的唯一手段。

### 6.6 代码风格

| 项 | 要求 |
|---|---|
| 缩进 | **4 空格** |
| 换行符 | **LF** |
| 文件末尾 | 保留一个空行 |
| 编码 | UTF-8 |
| Python | `ruff` + `black`（`pyproject.toml`） |
| C/C++ | `clang-format`（`.clang-format`） |
| YAML/JSON | `prettier` |
| Python 命名 | 变量/函数 `snake_case` · 类 `PascalCase` · 常量 `UPPER_SNAKE_CASE` · 私有 `_` 前缀 |
| C/C++ 命名 | 变量/函数 `snake_case` · 结构体/类 `PascalCase` · 宏/枚举 `UPPER_SNAKE_CASE` |

**本项目尤其要执行的两条**
- **公共 API 必须有 Docstring / Doxygen**——ROS 2 话题/服务/参数都算公共 API
- **魔法数字必须注释来源**（规则手册页码、标定值）。区域划分表、各类阈值、地图原点全部属于此类
  （例：`# 2026 规则手册 P.12 补给区判定；2026-03 标定`）

**scope 词表**

| 仓 | scope |
|---|---|
| `foray_interfaces` | `msg` `srv` `action` `referee` `comms` `map` |
| `foray_platform` | `time` `math` `param` `log` `hal` `chassis` `gimbal` `shooter` `serial` `can` |
| `foray_localization` | `map` `relocalize` `icp` `residual` `tf` |
| `foray_vision` | `detector` `classifier` `preprocess` `tracker` `fusion` `snapshot` |
| `foray_auto_aim` | `detector` `tracker` `solver` `aimer` `shooter` `target` |
| `foray_navigation` | `planner` `controller` `costmap` `recovery` |
| `foray_decision` | `tactic` `skill` `safety` `gate` `comms` `rl` `sim` |
| `foray_robots` | `sentry` `infantry` `params` `launch` `field` |
| `foray_ws` | `deps` `ci` `docs` `scripts` |
| `<upstream>-foray-fork` | 不提交（只 pin） |

---

## 7. 仓库文件规范

### 7.1 `README.md`（强制）

**每个仓库必须包含 `README.md`**，无例外。最低内容：

| 内容 | 说明 |
|---|---|
| 一句话定位 + **所属层**（L1–L5 / R / 元仓） | 让新人知道它在架构里的位置 |
| 快速开始 | 拉取 → 依赖安装 → 构建 → 运行 |
| 依赖说明 | 依赖哪些自研仓（含版本）与哪些第三方 |
| 接口说明 | 本仓对外暴露的话题 / 服务 / 参数 |
| 上下游关系 | 上游是谁、下游是谁 |

> 接口仓 `foray_interfaces` 的 `README.md` 尤其关键——**必须写清字段语义 / 单位 / 坐标系 / 版本兼容规则**。

### 7.2 五件套

| 文件 | 组织规范 | 本项目 | 用法 |
|---|---|---|---|
| `README.md` | **强制** | 强制 | 见 §7.1 |
| `docs/` | 必要时 | 架构文档放这里 | 设计文档 |
| `AGENTS.md` | 未要求 | 用 AI Agent 协作的仓必备 | 统一 AI 协作方式 |
| `plan.md` | 未要求 | 建议 | 计划、进度、Before Snapshot |
| `tree.md` | 未要求 | 建议 | 目录结构说明 |
| `decision.md` | 未要求 | **必备** | fork patch、换版本、**破坏性接口变更**必须留痕 |

### 7.3 `CODEOWNERS`（建仓时一并提交）

**必须放在每个仓库自己**的 `.github/`、根目录或 `docs/` 下。

> ⚠️ 组织 `.github` 仓库里的 `CODEOWNERS` **不生效**——GitHub 默认社区健康文件的支持列表（CODE_OF_CONDUCT / CONTRIBUTING / FUNDING / Issue 与 PR 模板 / SECURITY / SUPPORT）**不含 `CODEOWNERS`**。

> ⚠️ 被引用的 team **必须先有该仓的 Write 权限**，否则静默失效。官方明确：即使 team 成员个人已通过组织成员身份或其他 team 获得写权限，**team 本身仍须有该仓的 Write**。

### 7.4 元仓 `foray_ws` 额外需要

- `.repos`（依赖清单，pin commit）
- 顶层 `README.md`：一站式上手指南（拉取 → 构建 → 启动）
- 版本清单：当前整机 tag 包含的各仓 commit

---

## 8. 迁移路径（Strangler Fig）

**不推倒重来。** 当前 fork（`pb2025_sentry_nav`）是有价值的工作基线，逐步替换：

| 步 | 动作 | 产出 |
|---|---|---|
| **M1** | 建 `foray_interfaces`，写入最小集：裁判系统消息 + 机间态势协议 | 契约先立 |
| **M2** | 建 `foray_ws`，用 `.repos` 把上游包 pin 住（此时全是第三方） | 可复现的基线 |
| **M3** | 把当前 fork 也作为元仓里的一个 pin 项 | 基线含现状 |
| **M4** | 按依赖顺序逐个替换为自研仓：`platform` → `localization` → `navigation` → `vision` → `auto_aim` → `decision` | 每替换一个，元仓里摘掉对应上游项 |
| **M5** | 建 `foray_robots`，兵种装配根成型 | 兵种复用落地 |
| **M6** | 打第一个整机 tag | 可复现的一台车 |

**每一步可独立验证**：替换后跑通仿真 = 该步成功；不通过就回退该步，不影响其他步。

**顺序理由**：先建 `interfaces` 因为它零依赖、且所有后续仓都要依赖它；先替换 `platform`/`localization` 因为它们在最底层，上层还不用动。

---

## 9. 验收标准

| # | 标准 | 检验方式 |
|---|---|---|
| 1 | 每个仓能独立 build + 独立 test | 不依赖其他自研仓的**未发布改动** |
| 2 | 两个组可在同一天各自 merge 而互不阻塞 | 观察一周的 PR 冲突率 |
| 3 | 接口仓变更频率 ≪ 实现仓 | 统计季度提交数比值 |
| 4 | 新兵种接入只需新增装配根 + 参数 | 触碰 L1–L5 的行数应接近 0 |
| 5 | 任一整机 tag 可完整复现一台能跑的车 | 在一台干净机器上按 tag 复现 |
| 6 | 零反向依赖 | CI 层级校验通过 |

> 标准 4 是**复用程度的量化指标**：记录「新增兵种改动行数 / 总行数」。

---

## 10. 落地状态

### 仓库

**§2 的 9 个仓已全部建立**，每个仓已含：

`README.md` · `AGENTS.md` · `plan.md` · `tree.md` · `decision.md` · `CODEOWNERS` · `.foray-layer` · `.gitignore` · `.github/workflows/ci.yml`

| 仓 | 附加 |
|---|---|
| `foray_robots` | `.gitattributes`（Git LFS，场地资产） |
| `foray_ws` | `.repos` · `VERSIONS.md` |

### 组织工程

| 项 | 状态 |
|---|---|
| 组织默认文件（`.github`） | CONTRIBUTING 多仓协作 + CI 模板（含 ROS 2 作业，**空仓自动跳过**） |
| team 授权 | 算法组 → 9 个自研仓 + `.github`/`foray_docs`；电控组 → `ControllerCode` + `.github`/`foray_docs` |
| 分支保护 | 9 个自研仓的 `main` + `dev`：PR + 1 approve · 禁强推 · 禁删除 |
| 上游 fork 仓 | 轻量保护（仅禁强推/删除，**不要求 PR**——避免干扰 Sync fork） |
| **规则真空** | `WebServer`——私有仓 + GitHub Free 不支持分支保护 |

> ⚠️ 当前 `enforce_admins = false`，owner 可绕过保护规则。这是给刚起步的团队留的逃生口，
> 流程跑顺后应改为 `true`。

### 尚未开始

- **M1**：`foray_interfaces` 最小契约集（裁判系统消息 + 机间态势协议）
- **三份优先冻结契约**：世界模型接口 · 决策产物接口 · 机间态势协议
- **区域划分设计准则**：粒度档数 · 边界 · 与导航 2D 栅格的对应
- `foray_navigation` 的 `dev-rz` 遗留分支退役
