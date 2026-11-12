# Diesel 命名迁移计划

> **状态**：`P0 已入库` · 待逐批执行
> **依据**：[`ZQU-Foray/.github` discussion #6](https://github.com/orgs/ZQU-Foray/discussions/6) ——
> 投票问「是否将 Diesel 作为 Foray 战队上位机软件控制系统命名，并统一 namespace」，
> 结果 **是 2 票 / 否 0 票**（2026-10-29 发起）。
> **范围**：上位机软件系统（L1–L5 + 工具层）的**仓名前缀**、**ROS 包名**、**C++ 命名空间**与**层声明文件**。

---

## 0. 为什么改、改到哪

`foray_` 这个前缀一直同时承担两种语义：

| 语义 | 例子 | 处置 |
|---|---|---|
| **战队名** | 组织 `ZQU-Foray`、队徽、fork 后缀 `<upstream>-foray-fork`、`foray_docs`、`foray_mcu`、`foray_web` | **保留**——它们是战队资产，不随系统代号走 |
| **上位机系统代号** | `foray_localization`、`foray_ws`、`foray_localization_core`、`namespace foray::localization` | **改名 Diesel** |

discussion #6 投票针对的是第二种。因此本次迁移只动「上位机软件系统」这一层，
战队名下位机（`foray_mcu`、`foray_mcu_controller`）、门户（`foray_web`）、
文档（`foray_docs`）与四个上游 fork 全部保留原名。

> `foray_web` 按**文档/门户类**处理——它是战队管理门户与 STM32 调试台，
> 不属于 ROS 2 上位机栈，与 `foray_docs` 同级，不进 Diesel 命名体系。

沿用 §2.2 的判据 C5「复用单元不得被兵种细节污染」的同一逻辑：
**命名单元不得被战队名与系统名混用污染。**

---

## 1. 目标命名

### 1.1 仓库名：`diesel_<domain>`

| # | 现名 | 新名 | 层/面 | 现状 |
|---|---|---|---|---|
| 1 | `foray_interfaces` | `diesel_interfaces` | X2 契约 | 骨架 |
| 2 | `foray_platform` | `diesel_platform` | L1 + L2-HAL | 骨架 |
| 3 | `foray_localization` | `diesel_localization` | L3 | **已实现**（5 包 / 145 测试） |
| 4 | `foray_vision` | `diesel_vision` | L3 | 骨架 |
| 5 | `foray_auto_aim` | `diesel_auto_aim` | L4 | 骨架 |
| 6 | `foray_navigation` | `diesel_navigation` | L4 | 骨架 |
| 7 | `foray_decision` | `diesel_decision` | L5 + L2 横向 + X5 | 骨架 |
| 8 | `foray_robots` | `diesel_robots` | R + 地图资产 | 骨架 |
| 9 | `foray_ml` | `diesel_ml` | T 离线工具 | 产线工具 |
| 10 | `foray_ws` | `diesel_ws` | 元仓 | **已实现**（`.repos` + 38 包可构建） |
| 11 | `foray_simulation` | `diesel_simulation` | 联调仿真 | **已实现**（107 文件） |

**不改名**：`foray_docs`、`foray_mcu`、`foray_mcu_controller`、`foray_web`、`foray-HelloWorld`、
`open_vins-foray-fork`、`VINS-Fusion-foray-fork`、`ORB_SLAM3-foray-fork`、`RoboRTS-foray-fork`、组织 `ZQU-Foray`。

### 1.2 ROS 2 包名：`diesel_<repo>_<module>`

已落地的包（现存 7 个）：

| 现包名 | 新包名 |
|---|---|
| `foray_localization_bringup` | `diesel_localization_bringup` |
| `foray_localization_core` | `diesel_localization_core` |
| `foray_localization_map` | `diesel_localization_map` |
| `foray_localization_relocalization` | `diesel_localization_relocalization` |
| `foray_localization_ros` | `diesel_localization_ros` |
| `foray_simulation_bringup` | `diesel_simulation_bringup` |
| `foray_rmul_2027_field` | `diesel_rmul_2027_field` |

骨架仓暂无包，建包时直接用新前缀。

### 1.3 C++ 命名空间与 include 路径

| 项 | 现 | 新 |
|---|---|---|
| 命名空间 | `namespace foray::localization` | `namespace diesel::localization` |
| include 路径 | `#include "foray/localization/health.hpp"` | `#include "diesel/localization/health.hpp"` |
| 安装头目录 | `include/foray/...` | `include/diesel/...` |
| CMake 目标 | `foray_localization_core` | `diesel_localization_core` |

规模（实测）：`foray_localization` 92 个文件含 `foray`、**81 处** `#include "foray/…"`、
12 处 `namespace foray::`；`foray_simulation` 107 个跟踪文件、51 个含 `foray`。
`foray::` 是本仓自有 API 前缀，改造属**破坏性接口变更**，必须在 `decision.md` 留痕。

### 1.4 层声明文件

`.foray-layer` → `.diesel-layer`（7 个仓根各一份）。文件内容 `layer:` 取值不变。

### 1.5 ROS 运行时 namespace：`/diesel/<robot>`

上位机共用一套 ROS 图，实例按兵种区分。统一规则：

| 层 | 现 | 新 |
|---|---|---|
| 实例根 | 扁平（全局同名话题） | `/diesel/sentry` · `/diesel/infantry` |
| 定位状态 | `/localization/state` | `/diesel/<robot>/localization/state` |
| 地图元信息 | `/localization/map_meta` | `/diesel/<robot>/localization/map_meta` |
| 动态障碍 | `/localization/obstacles` | `/diesel/<robot>/localization/obstacles` |
| 节点名 | `foray_localization` · `foray_map_server` | `diesel_localization` · `diesel_map_server` |

**实现方式（关键）**：节点内话题名一律改为**相对名**（`localization/state`），
由 launch / 兵种 profile 注入 `namespace:=/diesel/sentry`。不在 C++ 里拼绝对前缀，
也不靠逐条 remap 写死——否则多机与单机回放要维护两套名字。

**保留**：TF 帧名（`map` / `odom` / `base_footprint` / `body`）不进 namespace，
tf2 的帧名是全局唯一的树状标识，加前缀会破坏 `tf2_echo`、nav2 与 RViz 配置。

### 1.6 环境变量与实机路径

| 现 | 新 |
|---|---|
| `FORAY_LIVOX_SDK2_PREFIX` / `FORAY_MVS_PREFIX` / `FORAY_SKIP_PACKAGES` | `DIESEL_LIVOX_SDK2_PREFIX` / `DIESEL_MVS_PREFIX` / `DIESEL_SKIP_PACKAGES` |
| `/opt/foray/livox-sdk2` · `/opt/foray/mvs-sdk`（实机镜像路径） | `/opt/diesel/livox-sdk2` · `/opt/diesel/mvs-sdk` |

实机镜像需同批改路径；`~/.cache/foray-platform-prefix/`（开发机默认安装点）一并改
`~/.cache/diesel-platform-prefix/`。**保留**：`foray_docs` 的文档链接与战队名。

---

## 2. 迁移批次

每批一个 merge window：**中间态不可构建的改动必须成组合并**。

| 批次 | 内容 | 验收 |
|---|---|---|
| **P0**（本次） | 本计划入库；`tree.md`/`README.md` 索引更新 | 文档可读，索引可达 |
| **P1** | `repository_structure.md` §2/§4.1/§6.6 与 `.github/profile/README.md` 主页表格改指 `diesel_*` | 规范与主页无 `foray_` 系统残留、无坏链 |
| **P2** 试点 | **`diesel_ws` + `diesel_robots`** 两仓全链改造（元仓文档/脚本/`.diesel-layer`/环境变量 + 装配根骨架），仓名暂不改 | ws：构建入口仍 38 包 0 failed、`check_repos_drift.sh` exit 0；robots：四道门禁全绿 |
| **P3** 批量 | `interfaces`（契约先行）→ `platform` → `simulation` → 其余骨架仓的包名与 `.diesel-layer` | 同上，且下游 `package.xml` 依赖同步 |
| **P4** 远程改名 | `gh api -X PATCH /repos/ZQU-Foray/<old> -f name=<new>` ×11；`.repos` 自研仓 URL；本地 remote | 旧链接 302 到新仓；`scripts/check_repos_drift.sh` exit 0 |
| **P5** 收尾 | 徽章、`CODEOWNERS`、文档互链全局 grep、打 `diesel-v0.1` tag | 白名单外零 `foray` 残留 |

**为什么远程改名放 P4**：GitHub 改名会 redirect，但所有 PR 的 base、CI badge、
`.repos`、本地 remote 都会瞬时错位；把不可逆动作留到代码与文档都改完之后，只需要一次切换。

---

## 3. 破坏性与回滚

| 风险 | 影响 | 处置 |
|---|---|---|
| `diesel_interfaces` 契约改名 | 所有下游 `package.xml`、launch、消息类型名断裂 | 契约与下游同批合并；先改 interfaces 再改消费者，PR 链一次合并 |
| C++ namespace / include 改名 | 外部消费者编译失败 | 该版本无外部消费者，仍记 `decision.md` 破坏性变更 |
| GitHub 改名 | 旧 URL 重定向、外部 CI 引用失效 | 只改 11 个自研仓；`gh api PATCH` 可逆 |
| 中间态 | 单仓改一半无法构建 | 每批一个 merge window，分支保护 + 组内同合 |

**回滚**：`gh api -X PATCH` 改回原名 + `git revert` 对应 PR；redirect 残留无害。

---

## 4. 验收检查项

1. **构建**：`source /opt/ros/humble/setup.bash && cd ~/Projects/diesel_ws && scripts/colcon_build.sh` → `38 packages finished`，0 failed。
2. **门禁**：`scripts/ci/check_style.py`、`check_tree_doc.py`、`check_params.py`、`check_contracts.py` 全 exit 0。
3. **命名一致性**：仓内所有 `package.xml` 的 `<name>` 以 `diesel_` 开头；每仓根有 `.diesel-layer`；
   C++ 源文件命名空间为 `diesel::`。
4. **残留扫描**：`diesel_ws/scripts/check_naming.sh`（已落地，CI `naming` 作业执行）——白名单外不得出现
   `foray`；`grep -rIl -i foray .` 的命中只能落在白名单：`ZQU-Foray`、`foray_docs`、`*-foray-fork`、
   战队文案/版权头（`Foray`/`FORAY`）、`foray_mcu`、`foray_web`、`foray-HelloWorld`、P4 前的
   `foray_localization` 引用、已删除沙箱目录名 `foray_sim_ws` 的历史叙述。
   2026-11-12 实跑：11 个上位机仓全部通过；首跑抓到 `scripts/build_livox_sdk2.sh` 的
   `~/.cache/foray-build` 两处真残留并修掉。
5. **链接**：`foray_docs/README.md`、`.github/profile/README.md` 中 11 个仓链接全部 200。

---

## 5. 文档同步清单

| 文件 | 改动 |
|---|---|
| `repository_structure.md` §2 | 10 仓表格改 `diesel_*`，§2.2 保留历史并追加「2026-11-12 系统命名 Diesel」 |
| `repository_structure.md` §4.1 | 明确 `<upstream>-foray-fork` **不适用**新前缀（战队名归属） |
| `repository_structure.md` §6.6 | scope 词表仓名改 `diesel_*` |
| `README.md` | 「上位机自研仓」链接表改 11 个新仓 |
| `tree.md` | 新增本文件条目 |
| `.github/profile/README.md` | 「上位机（ROS 2）」表格改 11 行 |
| 各仓 `tree.md` / `.repos` / `README` 徽章 | 随批次更新 |

---

## 6. 未决问题

| # | 问题 | 结论 |
|---|---|---|
| 1 | `foray_web`（战队门户 + STM32 调试台）是否纳入 Diesel | **已决：不纳入**，归入文档/门户类，与 `foray_docs` 同级保留 |
| 2 | ROS 运行时 namespace 是否统一为 `/diesel/<robot>` | **已决：统一**，见 §1.5（相对话题名 + launch 注入 namespace，TF 帧名不动） |
| 3 | `foray_docs` 是否改名 `diesel_docs` | **不纳入**：战队级文档 |
| 4 | 采用「改名」还是「新建 + archive 旧仓」 | 改名（保留 issue/PR 历史与 redirect） |
| 5 | 旧前缀是否需要一段废弃期（双包共存） | 不需要：仓内无外部消费者 |
| 6 | 实机镜像 `/opt/foray/*` 的切换时点 | 随 P4 改名同批，由部署侧重新出镜像 |

---

## 7. 执行进度（2026-11-12）

### P2 试点（已完成，等评审）

| 仓 | 改动 | 回执 |
|---|---|---|
| `diesel_ws`（原 `foray_ws`） | `.diesel-layer`、`DIESEL_*` 环境变量、`/opt/diesel`、脚本与文档全量 | `scripts/check_repos_drift.sh` →「11 条 pin 无 commit 漂移」 |
| `diesel_robots`（原 `foray_robots`） | `.diesel-layer`、README/tree 引用 | 四道门禁全绿 |

### P1 规范（部分完成）

- `repository_structure.md` 已按目标态改写：§2 起仓名前缀为 `diesel_*`，并加迁移说明
  （哪些名字保留、P4 前远端仍是旧名、GitHub 保留重定向）；§0 拓扑图同步改名。
- 仍待 P4：`foray_docs/README.md` 的 9 条仓链接、`.github/profile/README.md` 的上位机表格
  —— 链接指向真实 URL，提前改会在改名完成前 404。

### P3 批量（11 份 PR 已开、CI 全绿，等待 review 后合入）

| 仓（新名） | PR | 备注 |
|---|---|---|
| `diesel_interfaces` | #4 | 契约先行，建议第一个合 |
| `diesel_ws` | #4 | **堆叠在 #3 `fix/m2-build-root` 之上**，先合 #3 |
| `diesel_robots` | #1 | |
| `diesel_localization` | #11 | 含 namespace 改造（相对话题名） |
| `diesel_simulation` | #8 | 含 namespace 注入 `robot_namespace` |
| `diesel_platform` | #7 | 含 polkit/NM profile 等对外标识改名 |
| `diesel_navigation` | #2 | 含头文件目录 `include/diesel/nav/` |
| `diesel_auto_aim` | #7 | |
| `diesel_decision` | #1 | |
| `diesel_vision` | #1 | |
| `diesel_ml` | #2 | |

验证口径：改名后 `diesel_ws` 全量构建 38 包 0 failed；`diesel_localization` 四道仓级门禁
（file-hygiene / doc-sync / param-contract / contract-lock）exit 0；`diesel_simulation`
整链 `accept.py ready` → `ready=PASS last_state=OK state_reason="warmup complete"`，
话题出现在 `/diesel/sentry/localization/{state,map_meta,dynamic_obstacles}`。

已知非回归：`test_localization_node` 的 `SlowAlignmentKeepsHealthOk` 在开发机（高负载、
wall clock 与模拟时钟不同源）会红，同失败在 `origin/dev` 基线上可复现，CI 机器上为绿。

### P4 远程改名（未开始）

按 plan 的放行条件，等 P3 各 PR 合入 `dev` 后再执行；执行前需要一次显式确认。

---

## 8. 参考

- [`diesel_naming_migration.md`](diesel_naming_migration.md) 本文 §7 执行进度
- [`repository_structure.md`](repository_structure.md) §2 起步期 10 仓、§4.1 fork 命名、§6.6 scope 词表
- [`algorithm_structure.md`](algorithm_structure.md) §1 分层骨架（L1–L5 / X2 / R / T）
- [discussion #6](https://github.com/orgs/ZQU-Foray/discussions/6) 命名投票
