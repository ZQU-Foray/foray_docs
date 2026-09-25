# 仓库解耦切割与协作开发规划

> 编者：RyanZzzz
> 目标：把整套上位机软件切割为可并行协作的多个仓库
> 配套：`architecture_diagram.md`（结构）· `upper_computer_architecture.md`（决策）· `decision_layer_plan.md`（执行）
> 工程规范：遵循 `project-bootstrap-workflow`（Conventional Commits · 五件套文档 · 知识索引）

---

## 0. 切割判据（用判据，不用直觉）

多仓还是单仓、切多细，必须由**可复述的判据**决定，否则每次讨论都从零吵起。

| # | 判据 | 含义 | 违反的后果 |
|---|---|---|---|
| **C1** | **接口 vs 实现** | 接口必须独立成仓，实现不得把接口私有化 | 跨队/跨兵种无法复用；接口变更引发全仓重编译 |
| **C2** | **变更率（change rate）** | 变更率差异大的模块不共仓 | 高频仓的提交历史淹没稳定仓；review 负担错配 |
| **C3** | **团队边界（Conway）** | **一个仓 = 一个 owner 小组** | 两人同时改一个仓 → 频繁冲突、互相阻塞 |
| **C4** | **依赖重量** | 依赖重的（TensorRT/OpenCV vs PCL/Ceres）分开 | 所有人被迫安装全套依赖；CI 时间爆炸 |
| **C5** | **复用范围** | 被多个兵种复用的下沉；只服务单兵种的上浮 | 复用单元被兵种细节污染 |
| **C6** | **跨队共享需求** | 需共享给步兵/其他队的必须独立且稳定 | 权限无法细分；外部依赖你的私有结构 |

**粒度结论**：
> **一个仓 = 一个团队角色 + 一个变更率档位**；仓内可以包含多个 ROS2 package。

**反模式（不要做）**：
- ❌ 每个 ROS2 package 建一个仓 → 版本矩阵地狱
- ❌ 把测试、launch、config 拆出独立仓 → 它们属于装配根
- ❌ 为了"看起来整齐"按分层机械切仓 → 会切断团队边界

---

## 1. 目标仓拓扑

