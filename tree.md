# 目录结构说明

```
foray_docs/
├── README.md                    总索引
├── tree.md                      本文件：目录结构说明
├── algorithm_structure.md       算法结构
├── repository_structure.md      仓库结构
└── dual_stream_disentanglement.md  双流特征解耦（视觉识别专题调研）
```

---

## 按问题找文档

| 你想知道 | 看哪份 | 章节 |
|---|---|---|
| 整体分几层、各层职责 | `algorithm_structure.md` | §1 分层骨架 |
| 世界模型怎么形成 | `algorithm_structure.md` | §2 世界模型 |
| 决策层接口怎么定 | `algorithm_structure.md` | §3 决策层 |
| RL 放在哪个位置 | `algorithm_structure.md` | §4 离线开发闭环 |
| 三份必须优先冻结的契约 | `algorithm_structure.md` | §5 关键契约 |
| 仓库怎么切、依赖方向 | `repository_structure.md` | §0–§3 拓扑与判据 |
| 版本怎么锁 | `repository_structure.md` | §5 版本锁定 |
| 分支 / 提交 / CI 规范 | `repository_structure.md` | §6 协作规范 |
| 从现有 fork 怎么过渡 | `repository_structure.md` | §8 迁移路径 |
| 灯条颜色与图案为什么不能同时学到 | `dual_stream_disentanglement.md` | §1 问题拆解 |
| 双流 / 多输入网络怎么搭、怎么融合 | `dual_stream_disentanglement.md` | §3 方案设计 |
| 4:2:0 一致性、颜色交换、属性头怎么训 | `dual_stream_disentanglement.md` | §4 训练与数据 |
| 怎么用消融证明改动有效 | `dual_stream_disentanglement.md` | §5 实验计划 |

---

## 文档关系

```
algorithm_structure.md          算法长什么样（分层 · 世界模型 · 决策接口）
        │
        │  按这套结构切仓
        ▼
repository_structure.md         仓怎么切、怎么协作、怎么发布
        │
        ▼
ZQU-Foray/*                     具体仓库

dual_stream_disentanglement.md  视觉识别专题：灯条颜色 × 图案的特征解耦
        （横切设计，落地到 AutoAim / 感知相关仓）
```
