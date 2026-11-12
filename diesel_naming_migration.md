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

### 1.5 ROS 运行时 namespace（待决，见 §6）

当前实例根为 `sentry_root` / `infantry_root`。是否统一挂到 `/diesel/<robot>` 需要与
多机联调脚本（`accept.py`）、nav2 参数、launch 一次性协调，不并入前四批。

---

## 2. 迁移批次

每批一个 merge window：**中间态不可构建的改动必须成组合并**。

| 批次 | 内容 | 验收 |
|---|---|---|
| **P0**（本次） | 本计划入库；`tree.md`/`README.md` 索引更新 | 文档可读，索引可达 |
| **P1** | `repository_structure.md` §2/§4.1/§6.6 与 `.github/profile/README.md` 主页表格改指 `diesel_*` | 规范与主页无 `foray_` 系统残留、无坏链 |
| **P2** 试点 | `foray_localization` 单仓全链改造（5 包 + namespace + include + `.diesel-layer`），**仓名暂不改** | `diesel_ws` 全量构建 38 包 0 failed；四道门禁全绿；`navigation_sim` + `accept.py ready` 回执 |
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
3. **命名一致性**（建议新增 `scripts/ci/check_naming.py`）：仓内所有 `package.xml` 的 `<name>` 以 `diesel_` 开头；
   每个仓根存在 `.diesel-layer`；C++ 源文件命名空间为 `diesel::`。
4. **残留扫描**：`grep -rIl "foray" .` 的命中必须落在白名单——`decision.md` 历史条目、
   `README.md` 战队名段落、`*-foray-fork` 引用、`ZQU-Foray` 组织链接。
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

| # | 问题 | 现状倾向 |
|---|---|---|
| 1 | `foray_web`（战队门户 + STM32 调试台）是否纳入 Diesel | **不纳入**：非 ROS 上位机栈 |
| 2 | ROS 运行时 namespace 是否统一为 `/diesel/<robot>` | 待定：需与 `accept.py`、nav2、多机联调同批 |
| 3 | `foray_docs` 是否改名 `diesel_docs` | **不纳入**：战队级文档 |
| 4 | 采用「改名」还是「新建 + archive 旧仓」 | 改名（保留 issue/PR 历史与 redirect） |
| 5 | 旧前缀是否需要一段废弃期（双包共存） | 不需要：仓内无外部消费者 |

---

## 7. 参考

- [`repository_structure.md`](repository_structure.md) §2 起步期 10 仓、§4.1 fork 命名、§6.6 scope 词表
- [`algorithm_structure.md`](algorithm_structure.md) §1 分层骨架（L1–L5 / X2 / R / T）
- [discussion #6](https://github.com/orgs/ZQU-Foray/discussions/6) 命名投票