```
                    ┌──────────────────────────────────────────────┐
                    │  foray_ws  （元仓 · 零业务代码）              │
                    │  .repos 锁定 · 集成 CI · 一键拉取/构建/启动   │
                    │  ★ 整机 tag = 可复现的"那一台能跑的车"        │
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

**依赖方向严格自上而下**：`foray_interfaces` 是最底层，**不依赖任何自研仓**。

---

## 2. 起步期：9 仓（不要一次切太细）

一次性切 14 个仓会杀死推进力。起步期先立骨架，成熟后再拆。

| # | 仓库 | 对应层/面 | 内容 | Owner | 变更率 |
|---|---|---|---|---|---|
| 1 | `foray_interfaces` | **X2 契约** | 全部 msg/srv/action：机内接口 + **机间态势协议** + 裁判系统消息 | 架构 | **极低** |
| 2 | `foray_platform` | **L1 + L2-HAL** | 时间时钟 · 数学/滤波 · 参数系统 · 日志录制 · 错误码 + 下行 HAL（底盘/云台/发射/串口/CAN/传感器封装） | 平台 | 低 |
| 3 | `foray_localization` | **L3** | 定位服务（内置地图 + 激光配准）· 地图管理 · 配准残差动态物体检测 | 导航 | 高 |
| 4 | `foray_vision` | **L3** | 目标感知（装甲板检测/跟踪）· **态势融合** · WorldSnapshot 生成 | 视觉 | 高 |
| 5 | `foray_auto_aim` | **L4** | 自瞄（**哨兵/步兵/英雄共用**）· 多源目标注入 | 视觉 | 高 |
| 6 | `foray_navigation` | **L4** | 静态地图规划 · 动态障碍规避 · 全向控制器 | 导航 | 高 |
| 7 | `foray_decision` | **L5 + L2横向 + X5** | 战术层(规则/RL) · 技能层 · 安全门 · 机间协同 · 对局仿真（临时） | 决策 | 高 |
| 8 | `foray_robots` | **R + 地图资产** | 兵种装配根（`sentry_bringup` / `infantry_bringup`）· 参数 · 场地资产（LFS） | 集成 | 中 |
| 9 | `foray_ws` | **元仓** | `.repos` · 顶层文档 · 集成 CI · 一键脚本。**零业务代码** | 架构 | **极低** |

### 2.1 三处需要说明的取舍

**① 把 L1 与 L2-HAL 合并为 `foray_platform`**
两者都是"被所有人依赖的底层"、变更率都低、owner 都是对接电控的平台组。合并减少一个仓，符合 C2/C3。

**② 把地图资产先放进 `foray_robots`（用 Git LFS）**
地图随赛季变、且是大文件（PCD 可达数十 MB）。起步期放装配根仓 + LFS 足够；成熟后独立成 `foray_field`。

**③ `foray_decision` 初期临时容纳对局仿真**
对局仿真是离线工具、变更率高，长期应独立。起步期放在决策仓内（同一 owner），成熟后拆出 `foray_battle_sim`。

### 2.2 命名修正建议（重要）

现有仓名 `foray_sentry_nav` 的语义是"**兵种_功能**"，但它隐含"哨兵专用"——
而**导航恰恰是要复用给步兵的**。这与复用目标直接冲突。

> **建议改为按领域命名**：`foray_navigation`（领域），兵种差异移到 `foray_robots/sentry_bringup`。
> 判据 C5：复用单元不得被兵种细节污染。

---

## 3. 成熟期：+5 仓（按需拆，不预先拆）

| # | 新增仓 | 从哪拆出 | 触发条件 |
|---|---|---|---|
| 10 | `foray_field` | `foray_robots` 的场地资产 | 地图资产体积/版本频率成为负担 |
| 11 | `foray_comms` | `foray_decision` 的机间协同 | 机间协议迭代频繁，或需开放给其他队 |
| 12 | `foray_battle_sim` | `foray_decision` 的对局仿真 | 红队 RL 正式立项 |
| 13 | `foray_calibration` | `foray_platform` / 各仓零散标定脚本 | 标定需要现场独立运行 |
| 14 | `foray_debug` | 各仓的调试装配根 | 调试根数量膨胀 |

> **原则：拆仓是"因为痛了才拆"，不是"因为整齐才拆"。** 每个新仓都必须能回答
> "不拆会怎样"——答不上来就不拆。

---

## 4. 第三方依赖：fork 与 pin

上游包**不是我们的仓**，但必须纳入元仓统一锁定。

### 4.1 命名规范（沿用组织既有约定）

组织已有既定约定：**`<upstream>-foray-fork`**

| 组织现有 | 我们需要的 |
|---|---|
| `open_vins-foray-fork` | `point_lio-foray-fork` |
| `VINS-Fusion-foray-fork` | `livox_ros_driver2-foray-fork` |
| `ORB_SLAM3-foray-fork` | `small_gicp-foray-fork` |
| `RoboRTS-foray-fork` | `pb_omni_pid_pursuit_controller-foray-fork` |
| | `pb_nav2_plugins-foray-fork` |

> 好处：一眼可辨「这是上游代码，不是我们的」，并与组织既有仓库列表风格一致。

### 4.2 处置表

| 类别 | 例子 | 处置 |
|---|---|---|
| 传感器驱动 | `livox_ros_driver2`、`hik_camera_ros2_driver` | fork 到组织下，pin commit |
| 状态估计 | `point_lio`、`small_gicp` | fork，pin 到**指定分支的 commit**（注意 pb2025 用的是特制分支） |
| 规划控制插件 | `pb_omni_pid_pursuit_controller`、`pb_nav2_plugins` | fork，pin commit |
| ROS 官方 | `nav2_*`、`slam_toolbox`（仅离线建图） | 用 apt 或源码 pin 版本 |

**硬规则**：
- **仓库名必须是 `<upstream>-foray-fork`**
- **不得依赖分支**（`main`/`dev` 会漂移）——必须 pin commit 或 tag
- **不修改上游代码**；必须改则 fork + 在 `decision.md` 记录 patch 内容与原因，并尽量上游化
- 任何 fork 都要在元仓 `.repos` 里可见，不允许"某台电脑上手动 clone 的版本"

---

## 5. 版本与整机锁定机制（最关键）

### 5.1 为什么需要"整机 tag"

学生队最容易崩的场景：赛前一周发现"上周能跑的车现在跑不起来"，
没人知道是哪个仓动了。**多仓必须有一个统一的版本锚点。**

### 5.2 三层版本机制

| 层 | 机制 | 说明 |
|---|---|---|
| **仓级** | 语义化版本 + tag | 接口仓尤其严格：`v1.3.0` |
| **引用级** | 元仓 `.repos` **pin commit** | 不用分支，不用"大概最新" |
| **整机级** | **元仓 meta tag** | 如 `sentry-2026-regional-v1.0.0`，内容 = `.repos` 快照 + 版本清单 |

**规则**：
- 实现仓依赖接口仓时，**依赖 tag 而非分支**
- 每次比赛/里程碑打一个**整机 tag**
- 排障第一步：**确认场上跑的是哪个整机 tag**

### 5.3 用 vcstool 而非 git submodule

| 方案 | 评价 |
|---|---|
| `git submodule` | 原生，但操作繁琐、易出错（子模块未初始化是新人第一大坑） |
| **`vcstool` + `.repos`** ✅ | SMBU 与 RMOSS 生态均采用（`dependencies.repos`）；对新手友好；元仓一文件描述全部依赖 |

> 你 fork 的上游仓库就有 `dependencies.repos`，直接沿用同一套工具链。

---

## 6. 分支、提交与协作规范

> **本节以 [`ZQU-Foray/.github`](https://github.com/ZQU-Foray/.github) 的组织规范为准。**
> 我方架构需求只做**必要追加**（在 §11.0 逐条记录），不另立一套。

### 6.1 提交

- **Conventional Commits**：`feat` / `fix` / `docs` / `style` / `refactor` / `perf` / `test` / `chore`
- **scope 按仓库模块划分**（组织规范要求）→ **每个仓必须定义自己的 scope 词表**，见 §11.1
- 按逻辑单元**小步提交**，不得一次性提交全部代码
- **禁止破坏性操作**：`reset --hard` / `push --force` / 非必要 `rebase`

### 6.2 分支

⚠️ **组织规范原文：不要在 `main` 分支进行任何 push 和 merge 操作。**

```
main          ← 稳定版本，经过测试与 Review
  └─ dev      ← 开发分支，所有更改在此分支调试
       └─ feat/xxx · fix/xxx · docs/xxx
