# 回馈组织 `.github` 的改动材料

> 目标仓库：[`ZQU-Foray/.github`](https://github.com/ZQU-Foray/.github)
> 目标文件：`CONTRIBUTING.md`（5 处改动）· `workflows/ci-template.yml`（追加 1 个作业）
> 目的：把「仓库文件规范 / Review 分级 / 多仓协作 / ROS 2 CI」固化进**组织默认文件**——
> **改一次，对全组织生效**，而不是每个仓各抄一份。

## ✅ 当前状态

| 项 | 状态 |
|---|---|
| 分支 | `docs/contributing-multi-repo` |
| 提交 | `eaf367b` docs(contributing) · `fec63e7` chore(ci) · `0a8eb79` docs(contributing) |
| 已推送 | ✅ `origin/docs/contributing-multi-repo`（领先 `main` 3 个提交） |
| 创建 PR | ⬜ **待操作** → https://github.com/ZQU-Foray/.github/pull/new/docs/contributing-multi-repo |
| 目标分支 | ✅ `main`——`.github` 属**文档类仓库**（无构建产物），按改动 5 明确**不需要三级分支模型** |

> ⚠️ 注意：若改动 5 未被接受，则本 PR 应改提 `dev`（需先创建该分支）。
> 改动 5 正是为了让「PR → `main`」**符合规范**，而不是绕过规范。

## 文件对应关系

| 本目录文件 | 落到组织仓库的哪里 | 动作 |
|---|---|---|
| `CONTRIBUTING.addendum.md` | `CONTRIBUTING.md` | 5 处改动，逐处给了原文与替换文 |
| `ci-template.yml` | `workflows/ci-template.yml` | **完整替换**（原有 3 个作业未改动，末尾追加 `build-ros2`） |
| `PR_BODY.md` | 提交 PR 时粘贴到描述框 | 已按 `PULL_REQUEST_TEMPLATE.md` 填好 |
| `ROLLOUT.md` | **不提交**，给队长/组长的落地清单 | 见下 |

---

## ✅ `.github` 的 PR 目标就是 `main`，且符合规范

`.github` 属于**文档类仓库**（以 Markdown / 配置为主、**无构建产物**）。
按改动 5 明确的规则，这类仓库**不需要三级分支模型**——
直接 PR 到 `main` 是**符合规范**的做法，而不是绕过规范。

### 但代码类仓库确实缺 `dev`（另一个问题）

实测组织各仓库分支现状，按类型区分：

| 仓库 | 类型 | 现有分支 | 是否应有 `dev` |
|---|---|---|---|
| `.github` | 文档 | `main` | ❌ 不需要（文档仓例外） |
| `foray_docs` | 文档 | `main` | ❌ 不需要（文档仓例外） |
| `ControllerCode` | **代码** | `main` | ✅ **应有，但缺失** |
| `foray_sentry_nav` | **代码** | `dev-rz` `fork` | ✅ **应有，但缺失** |
| `WebServer` | **代码** | `main` | ✅ **应有，但缺失** |
| `Foray-HelloWorld` | **代码** | `linnan/ros-check` `main` | ✅ **应有，但缺失** |

**所有代码类仓库都没有 `dev` 分支**，而 `CONTRIBUTING.md` 要求所有 PR 落到 `dev`。
规范已写好，但**尚未在任何代码仓落地**——这正是「第一个 PR 就绕过去了」的根源。

为代码仓建 `dev` 只需执行一次：

```bash
git checkout main && git pull
git checkout -b dev
git push -u origin dev
```

随后在 **Settings → Branches** 把 `main` 与 `dev` 都设为受保护分支
（`main` 禁止直推；`dev` 要求 PR + Review）。

---

## 提交步骤（路径 A）

```bash
# 1. 从 main 切出功能分支（.github 属文档类仓库，不走 dev）
git clone git@github.com:ZQU-Foray/.github.git
cd .github
git checkout main
git pull origin main
git checkout -b docs/contributing-multi-repo

# 2. 按 CONTRIBUTING.addendum.md 修改 CONTRIBUTING.md
#    （5 处：目录 / 仓库文件规范 / Review 要求 / 多仓协作 / 文档仓分支例外）

# 3. 用 ci-template.yml 覆盖 workflows/ci-template.yml
#    （原有 3 个作业保持不动，仅末尾追加 build-ros2）

# 4. 按逻辑单元分三次提交
git add CONTRIBUTING.md
git commit -m "docs(contributing): 补充仓库文件规范、Review 分级与多仓协作约定"

git add workflows/ci-template.yml
git commit -m "chore(ci): ci-template 增加 ROS 2 colcon 构建作业"

git add CONTRIBUTING.md
git commit -m "docs(contributing): 明确文档类仓库不需要三级分支模型"

# 5. 推送并创建 PR → main
git push origin docs/contributing-multi-repo
```

> PR 描述直接粘贴 `PR_BODY.md` 的内容。

---

## 为什么分三次提交

组织规范要求「按逻辑单元小步提交，不得一次性提交全部代码」。
文档约定、CI 模板、分支规则例外是三个独立逻辑单元，审阅人也不同
（文档全员相关、CI 只影响自动化、分支例外影响所有人的日常操作），
分开提交可让 Review 各看各的。

---

## 改动要点回顾

1. **`README.md` 强制化**——把原本的「必须包含」写死为「强制，无例外」，
   并补上「大写 `README.md`」的大小写提醒（Linux / ROS 2 区分大小写）。
2. **新增五件套**——`AGENTS.md` / `plan.md` / `tree.md` / `decision.md`，
   并明确 `decision.md` 用于留痕 **fork patch、依赖换版本、破坏性接口变更**。
3. **Review 分级**——跨仓库契约类改动 **≥2 人 + 兼容性评估**；依赖清单换版本需说明理由。
4. **新增多仓协作小节**——三层版本机制、`.repos` 而非 submodule、meta tag、依赖方向校验。
5. **CI 补 ROS 2 作业**——原模板**只有 Python 与格式检查，没有任何构建任务**，
   对 colcon 工作空间仓库等于没做集成验证。
6. **文档类仓库免三级分支**——`.github` / `foray_docs` 这类无构建产物的仓库，
   直接在 `main` 上工作；同时也让组织内其他纯文档仓有据可依。
