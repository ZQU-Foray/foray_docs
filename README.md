# Foray 技术文档

> 肇庆学院 Foray 战队 · RoboMaster
> 远端仓库：[`ZQU-Foray/foray_docs`](https://github.com/ZQU-Foray/foray_docs)

---

## 文档

| 文档 | 内容 |
|---|---|
| [algorithm_structure.md](algorithm_structure.md) | **算法结构** —— 分层骨架、世界模型、决策层接口、离线开发闭环 |
| [repository_structure.md](repository_structure.md) | **仓库结构** —— 仓拓扑、切割判据、依赖方向、版本锁定、协作规范、迁移路径 |
| [dual_stream_disentanglement.md](dual_stream_disentanglement.md) | **双流特征解耦** —— 灯条颜色 × 图案的任务解耦、非对称双流结构、训练策略与消融方案 |
| [diesel_naming_migration.md](diesel_naming_migration.md) | **命名迁移** —— 战队名 `Foray` 与系统名 `Diesel` 的边界、仓名 / ROS 包名 / C++ namespace 改名批次与验收 |

**阅读顺序**：先 `algorithm_structure.md` 建立整体结构，再看 `repository_structure.md` 了解如何分工落地；视觉识别方向直接看 `dual_stream_disentanglement.md`。

---

## 相关仓库

| 仓库 | 说明 |
|---|---|
| [`ZQU-Foray/.github`](https://github.com/ZQU-Foray/.github) | 组织默认文件：贡献指南、Issue / PR 模板、CI 模板 |
| [`ControllerCode`](https://github.com/ZQU-Foray/ControllerCode) | 电控代码：MC02 (H723) 与官方 C 板 (F407) |

**上位机自研仓**（结构见 [`repository_structure.md`](repository_structure.md) §2）：

[`foray_interfaces`](https://github.com/ZQU-Foray/diesel_interfaces) ·
[`foray_platform`](https://github.com/ZQU-Foray/diesel_platform) ·
[`foray_localization`](https://github.com/ZQU-Foray/diesel_localization) ·
[`foray_vision`](https://github.com/ZQU-Foray/diesel_vision) ·
[`foray_auto_aim`](https://github.com/ZQU-Foray/diesel_auto_aim) ·
[`foray_navigation`](https://github.com/ZQU-Foray/diesel_navigation) ·
[`foray_decision`](https://github.com/ZQU-Foray/diesel_decision) ·
[`foray_robots`](https://github.com/ZQU-Foray/diesel_robots) ·
[`foray_ws`](https://github.com/ZQU-Foray/diesel_ws)

> 上面的仓名仍是 `foray_*`：Diesel 迁移按 [`diesel_naming_migration.md`](diesel_naming_migration.md)
> 分批推进，P4 会把远程仓名改成 `diesel_*`，本段链接由脚本统一替换（旧 URL 会重定向）。
> 目标态的仓名与职责见 [`repository_structure.md`](repository_structure.md) §2。

---

## 贡献

遵循组织[贡献指南](https://github.com/ZQU-Foray/.github/blob/main/CONTRIBUTING.md)。

> 本仓属于**文档类仓库**（以 Markdown / 配置为主、不含构建产物），
> 按组织规范**不需要三级分支模型**——直接在 `main` 上提交、直接 PR 到 `main`。