```

⇒ 因此：
- **所有 PR 的目标分支是 `dev`，不是 `main`**
- `main` 只承载里程碑级稳定快照（由队长/组长从 `dev` 走 PR 合入）
- 功能分支从 `dev` 切出，用 `git rebase dev` 同步上游（属于组织规范允许的必要 rebase）

> **例外（决议 D7）**：**文档类仓库**——以 Markdown / 配置为主、**不含构建产物**的仓库
> （如 `.github`、`foray_docs`）——**不需要三级分支模型**，直接在 `main` 上提交并 PR 到 `main`。
>
> 理由：三级分支的价值在于隔离「可能构建失败的开发中代码」；文档没有构建产物，
> 多一层 `dev` 只是多一次合并。

### 6.3 评审要求（按仓分级）

| 仓类型 | 组织规范基线 | **我方追加** |
|---|---|---|
| 所有仓 | ≥1 人 Review | — |
| 视觉 / 电控方向 | 建议**组长** Review | — |
| `foray_interfaces` | ≥1 人 | ⚠️ **≥2 人 + 兼容性影响评估**（它被全部仓依赖） |
| `foray_ws` 的 `.repos` 变更 | ≥1 人 | ⚠️ **必须说明换版本理由** |

> 追加理由：接口仓与元仓的改动**影响面跨仓**，组织规范的「≥1 人」是为单仓开发设计的。
> ✅ **已决议采纳**（见 §12 决议记录）。

### 6.4 每仓 CI

组织 `workflows/ci-template.yml` 已提供：`lint-python`(ruff+black) · `lint-c`(clang-format) · `test-python`(pytest)。
**直接复用，不要自拟。**

| 仓 | CI 内容 |
|---|---|
| 所有仓 | 复制组织 `ci-template.yml` → `.github/workflows/ci.yml` |
| 所有 ROS2 仓 | + **`colcon build` / `colcon test`**（组织模板缺失，见下方补丁） |
| `foray_interfaces` | + 变更影响面检查（哪些仓依赖被改动的消息） |
| `foray_ws` | **集成 CI**：按 `.repos` 拉全量 → 全量 build → 集成测试 |

⚠️ **缺口：组织模板没有 ROS2 构建任务。** 我们的仓是 colcon 工作空间，必须补：

```yaml
  # ---- ROS2 colcon 构建（组织模板缺失，需补） ----
  build-ros2:
    runs-on: ubuntu-22.04
    container: ros:humble
    steps:
      - uses: actions/checkout@v4
      - name: rosdep
        run: |
          apt-get update && rosdep update
          rosdep install -y -r --from-paths . --ignore-src --rosdistro humble
      - name: colcon build
        shell: bash
        run: |
          source /opt/ros/humble/setup.bash
          colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release
      - name: colcon test
        shell: bash
        run: |
          source /opt/ros/humble/setup.bash
          colcon test --event-handlers console_direct+ && colcon test-result --verbose
