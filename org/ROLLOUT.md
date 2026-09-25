# 组织规范落地清单（**不必提交，给队长/组长**）

> 这份文件回答一个比「再写点规范」更重要的问题：
> **为什么你们的规范已经写得不错，但协作仍然没有按规范发生。**

---

## 0. 核心判断

`ZQU-Foray/.github` 的规范质量**高于绝大多数学生战队**——
Conventional Commits、分支模型、Review 要求、代码风格、Issue/PR 模板、CI 模板俱全。

**问题不在内容，在落地。** 实测现状：

| 仓库 | `dev` 分支 | CI 配置 | `CODEOWNERS` | 按规范的分支命名 |
|---|---|---|---|---|
| `.github` | ❌ 无 | — | 骨架（三节为空） | — |
| `ControllerCode` | ❌ 无 | ❓ | ❓ | — |
| `foray_sentry_nav` | ❌ 无（只有 `dev-rz` `fork`） | ❓ | ❓ | ❌ `dev-rz` 非 `feat/xxx` |
| `WebServer` | ❌ 无 | ❓ | ❓ | — |
| `Foray-HelloWorld` | ❌ 无（只有 `main`） | ❓ | ❓ | ❌ `linnan/ros-check` 非 `feat/xxx` |

⇒ **规范里写的三层分支模型，全组织一处都没实现。** 所有仓库只有 `main`。

这解释了一个常见现象：规范发布时大家看过，但**第一个 PR 就绕过去了**（因为
`dev` 不存在，只能 PR 到 `main`），之后规范就只是文档。

> **规范的生命力取决于第一次执行是否顺畅。** 所以落地比增补更重要。

---

## 1. 最高优先级：把分支模型真正建起来（约 30 分钟）

这是**唯一一件做完就能让后续所有协作自动合规**的事。

### 对每个活跃仓库执行

```bash
# 建 dev 分支
git checkout main && git pull
git checkout -b dev
git push -u origin dev
```

然后在 GitHub **Settings → Branches → Add branch protection rule**：

| 分支 | 保护设置 |
|---|---|
| `main` | ✅ 禁止直接 push ✅ 禁止 force push ✅ 要求 PR ✅ 要求 1 个 approve |
| `dev` | ✅ 禁止直接 push ✅ 要求 PR ✅ 要求 1 个 approve |

> `CONTRIBUTING.md` 说「不要在 main 分支进行任何 push 和 merge」——
> **光靠这句话是拦不住的，必须由分支保护强制执行。**
> 规范文档负责「让人知道」，分支保护负责「让人做不到」。

**建议顺序**：`.github` → `foray_sentry_nav` → `ControllerCode` → `WebServer`

---

## 2. 次高优先级：让 CI 真的能挡住问题

### 2.1 ROS 2 仓库

`foray_sentry_nav` 是 ROS 2 / Nav2 工作空间，**当前 CI 模板对它等于没做集成验证**
（模板只有 Python 与格式检查，没有一行 colcon）。后果：`colcon build` 全挂，CI 仍显示绿灯。

⇒ 合并本目录的 CI 补丁后，把模板复制到 `.github/workflows/ci.yml`。
先跑通一次，确认能真的拦住一个编译错误。

### 2.2 建议增补的 CI 内容（各仓按需）

| 检查 | 适用 | 为什么值得 |
|---|---|---|
| `clang-format` 强制 | 所有 C/C++ 仓 | 已在模板里；配合 `.clang-format` 才能生效——**先补配置文件** |
| `ruff` / `black` | Python 仓 | 已在模板里；同样需要 `pyproject.toml` |
| `colcon build` + `test` | ROS 2 仓 | **本次补丁新增** |
| 依赖方向校验 | 分层架构的多仓项目 | 让「解耦」从文档变事实（见项目文档 §6.5） |

> ⚠️ 模板里跑 `clang-format` / `ruff` / `black`，但仓库若**没有对应配置文件**，
> 会使用工具默认风格——与「4 空格 / LF」的规范可能不一致。
> **落地 CI 时必须同时提交配置文件**，否则 CI 会因风格不符而普遍失败，然后被绕过。

---

## 3. 中优先级：`CODEOWNERS`

当前 `.github/CODEOWNERS` 只有一行 `* @队长用户名`，视觉 / 电控 / CI 三节全空。

⇒ **所有 PR 都堆给队长**。多仓之后这是瓶颈，且队长成为唯一单点。

### 建议填充

```gitignore
# 全局兜底
*                       @队长用户名

# 组织规范与 CI
/.github/               @队长用户名
CODEOWNERS              @队长用户名
CONTRIBUTING.md         @队长用户名
/workflows/             @队长用户名

# 按方向（填入真实用户名或 team）
# 视觉
# 电控
```

> 各仓库自己的 `CODEOWNERS` 更精细，但**组织级 `.github/CODEOWNERS` 优先级更低**，
> 会被仓库内的覆盖——所以组织级只需做方向级兜底。

---

## 4. 低优先级：把已有仓库对齐文件规范

按本次增补的五件套逐仓补齐。**不必一次全做**，按活跃度排序：

| 仓库 | `README.md` 内容完整度 | 建议补 |
|---|---|---|
| `foray_sentry_nav` | 待评估 | `README.md` 完善（层归属/快速开始/接口）+ `decision.md` + `tree.md` |
| `ControllerCode` | 待评估 | `README.md` + `.clang-format` |
| `WebServer` | 待评估 | `README.md` + `pyproject.toml` |
| `.github` | 已有 `profile/README.md` | 补 `CODEOWNERS` 内容 |

> 顺带一提：`CONTRIBUTING.md` 里「技术文档 → docs 仓库」指向
> `https://github.com/ZQU-Foray/docs`，但该仓库当前返回 404（未建或私有）。
> 要么建它，要么改掉这个链接，避免新队员点进去是死链。

---

## 5. 建议的执行顺序（一页）

```
第 1 步（今天，30 分钟）
  └─ 为 .github 与 foray_sentry_nav 建 dev 分支 + 分支保护
     ⇒ 从这一刻起，所有 PR 自动落在 dev，规范开始自我执行

第 2 步（本周）
  └─ 合并本目录的 CONTRIBUTING + CI 补丁
  └─ 把 CI 模板铺到 foray_sentry_nav，确认能拦住一次真实编译错误

第 3 步（本月）
  └─ 补齐 CODEOWNERS（组织级 + 仓库级），解除队长单点
  └─ 其余仓库补 dev 分支与分支保护

第 4 步（随新仓建设）
  └─ 按项目文档 §11.2 的 Checklist 建新仓，一步到位
```

> **判断落地成功的唯一标准**：下一次有人想图省事直接 push `main` 时，
> 他**做不到**——而不是「记得规范里说过不行」。
