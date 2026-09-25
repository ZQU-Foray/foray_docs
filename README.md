# Foray 技术文档

> 肇庆学院 Foray 战队 · RoboMaster
> 远端仓库：[`ZQU-Foray/foray_docs`](https://github.com/ZQU-Foray/foray_docs)

---

## 文档

| 文档 | 内容 |
|---|---|
| [algorithm_structure.md](algorithm_structure.md) | **算法结构** —— 分层骨架、世界模型、决策层接口、离线开发闭环 |
| [repository_structure.md](repository_structure.md) | **仓库结构** —— 仓拓扑、切割判据、依赖方向、版本锁定、协作规范、迁移路径 |

**阅读顺序**：先 `algorithm_structure.md` 建立整体结构，再看 `repository_structure.md` 了解如何分工落地。

---

## 相关仓库

| 仓库 | 说明 |
|---|---|
| [`ZQU-Foray/.github`](https://github.com/ZQU-Foray/.github) | 组织默认文件：贡献指南、Issue / PR 模板、CI 模板 |
| [`ControllerCode`](https://github.com/ZQU-Foray/ControllerCode) | 电控代码：MC02 (H723) 与官方 C 板 (F407) |

**上位机自研仓**（结构见 [`repository_structure.md`](repository_structure.md) §2）：

[`foray_interfaces`](https://github.com/ZQU-Foray/foray_interfaces) ·
[`foray_platform`](https://github.com/ZQU-Foray/foray_platform) ·
[`foray_localization`](https://github.com/ZQU-Foray/foray_localization) ·
[`foray_vision`](https://github.com/ZQU-Foray/foray_vision) ·
[`foray_auto_aim`](https://github.com/ZQU-Foray/foray_auto_aim) ·
[`foray_navigation`](https://github.com/ZQU-Foray/foray_navigation) ·
[`foray_decision`](https://github.com/ZQU-Foray/foray_decision) ·
[`foray_robots`](https://github.com/ZQU-Foray/foray_robots) ·
[`foray_ws`](https://github.com/ZQU-Foray/foray_ws)

---

## 贡献

遵循组织[贡献指南](https://github.com/ZQU-Foray/.github/blob/main/CONTRIBUTING.md)。

> 本仓属于**文档类仓库**（以 Markdown / 配置为主、不含构建产物），
> 按组织规范**不需要三级分支模型**——直接在 `main` 上提交、直接 PR 到 `main`。