```

> **建议**：把这个补丁**回馈到组织 `.github` 仓库的 `ci-template.yml`**（改一次全组织受益），
> 而不是每个仓各自复制一份。

### 6.5 依赖方向校验（把架构变成可执行约束）

每仓在仓根声明所属层级：

```
# .foray-layer  （或写入 package.xml 的自定义字段）
layer: L3
depends_on_layers: [L1, L2]
```

CI 校验：
- `L(n)` **不得**依赖 `L(>n)`
- **不得依赖装配根仓**（`foray_robots`）
- `foray_interfaces` 不得依赖任何自研仓

> **这是"功能解耦"从文档变成事实的唯一手段。** 不做这一步，架构文档写完就烂。

### 6.6 代码风格（组织规范，直接沿用）

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

**尤其适用于本项目的两条**（组织规范已有，我们要认真执行）：

- **公共 API 必须有 Docstring / Doxygen**——我们的 ROS2 话题/服务/参数都算公共 API
- **魔法数字必须注释来源**（规则手册页码、标定值）
  → **区域划分表、各类阈值、地图原点**全部属于此类，必须注明**来源与赛季**
  （例：`# 2026 规则手册 P.12 补给区判定；2026-03 标定`）

---

## 7. 文档与仓库文件规范

### 7.1 强制基线（组织规范）

组织 `CONTRIBUTING.md` 明确：**每个仓库必须包含 `README.md`**（项目介绍、快速开始、依赖说明），必要时含 `docs/`。

⇒ **`README.md` 是所有仓的强制项，无例外。** 各仓 `README.md` 最低内容：

| 内容 | 说明 |
|---|---|
| 一句话定位 + **所属层**（L1–L5 / R / 元仓） | 让新人知道它在架构里的位置 |
| 快速开始 | 拉取 → 依赖安装 → 构建 → 运行 |
| 依赖说明 | 依赖哪些自研仓（含版本）与哪些第三方 |
| 接口说明 | 本仓对外暴露的话题 / 服务 / 参数 |
| 上下游关系 | 上游是谁、下游是谁 |

> 接口仓 `foray_interfaces` 的 `README.md` 尤其关键——
> **必须写清字段语义 / 单位 / 坐标系 / 版本兼容规则**。

### 7.2 我方追加：Agent 协作五件套（✅ 已采纳）

`project-bootstrap-workflow` 约定每仓五件套，**组织规范只强制 `README.md`**。逐项标注差异：

| 文件 | 组织规范 | 我方建议 | 在本项目的用法 |
|---|---|---|---|
| `README.md` | ✅ **强制** | ✅ 强制 | 见 §7.1 |
| `docs/` | 必要时 | ✅ 架构文档放这里 | 我们的设计文档 |
| `AGENTS.md` | 未要求 | 建议：用 AI Agent 协作的仓必备 | 统一 AI 协作方式 |
| `plan.md` | 未要求 | 建议：用于可中断恢复 | 计划、进度、Before Snapshot |
| `tree.md` | 未要求 | 建议：新人第一份读物 | 目录结构说明 |
| `decision.md` | 未要求 | **建议必备** | fork patch、换版本、**破坏性接口变更**必须留痕 |

> ⚠️ 这四项是**我方追加**，与组织规范不冲突，但增加了约束。
> **建议由你在队内确认后，决定是否回馈进组织 `.github` 的 `CONTRIBUTING.md`**——
> 若采纳，全组织新仓统一受益。
>
> ✅ **已决议采纳，并回馈组织**——拟补充进 `ZQU-Foray/.github` 的 `CONTRIBUTING.md`。
> 可直接提交的 PR 材料见 `contrib/org-github/`。

### 7.3 `CODEOWNERS`（建仓时必须一并提交）

