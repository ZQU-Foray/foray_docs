# Foray 技术文档

> 肇庆学院 Foray 战队 · RoboMaster
> 远端仓库：[`ZQU-Foray/foray_docs`](https://github.com/ZQU-Foray/foray_docs)
> 本仓集中存放**架构设计**与**工程协作**文档。

---

## 📂 目录

### `architecture/` — 上位机算法架构

| 文档 | 内容 | 适合谁读 |
|---|---|---|
| [architecture_diagram.md](architecture/architecture_diagram.md) | **算法结构设计图**：主图 + 3 张补充视图 + 23 条要点速查 | 所有人 / 宣讲首选 |
| [upper_computer_architecture.md](architecture/upper_computer_architecture.md) | **架构决策记录**：分层论证、3v3 裁剪、三项决议、RL 评估、开火授权安全语义 | 算法组 |
| [decision_layer_plan.md](architecture/decision_layer_plan.md) | **决策层接口规范与执行计划**：字段清单、对局仿真建模、红队 RL 流程、数据采集 | 决策 / 导航 / 视觉 |

**建议阅读顺序**：先看 `architecture_diagram.md` 的**主图**建立整体印象；
需要论证依据时再查 `upper_computer_architecture.md`。

### `engineering/` — 工程与协作

| 文档 | 内容 |
|---|---|
| [repo_decomposition_plan.md](engineering/repo_decomposition_plan.md) | **仓库解耦切割规划**：切割判据、起步 9 仓拓扑、版本锁定机制、组织规范对齐、迁移路径 |

### `org/` — 组织规范材料

回馈 [`ZQU-Foray/.github`](https://github.com/ZQU-Foray/.github) 的改动材料（分支已推送，待合并）。

| 文件 | 内容 |
|---|---|
| [org/README.md](org/README.md) | 提交指引与当前状态 |
| [org/CONTRIBUTING.addendum.md](org/CONTRIBUTING.addendum.md) | `CONTRIBUTING.md` 的增补内容 |
| [org/ci-template.yml](org/ci-template.yml) | 新版 CI 模板（含 ROS 2 构建作业） |
| [org/PR_BODY.md](org/PR_BODY.md) | 可直接粘贴的 PR 描述 |
| [org/ROLLOUT.md](org/ROLLOUT.md) | ⭐ 组织规范**落地清单**（为什么规范没被执行、怎么让它生效） |

---

## 🔗 相关仓库

| 仓库 | 说明 |
|---|---|
| [`.github`](https://github.com/ZQU-Foray/.github) | 组织默认文件：贡献指南、Issue / PR 模板、CI 模板 |
| [`foray_sentry_nav`](https://github.com/ZQU-Foray/foray_sentry_nav) | 哨兵导航栈（ROS 2 / Nav2） |
| [`ControllerCode`](https://github.com/ZQU-Foray/ControllerCode) | 电控代码：MC02 (H723) 与官方 C 板 (F407) |

---

## 📌 尚未归档的素材

以下材料目前不在本仓，待确认后归档：

| 材料 | 当前位置 | 备注 |
|---|---|---|
| `27赛季视觉算法组备战调研报告.md` | `/home/ryanz/eDisk/RoboMaster/` | 视觉方向调研 |
| `27赛季视觉规划` | `/home/ryanz/eDisk/RoboMaster/` | 视觉方向规划 |
| `nav_stack_architecture_design.md` | `foray_sentry_nav/docs/` | 已有归属，未重复归档 |
| `foray_sentry_nav/docs/knowledge/` | `foray_sentry_nav/docs/` | 知识索引，随该仓维护 |

---

## 🤝 贡献

遵循组织[贡献指南](https://github.com/ZQU-Foray/.github/blob/main/CONTRIBUTING.md)。

> ✅ **本仓属于「文档类仓库」**（以 Markdown / 配置为主、**不含构建产物**），
> 按组织规范**不需要三级分支模型**——直接在 `main` 上提交、直接 PR 到 `main` 即可。
