## 📝 描述

补充组织协作规范，使其覆盖**多仓并行开发**场景。共四类改动：

1. **`CONTRIBUTING.md`**
   - 把「每个仓库必须包含 `README.md`」由建议改为**强制**，并补充大写文件名提醒
     （Linux / ROS 2 区分大小写，`readme.md` 会导致「文档存在但读不到」）
   - 新增多仓协作推荐文件：`AGENTS.md` / `plan.md` / `tree.md` / `decision.md`，
     其中 `decision.md` 用于留痕 **fork patch、依赖换版本、破坏性接口变更**
   - 「Review 要求」增补两条：**跨仓库契约类改动需 ≥2 人 Review 且评估兼容性影响**；
     **依赖清单（`.repos`）版本变更需说明理由**
   - 新增「多仓协作」小节：三层版本机制、`.repos` 而非 `git submodule`、
     meta tag 版本锚点、依赖方向校验
   - 补充 `CODEOWNERS` 要求（把 Review 分派到方向负责人，而非全堆给队长）
   - **明确「文档类仓库」不需要三级分支模型**：以 Markdown / 配置为主、
     不含构建产物的仓库（`.github`、`foray_docs` 等）可直接在 `main` 上工作。
     原规范「不要在 main 分支做任何 push 和 merge」是无条件表述，
     而这类仓库本就不适用三级分支——不写明例外会造成「规范说不行、实际只能这么做」

2. **`workflows/ci-template.yml`**
   - 追加 `build-ros2` 作业（`rosdep` + `colcon build` + `colcon test`）
   - 原有 `lint-python` / `lint-c` / `test-python` **未作任何修改**
   - 该作业带 `package.xml` 探测，仓库内无 ROS 2 package 时整个作业自动跳过，
     因此可安全保留在通用模板中，不影响纯 Python / 纯 C 仓库

## 🔗 关联 Issue

Closes #

## 🧪 测试

- [x] 改动均为 Markdown / YAML 文本，不涉及代码逻辑
- [x] `ci-template.yml` 已校验 YAML 语法
- [x] `build-ros2` 的跳过逻辑在无 `package.xml` 的仓库中会正常短路
- [ ] **待验证**：合并后在 `foray_sentry_nav` 复制该模板，确认 CI 能实际跑通 colcon 构建

## 📸 效果展示

改动前后对比：

| 项 | 改动前 | 改动后 |
| --- | --- | --- |
| `README.md` | 「必须包含」（未强调强制） | **强制，无例外** + 大小写提醒 |
| 协作文件 | 仅 `README.md` + 可选 `docs/` | + `AGENTS.md` / `plan.md` / `tree.md` / `decision.md` |
| Review | 统一 ≥1 人 | 契约类改动 **≥2 人 + 兼容性评估** |
| 多仓协作 | 未覆盖 | 三层版本机制 + `.repos` + meta tag + 依赖方向 |
| ROS 2 CI | **无任何构建任务** | `build-ros2`（自动跳过非 ROS 2 仓） |
| 文档仓分支 | 与代码仓同规则（无条件禁 main） | **文档类仓库免三级分支**，直接用 `main` |

## 🗒️ Checklist

- [x] 代码符合风格规范（格式化工具有跑）—— 本次仅文档与 YAML
- [x] 提交符合 [Conventional Commits](https://www.conventionalcommits.org/)
- [x] 添加了必要的注释（CI 作业内注明适用仓库与跳过条件）
- [x] 没有调试代码、临时文件、密钥
- [x] README / 文档已同步更新

## 📎 补充说明

**请重点评审以下几条**：
- **Review 分级（≥2 人）** 是对现有「≥1 人」的加严。理由：接口/依赖清单类改动
  影响面跨仓，单人对「改一处崩一片」防不住。若认为对电控方向过重，可限定为
  「跨仓库契约类改动」适用。
- **ROS 2 CI 作业** 是本次唯一的实质性能力补充。当前模板对 colcon 工作空间仓库
  等于没有集成验证——`foray_sentry_nav` 这类仓库即使构建全挂，CI 也会显示绿灯。
- **文档类仓库免三级分支**（第 6 条）是对现有分支规则的**例外说明**，不是放宽代码仓要求。
  判据是「**是否含构建产物**」：文档仓没有可构建的东西，`dev` 只是多一次合并。
  本 PR 的目标分支 `main` 正是依据这一条。

**另有一项现状问题，建议一并处理（见下方）**：

实测组织各仓库分支现状，按类型区分：

| 仓库 | 类型 | 现有分支 | 是否应有 `dev` |
| --- | --- | --- | --- |
| `.github` | 文档 | `main` | ❌ 不需要（本 PR 的改动 5 明确了这一例外） |
| `ControllerCode` | 代码 | `main` | ✅ **应有，但缺失** |
| `foray_sentry_nav` | 代码 | `dev-rz` `fork` | ✅ **应有，但缺失** |
| `WebServer` | 代码 | `main` | ✅ **应有，但缺失** |
| `Foray-HelloWorld` | 代码 | `linnan/ros-check` `main` | ✅ **应有，但缺失** |

即**所有代码类仓库都没有 `dev` 分支**，而 `CONTRIBUTING.md` 要求所有 PR 落到 `dev`。
规范已写好但尚未在代码仓落地。建议：

1. 为**代码类仓库**（`ControllerCode` / `foray_sentry_nav` / `WebServer` / `Foray-HelloWorld`）
   补 `dev` 分支
2. 在 GitHub **Settings → Branches** 为 `main` / `dev` 设置分支保护
   （文档负责「让人知道」，分支保护负责「让人做不到」）
3. 补齐 `CODEOWNERS`（当前视觉 / 电控 / CI 三节为空，全局仅 `*`，
   导致所有 Review 堆给队长）

> 本 PR 的目标分支为 `main`——`.github` 属文档类仓库，按改动 5 **符合规范**，
> 不需要先建 `dev`。

@队长