组织已有 `CODEOWNERS` 骨架，但**视觉 / 电控 / CI 三节为空**（全局仅 `* @队长用户名`）。

⇒ 建新仓时**必须同时提交该仓的 `CODEOWNERS`**，按 §11.1 的 owner 填写，
使 Review 自动分派到对应组长，而不是全部堆给队长。

### 7.4 元仓 `foray_ws` 额外需要

- `.repos`（依赖清单，pin commit）
- 顶层 `README.md`：一站式上手指南（拉取 → 构建 → 启动）
- 版本清单：当前整机 tag 包含的各仓 commit

> ⚠️ **注意**：按你们的工作流，`readme.md` 是**唯一需求来源**。
> 当前仓库的 `readme.md` 还是空模板（只有"功能概述/文件结构/接口文档/贡献"四个空标题）。
> **在建仓之前，建议先把它填成真实需求**——否则后续 Agent 协作会"无据可依"。
> 我们这几轮产出的三份设计文档正好可以作为填写素材。

> ⚠️ **两份规范的差异需要你裁决**：组织规范把 `README.md` 列为**每个仓库必须包含**的文件，
> 但未声明它是唯一需求来源；而 `project-bootstrap-workflow` 把 `readme.md` 当作**唯一需求来源**。
> **若采纳后者**，则接口仓的 `README.md` 就是字段契约的**唯一权威文本**，
> 必须与 `msg/srv` 定义同步更新——否则会出现「文档说 A、代码做 B」。

---

## 8. 迁移路径：从当前 fork 过渡（Strangler Fig）

**不要推倒重来。** 当前 fork（`pb2025_sentry_nav`）是有价值的工作基线，用"绞杀者模式"逐步替换：

| 步 | 动作 | 产出 |
|---|---|---|
| **M1** | 建 `foray_interfaces`，写入**最小集**：裁判系统消息 + 机间态势协议 | 契约先立 |
| **M2** | 建 `foray_ws`，用 `.repos` 把**上游包 pin 住**（此时全是第三方） | 可复现的基线 |
| **M3** | 把当前 fork 也作为元仓里的一个 pin 项 | 基线含现状 |
| **M4** | 按依赖顺序逐个替换为自研仓：`platform` → `localization` → `navigation` → `vision` → `auto_aim` → `decision` | 每替换一个，元仓里把对应上游项摘掉 |
| **M5** | 建 `foray_robots`，兵种装配根成型 | 兵种复用落地 |
| **M6** | 打第一个**整机 tag** | 可复现的一台车 |

**每一步都可独立验证**：替换后跑通仿真 = 该步成功；不通过就回退该步，不影响其他步。

**顺序理由**：先建 `interfaces` 是因为它零依赖、且所有后续仓都要依赖它；
先替换 `platform`/`localization` 是因为它们在最底层，上层还不用动。

---

## 9. 切割的验收标准（可检验，不是感觉）

| # | 标准 | 检验方式 |
|---|---|---|
| 1 | **每个仓能独立 build + 独立 test** | 不依赖其他自研仓的**未发布改动** |
| 2 | **两个组可在同一天各自 merge 而互不阻塞** | 观察一周的 PR 冲突率 |
| 3 | **接口仓变更频率 ≪ 实现仓** | 统计季度提交数比值 |
| 4 | **新兵种接入只需新增装配根 + 参数** | 触碰 L1–L5 的行数应接近 0 |
| 5 | **任一整机 tag 可完整复现一台能跑的车** | 在一台干净机器上按 tag 复现 |
| 6 | **零反向依赖** | CI 层级校验通过 |

> 标准 4 是**复用程度的量化指标**：记录"新增兵种改动行数 / 总行数"。

---

## 10. 一页速查

