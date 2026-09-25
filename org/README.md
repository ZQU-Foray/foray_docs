# 回馈组织 `.github` 的改动材料

> 目标仓库：[`ZQU-Foray/.github`](https://github.com/ZQU-Foray/.github)
> 目标文件：`CONTRIBUTING.md`（4 处改动）· `workflows/ci-template.yml`（追加 1 个作业）
> 目的：把「仓库文件规范 / Review 分级 / 多仓协作 / ROS 2 CI」固化进**组织默认文件**——
> **改一次，对全组织生效**，而不是每个仓各抄一份。

## ✅ 当前状态

| 项 | 状态 |
|---|---|
| 分支 | `docs/contributing-multi-repo` |
| 提交 | `eaf367b` docs(contributing) · `fec63e7` chore(ci) |
| 已推送 | ✅ `origin/docs/contributing-multi-repo` |
| 创建 PR | ⬜ **待操作** → https://github.com/ZQU-Foray/.github/pull/new/docs/contributing-multi-repo |
| 目标分支 | ⚠️ 实际为 `main`——`.github` 只有 `main`；组织规范要求 PR 落到 `dev`，但 **`dev` 尚未创建** |

> 先建 `dev` 再改目标分支更符合规范，见下方「提交前必须知道」与 `ROLLOUT.md`。

## 文件对应关系

| 本目录文件 | 落到组织仓库的哪里 | 动作 |
|---|---|---|
| `CONTRIBUTING.addendum.md` | `CONTRIBUTING.md` | 4 处改动，逐处给了原文与替换文 |
| `ci-template.yml` | `workflows/ci-template.yml` | **完整替换**（原有 3 个作业未改动，末尾追加 `build-ros2`） |
| `PR_BODY.md` | 提交 PR 时粘贴到描述框 | 已按 `PULL_REQUEST_TEMPLATE.md` 填好 |
| `ROLLOUT.md` | **不提交**，给队长/组长的落地清单 | 见下 |

---

## ⚠️ 提交前必须知道：`.github` 没有 `dev` 分支

实测组织各仓库分支现状：

| 仓库 | 现有分支 |
|---|---|
| `.github` | `main` |
| `ControllerCode` | `main` |
| `foray_sentry_nav` | `dev-rz` `fork` |
| `WebServer` | `main` |
| `Foray-HelloWorld` | `linnan/ros-check` `main` |

**没有任何一个仓库存在 `dev` 分支。**

而 `CONTRIBUTING.md` 要求「不要在 main 分支进行任何 push 和 merge 操作」——
即所有 PR 应落到 `dev`。**规范已写好，但尚未在任何仓库落地。**

这决定了本次 PR 有两条路径：

| 路径 | 做法 | 评价 |
|---|---|---|
| **A（推荐）** | 先给 `.github` 建 `dev` 分支，再 PR → `dev` | 与规范一致；顺便让 `.github` 成为**第一个真正落地三层分支模型**的仓库 |
| **B** | 直接 PR → `main`，在描述中说明「因 `dev` 尚不存在」 | 更快，但又一次绕过自己的规范 |

建 `dev` 只需队长执行一次：

```bash
git clone git@github.com:ZQU-Foray/.github.git
cd .github
git checkout main && git pull
git checkout -b dev
git push -u origin dev
```

随后在 **Settings → Branches** 把 `main` 与 `dev` 都设为受保护分支
（`main` 禁止直推；`dev` 要求 PR + Review）。

---

## 提交步骤（路径 A）

```bash
# 1. 从 dev 切出功能分支（组织规范要求）
git clone git@github.com:ZQU-Foray/.github.git
cd .github
git checkout dev
git pull origin dev
git checkout -b docs/contributing-multi-repo

# 2. 按 CONTRIBUTING.addendum.md 修改 CONTRIBUTING.md
#    （4 处：目录 / 仓库文件规范 / Review 要求 / 新增多仓协作小节）

# 3. 用 ci-template.yml 覆盖 workflows/ci-template.yml
#    （原有 3 个作业保持不动，仅末尾追加 build-ros2）

# 4. 按逻辑单元分两次提交
git add CONTRIBUTING.md
git commit -m "docs(contributing): 补充仓库文件规范、Review 分级与多仓协作约定"

git add workflows/ci-template.yml
git commit -m "chore(ci): ci-template 增加 ROS 2 colcon 构建作业"

# 5. 推送并创建 PR → dev
git push origin docs/contributing-multi-repo
```

> PR 描述直接粘贴 `PR_BODY.md` 的内容。

---

## 为什么分两次提交

组织规范要求「按逻辑单元小步提交，不得一次性提交全部代码」。
文档约定与 CI 模板是两个独立逻辑单元，审阅人也不同（前者全员相关，后者只影响 CI），
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
