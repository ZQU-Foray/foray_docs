# 目录结构说明

```
foray_docs/
├── README.md                           总索引与阅读顺序
├── tree.md                             本文件：目录结构说明
├── architecture/                       上位机算法架构
│   ├── architecture_diagram.md         算法结构设计图（主图 + 3 张补充视图 + 速查）
│   ├── upper_computer_architecture.md  架构决策记录（为什么这么设计）
│   └── decision_layer_plan.md          决策层接口规范与执行计划（具体怎么做）
├── engineering/                        工程与协作
│   └── repo_decomposition_plan.md      仓库解耦切割规划
└── org/                                组织规范材料（回馈 ZQU-Foray/.github）
    ├── README.md                       提交指引与当前状态
    ├── CONTRIBUTING.addendum.md        CONTRIBUTING.md 的增补内容
    ├── ci-template.yml                 新版 CI 模板（含 ROS 2 构建作业）
    ├── PR_BODY.md                      PR 描述
    └── ROLLOUT.md                      组织规范落地清单
```

---

## 按问题找文档

| 你想知道 | 看哪份 |
|---|---|
| 整体架构长什么样？ | `architecture/architecture_diagram.md` |
| 为什么要这样分层？ | `architecture/upper_computer_architecture.md` |
| 决策层接口怎么定？ | `architecture/decision_layer_plan.md` |
| 仓库怎么切、怎么协作？ | `engineering/repo_decomposition_plan.md` |
| 组织规范要改什么？ | `org/CONTRIBUTING.addendum.md` |
| 规范怎么才能真正落地？ | `org/ROLLOUT.md` |

---

## 文档关系

```
architecture_diagram.md          ← 入口：一张图看懂结构（宣讲首选）
        │
        ├── 论证依据 ──► upper_computer_architecture.md
        │                （分层理由、3v3 裁剪、三项决议、RL 评估、
        │                  开火授权安全语义、撤销记录）
        │
        └── 落地细节 ──► decision_layer_plan.md
                         （接口字段清单、对局仿真建模、红队流程、
                           数据采集、赛后闭环、序时计划）

repository 层面 ────────► engineering/repo_decomposition_plan.md
                         （切割判据、起步 9 仓、版本锁定、组织规范对齐、
                           迁移六步、验收标准、决议记录 D1–D6）

组织规范层面 ──────────► org/
                         （CONTRIBUTING 增补、CI 模板、PR 描述、落地清单）
```