- **判据**：接口/实现 · 变更率 · 团队边界 · 依赖重量 · 复用范围 · 跨队共享（C1–C6）
- **粒度**：一仓 = 一个团队角色 + 一个变更率档位；仓内可含多 package
- **起步 9 仓**：`interfaces` · `platform` · `localization` · `vision` · `auto_aim` · `navigation` · `decision` · `robots` · `ws`
- **成熟 +5 仓**：`field` · `comms` · `battle_sim` · `calibration` · `debug`（**痛了才拆**）
- **接口仓零依赖**，是所有仓的地基；**依赖 tag 不用分支**
- **第三方 fork 必须 pin commit**，不修改上游，改了要记 `decision.md`
- **三层版本**：仓 tag → `.repos` pin commit → **整机 meta tag**
- **元仓用 vcstool**，不用 submodule
- **依赖方向进 CI**——这是解耦从文档变事实的唯一手段
- **命名按领域**，不按兵种（`foray_navigation`，不是 `foray_sentry_nav`）
- **分支三层**：`main`（禁 push/merge）← `dev`（**PR 目标**）← `feat|fix|docs/xxx`
- **fork 命名**：`<upstream>-foray-fork`
- **每仓强制 `README.md`**（组织规范）+ **五件套**（已决议必备）
- **CI 复用组织 `ci-template.yml`**，但需补 ROS2 `colcon build` 缺口
- **建仓时一并提交 `CODEOWNERS`**，否则 Review 全堆给队长
- **迁移用绞杀者模式**，不推倒重来

---

## 11. 对齐组织规范：修正记录与每仓速查

### 11.0 我上一版提案 vs 组织规范（逐条修正）

| 项 | 我上一版提案 | **组织规范（以它为准）** | 处置 |
|---|---|---|---|
| PR 目标分支 | `feat/*` → `main` | **`feat/*` → `dev`**；main 禁任何 push/merge | ⚠️ **已修正** §6.2 |
| 分支层数 | 两层（main + feat） | **三层**（main / dev / feat） | ⚠️ **已修正** §6.2 |
| 每仓文档 | 五件套全强制 | 仅 `README.md` 强制 | ⚠️ 已修正：`README.md` 为强制基线；**五件套已决议采纳**为追加 §7.2 |
| fork 命名 | 未规定 | **`<upstream>-foray-fork`** | ⚠️ **已修正** §4.1 |
| 接口仓评审 | ≥2 人 | ≥1 人（视觉/电控建议组长） | ✅ 规范为基线；**≥2 人已决议采纳**为追加 §6.3 |
| CI | 自拟 | 已有 `ci-template.yml` | ⚠️ **改为复用组织模板** §6.4 |
| ROS2 构建 | 已提 | **模板缺失** | ✅ **我补上，并建议回馈组织** §6.4 |
| Commit scope | 未定义 | **按仓库模块划分** | ✅ **为每仓定义 scope 词表** §11.1 |
| 代码风格 | 未提 | 4 空格 / LF / UTF-8 / ruff / clang-format | ✅ **直接沿用** §6.6 |
| CODEOWNERS | 未提 | 已有骨架但三节为空 | ✅ **建仓时一并提交** §7.3 |
| 依赖方向校验 | 已提 | 规范未覆盖 | ✅ **已决议采纳** §6.5 |
| 整机 tag | 已提 | 规范未覆盖 | ✅ **已决议采纳** §5 |

### 11.1 每仓规范速查表

**所有仓共同**：PR → **`dev`** · 4 空格 / LF / UTF-8 · `README.md` 强制 · CI 复用组织 `ci-template.yml`（+ colcon job）

| 仓 | 层 | Owner | Commit scope 词表 | CI 额外 |
|---|---|---|---|---|
| `foray_interfaces` | X2 | 架构组长 | `msg` `srv` `action` `referee` `comms` `map` | 兼容性影响检查 |
| `foray_platform` | L1+L2 | 平台组长 | `time` `math` `param` `log` `hal` `chassis` `gimbal` `shooter` `serial` `can` | — |
| `foray_localization` | L3 | 导航组长 | `map` `relocalize` `icp` `residual` `tf` | — |
| `foray_vision` | L3 | 视觉组长 | `detector` `classifier` `preprocess` `tracker` `fusion` `snapshot` | — |
| `foray_auto_aim` | L4 | 视觉组长 | `detector` `tracker` `solver` `aimer` `shooter` `target` | — |
| `foray_navigation` | L4 | 导航组长 | `planner` `controller` `costmap` `recovery` | — |
| `foray_decision` | L5 | 决策组长 | `tactic` `skill` `safety` `gate` `comms` `rl` `sim` | — |
| `foray_robots` | R | 集成组长 | `sentry` `infantry` `params` `launch` `field` | — |
| `foray_ws` | 元 | 架构组长 | `deps` `ci` `docs` `scripts` | **集成 CI** |
| `<upstream>-foray-fork` | 第三方 | 架构组长 | 不提交（只 pin） | — |

> scope 词表对齐组织规范示例（`detector` / `classifier` / `chassis` / `ci` 等），并补齐我们的领域词。

### 11.2 建新仓 Checklist

- [ ] 仓名符合 §11.1（领域命名；fork 用 `<upstream>-foray-fork`）
- [ ] **代码仓**：创建 `dev` 分支并**设为默认分支**（保证 PR 落到 dev）
- [ ] **文档仓**（无构建产物，如 `foray_docs`）：**跳过 `dev`**，直接用 `main`（决议 D7）
- [ ] 保护 `main` 与 `dev`（禁直推）
- [ ] 提交 `README.md`（层归属 · 快速开始 · 依赖 · 接口 · 上下游）
- [ ] 提交 `CODEOWNERS`（填本仓 owner，不要全堆给队长）
- [ ] 提交 `.clang-format` / `pyproject.toml` / `.editorconfig`（4 空格 / LF / UTF-8）
- [ ] 复制组织 `ci-template.yml` → `.github/workflows/ci.yml`，**补 §6.4 的 colcon job**
- [ ] 提交 `.foray-layer`（层级声明，供 §6.5 依赖方向校验）
- [ ] 在 `foray_ws` 的 `.repos` 里登记（pin commit）
- [ ] 按 §7.2 加五件套（**已决议必备**）：`AGENTS.md` / `plan.md` / `tree.md` / `decision.md`

---

## 12. 决议记录

| # | 决议 | 结论 | 适用范围 |
|---|---|---|---|
| **D1** | 每仓五件套：`AGENTS.md` / `plan.md` / `tree.md` / `decision.md` | ✅ **采纳** | 所有自研仓 |
| **D2** | 跨仓库契约类改动评审 **≥2 人 + 兼容性影响评估** | ✅ **采纳** | `foray_interfaces` |
| **D3** | 依赖清单（`.repos`）版本变更须说明理由 | ✅ **采纳** | `foray_ws` |
| **D4** | 依赖方向校验进 CI（`.foray-layer`） | ✅ **采纳** | 所有自研仓 |
| **D5** | 整机 meta tag 版本锁定 | ✅ **采纳** | 元仓 + 全部子仓 |
| **D6** | 文件名统一大写 `README.md` | ✅ **采纳** | 所有仓 |
| **D7** | **文档类仓库免三级分支模型**（Markdown / 配置为主、无构建产物） | ✅ **采纳** | `.github`、`foray_docs` 等 |

**D1–D3 需同步补充进组织 `CONTRIBUTING.md`**；D4/D5 是本项目追加，不进组织规范。

可直接提交的 PR 材料：`contrib/org-github/`
（`README.md` · `CONTRIBUTING.addendum.md` · `ci-template.yml` · `PR_BODY.md` · `ROLLOUT.md`）

---

## 13. ⚠️ 现状核实：组织规范尚未落地

在准备回馈材料时实测了组织各仓库，发现一个比「再写规范」更重要的问题：

| 仓库 | 现有分支 |
|---|---|
| `.github` | `main` |
| `ControllerCode` | `main` |
| `foray_sentry_nav` | `dev-rz` `fork` |
| `WebServer` | `main` |
| `Foray-HelloWorld` | `linnan/ros-check` `main` |

**没有任何仓库存在 `dev` 分支**，而组织规范要求所有 PR 落到 `dev`。

⇒ 三条推论：

1. **规范质量不缺，缺落地。** `CONTRIBUTING.md` 写得比多数学生战队好，
   但三层分支模型一处都没实现——所以「第一个 PR 就绕过去了」。
2. **光靠文档拦不住违反。** 「不要在 main 分支 push/merge」必须由
   **分支保护规则**强制执行：文档负责「让人知道」，保护规则负责「让人做不到」。
3. **对本文档的影响**：§6.1 写的「PR → `dev`」在规范层面正确，但要真正生效，
   **前置动作是先给各仓建 `dev` + 开分支保护**。
4. **例外**：`.github` 与 `foray_docs` 属**文档类仓库**（无构建产物），
   按决议 D7 **直接用 `main`、不需要 `dev`**——本次 `.github` 的 PR 目标为 `main`
   **符合规范，不是绕过**。

**落地清单见 `contrib/org-github/ROLLOUT.md`**（含执行顺序与每步命令）。

> **判断落地成功的唯一标准**：下次有人想图省事直接 push `main` 时，
> 他**做不到**——而不是「记得规范里说过不行」。
