# 命名与代码风格

> 上位机软件系统的名字是 **Diesel**。代码里的顶层命名空间标识是 `diesel`。
> 仓库、ROS 包、话题、tf frame **仍然叫 `foray_*` / 标准名**——原因写在 §3。
>
> 本文是组织内**唯一权威**的命名与风格规范。
> 配套：[`repository_structure.md`](repository_structure.md)（仓拓扑与协作）、
> [`ZQU-Foray/.github/blob/main/CONTRIBUTING.md`](https://github.com/ZQU-Foray/.github/blob/main/CONTRIBUTING.md)（提交与评审流程）。
>
> 决策来源：[ZQU-Foray/.github Discussion #6](https://github.com/orgs/ZQU-Foray/discussions/6)
> ——「是否将 Diesel 作为 Foray 战队上位机软件控制系统命名，并统一 namespace」，投票 2 票「是」/ 0 否，2026-11-03 生效。

---

## 0. 一句话

**系统叫 Diesel，代码里叫 `diesel::`，仓和包装配层叫 `foray_*`，ROS 话题与 tf frame 保持 ROS 标准生态名。**
三层互不越界；任何一层想改，走 §8 的变更流程。

---

## 1. 三层命名边界

| 层 | 标识 | 取值 | 谁在用 | 能否随系统改名 |
|---|---|---|---|---|
| **品牌层** | 组织与仓库名 | `ZQU-Foray` · `foray_localization` | GitHub、`.repos`、文档链接 | ❌ 不随系统名变（仓名 = 团队边界，见 `repository_structure.md` C3/C5） |
| **系统层** | 产品名 | **Diesel** | README、答辩材料、release tag 说明、日志头 | ✅ 就是它 |
| **代码层** | C++ namespace · include 根 · ROS 节点名 | `diesel::…` · `include/diesel/…` · `diesel_*` | 源码、生成的符号、`ros2 node list` | ✅ 跟随系统层 |
| **装配层** | ROS 包名 · 话题 · tf frame | `foray_platform_core` · `/localization/state` · `/map` | launch、`.repos`、ament 依赖、Nav2 插件、外队共享接口 | ❌ 见 §3 |

一句话记法：**改代码看得见的东西（namespace / include / 节点名），不改装配线上被人依赖的东西（包名 / 话题 / tf）。**

---

## 2. 代码标识符规范

### 2.1 C++ namespace：`diesel::<domain>::<module>`

```
diesel::<domain>              ← domain = 仓的领域名（见下表），必填
diesel::<domain>::<module>    ← module = ROS 包名去掉 foray_<domain>_ 前缀后剩下的部分
diesel::<domain>::<module>::detail       ← 实现细节（内部可见）
diesel::<domain>::<module>::test_support ← 测试辅助
```

| 仓 | domain | 包名前缀 | 目标 namespace 示例 |
|---|---|---|---|
| `foray_platform` | `platform` | `foray_platform_` | `diesel::platform` · `diesel::platform::hal` · `diesel::platform::driver::livox::detail` |
| `foray_localization` | `localization` | `foray_localization_` | `diesel::localization` · `diesel::localization::map` · `diesel::localization::ros::test_support` |
| `foray_interfaces` | 协议名本身 | —（无 ROS 包） | `diesel::<protocol>`，如 `diesel::lower_link` |
| `foray_vision` | `vision` | `foray_vision_` | `diesel::vision::detector` |
| `foray_auto_aim` | `auto_aim` | `foray_auto_aim_` | `diesel::auto_aim::solver` |
| `foray_navigation` | `navigation` | `foray_navigation_` | `diesel::navigation::planner` |
| `foray_decision` | `decision` | `foray_decision_` | `diesel::decision::tactic` |
| `foray_robots` | `robots` | `foray_robots_` | `diesel::robots::sentry_bringup`（装配根通常无 C++ 库） |
| `foray_simulation` | `simulation` | `foray_simulation_` | `diesel::simulation::bridge` |
| `foray_ml` | `ml` | — | `diesel::ml` |
| `foray_ws` | — | — | 元仓无源码 |

**硬规则**

- ✅ 第一段永远是 `diesel`，第二段永远是 domain。
- ❌ 禁止用**包名整体**当 namespace：`diesel::localization_map`、`diesel::platform_core` 都不合法。
- ❌ 禁止跳层：`diesel::hal`（缺 domain）、`diesel::platform::foray_platform_core`（缺 module 语义）。
- ❌ 禁止在 `diesel::` 下再挂组织名：`diesel::foray::…`。
- ⚠️ 在 `diesel::localization::map` 这类以标准库名结尾的命名空间内**禁用 `using namespace std;`**
  ——否则 `map` 的可读性会被 `std::map` 干扰，且 ADL 行为会变得难以推理。

> **生成代码的 namespace 只能通过生成器改。** `foray_interfaces/generated/lower_link.hpp` 里的
> `foray::lower_link` 是 `scripts/gen_lower_link.py` 照着 `protocol/lower_link.yaml` 产出的：
> 要改名就改生成器模板再重新生成，**不要手改 `generated/` 下的文件**（下次生成即被覆盖）。
> 这一点与 ROS 消息头不同：ROS 的命名空间由包名决定、我们无法选；protocol 的命名空间是我们自己写的模板。

### 2.2 include 根：`include/diesel/<domain>/…`

```cpp
// ✅
#include "diesel/localization/geometry.hpp"      // 来自 foray/localization/
#include "diesel/localization/map/grid.hpp"      // 来自 foray/localization_map/
#include "diesel/platform/hal/can.hpp"           // 来自 foray/platform/hal/

// ❌
#include "foray/localization/geometry.hpp"
#include "localization/geometry.hpp"
```

- 每个 ROS 包的头文件根目录为 `<pkg>/include/diesel/<domain>/[<module>/]`。
  目录层级与 namespace 层级**一一对应**：`foray::localization` → `diesel/localization/`，
  `foray::localization_map` → `diesel/localization/map/`。有 module 层就不许省。
- `CMakeLists.txt` 的 `include_directories` / `ament_target_dependencies` 与 `install(DIRECTORY include/)` **不用改**（按目录整体安装）。
- 头文件名保持 `snake_case.hpp`。

### 2.3 ROS 节点名：`diesel_<module>`

```cpp
// ✅
rclcpp::Node("diesel_map_server")
// ❌
rclcpp::Node("foray_map_server")
```

节点名对 `ros2 node list`、`ros2 launch` 的 `__node:=` 重映射、以及排障脚本可见，属于**代码层**，跟随系统名。
launch 文件里如果用了节点名匹配，改名时要一起改（同一 PR 内完成）。

### 2.4 Python

Python 侧没有可分发的包命名空间（launch / scripts 直接跑），因此**不引入 `diesel_` 模块前缀**。
只需对齐 §5 的 `ruff` / `black` 配置。

---

## 3. 不变量：明确不跟着改的四样

| 对象 | 现状 | 为什么不动 |
|---|---|---|
| **ROS 包名** | `foray_localization_core` 等 | 包名在 `.repos` pin、`package.xml` 的 `<depend>`、launch 的 `pkg=`、每台机器的 `install/` 目录里都有副本。改名 = 全组织同步窗口 + 所有本地工作区重建，收益只是"看起来一致"。 |
| **ROS 话题** | `/localization/state`、`/localization/relocalize` 等 | 已有**模块域前缀**，语义清晰，与 [`repository_structure.md`](repository_structure.md) §6.6 的 scope 词表同构。加 `/diesel/` 会波及 Nav2 插件、`foray_robots` 的参数文件与跨队共享接口。 |
| **tf frame** | `map` / `odom` / `base_link` / 传感器 frame | ROS 生态约定名，Nav2、robot_state_publisher、RViz 与第三方插件全部硬依赖。加前缀会直接打断 `tf2` 链路。 |
| **仓库名** | `foray_*`、`<upstream>-foray-fork` | 仓名承载团队边界与 owner 划分（C3），与软件品牌无关。 |

> 例外还有一条：**ROS 消息/服务/动作生成的 C++ 类型**（如 `foray_interfaces::msg::X`）由包名直接决定，
> 是生成代码，不手写。**不要**试图给它们套 `diesel::`——包名不变，生成符号就不变。

---

## 4. 旧 → 新映射表

迁移时照抄，替换顺序自上而下（先长后短，避免 `localization` 先吃掉 `localization_map`）。

### 4.1 namespace

| 旧 | 新 |
|---|---|
| `namespace foray::localization_relocalization` | `namespace diesel::localization::relocalization` |
| `namespace foray::localization_map` | `namespace diesel::localization::map` |
| `namespace foray::localization_ros` | `namespace diesel::localization::ros` |
| `namespace foray::localization` | `namespace diesel::localization` |
| `namespace foray::platform` | `namespace diesel::platform` |
| `namespace foray::<protocol>`（生成物，如 `foray_interfaces/generated/lower_link.hpp` 里的 `foray::lower_link`） | `namespace diesel::<protocol>`——改 `scripts/gen_lower_link.py` 模板后重新生成 |
| `foray::` 的其他任何出现（含 `} // namespace` 尾注释、限定调用 `foray::x::y`） | `diesel::` |

### 4.2 include 根

| 旧目录 | 新目录 |
|---|---|
| `include/foray/localization_relocalization/` | `include/diesel/localization/relocalization/` |
| `include/foray/localization_map/` | `include/diesel/localization/map/` |
| `include/foray/localization_ros/` | `include/diesel/localization/ros/` |
| `include/foray/localization/` | `include/diesel/localization/` |
| `include/foray/platform/…` | `include/diesel/platform/…` |

> 目录迁移用 `git mv`（保留历史），不要用 `cp` + `rm`。

### 4.3 节点名与字符串常量

| 旧 | 新 |
|---|---|
| `"foray_map_server"` | `"diesel_map_server"` |
| `"foray_livox"` | `"diesel_livox"` |
| launch / 参数 / 测试里的其他 `foray_*` 节点名 | `diesel_*` |

### 4.4 迁移前先自查

```bash
# 待改项清单（结果必须为空才能把 mode 改成 enforced）
grep -rn 'namespace foray\|foray::' --include='*.hpp' --include='*.cpp' --include='*.h' --include='*.cc' .
grep -rn '#include [<"]foray/' --include='*.hpp' --include='*.cpp' --include='*.h' .
find . -type d -name foray -path '*/include/*'
grep -rn 'Node("foray_\|"foray_[a-z_]*"' --include='*.cpp' --include='*.py' .
```

---

## 5. 代码风格的唯一源

风格配置**不再各仓自己定**，canonical 放在组织仓库：

| 文件 | 位置 | 作用 |
|---|---|---|
| `.clang-format` | [`ZQU-Foray/.github/config/.clang-format`](https://github.com/ZQU-Foray/.github/blob/main/config/.clang-format) | C/C++ 唯一风格（4 空格 · LF · 100 列 · 指针左靠） |
| `pyproject.toml` | [`ZQU-Foray/.github/config/pyproject.toml`](https://github.com/ZQU-Foray/.github/blob/main/config/pyproject.toml) | `ruff` + `black` 的唯一配置（py310 / 88 列） |
| `.editorconfig` | [`ZQU-Foray/.github/config/.editorconfig`](https://github.com/ZQU-Foray/.github/blob/main/config/.editorconfig) | 缩进 / 换行 / 末尾空行 / 编码，编辑器侧的兜底 |

**用法**：新建仓直接复制这三个文件到仓根；已有仓覆盖为 canonical 版本后跑一次格式化，单独提交 `chore(style):`。

### 5.1 迁移前的实际差异（2026-11-03 盘点）

| 仓 | `.clang-format` | `pyproject.toml` | 处理 |
|---|---|---|---|
| `foray_platform` | 34 行组织版 | 无 | 复制 canonical，补 pyproject |
| `foray_simulation` | 34 行组织版 | 15 行（含 black 段） | 替换为 canonical |
| `foray_interfaces` | 34 行组织版 | 无 | 替换为 canonical（含生成器脚本） |
| `foray_localization` | **10 行精简版**（缺 `AllowShortIfStatements` 等） | 5 行（仅 ruff target-version） | **必须替换**，否则与其它仓格式化结果不一致 |
| `foray_navigation` | 缺失 | 无 | 补 |
| 其余仓 | 缺失 | 无 | 建仓时复制 |

### 5.2 命名风格（语言层约定，不变）

| 项 | 规则 |
|---|---|
| Python | 变量/函数 `snake_case` · 类 `PascalCase` · 常量 `UPPER_SNAKE_CASE` · 私有 `_` 前缀 |
| C/C++ | 变量/函数 `snake_case` · 结构体/类 `PascalCase` · 宏/枚举 `UPPER_SNAKE_CASE` |
| 文件 | C++ `snake_case.hpp/.cpp` · Python `snake_case.py` · launch `<功能>.launch.py` |

---

## 6. 迁移状态与待办

代码仓的实际改名由**各仓负责人**自行跟进，组织只提供规范与门禁。
每个仓在仓根放一个声明文件 `.diesel-naming`，CI 据此决定严格程度：

```yaml
# .diesel-naming —— 命名迁移状态，组织 CI 的 namespace-guard / style-config 作业读取
mode: migrating        # enforced | migrating | na
owner: 导航组
note: 预计 11 月第三周完成
```

| mode | namespace-guard | style-config | 用于 |
|---|---|---|---|
| `enforced` | 发现 `foray::` / `include "foray/` 即 **失败** | 配置与 canonical 不一致即**失败** | 已迁移完成的仓 |
| `migrating` | 只输出**警告注释**，作业通过 | 同上，只警告 | 迁移中的仓 |
| `na` | 跳过 | 跳过 | 非 C++ 仓 / 元仓 / 文档仓 |

> 声明文件缺失时，CI 按 `migrating` 处理并给出一条提示——**保证新仓和空仓不会因为还没改名就红**。

### 6.1 各仓状态（2026-11-03，由各仓负责人更新）

| 仓 | C++ 文件数 | 现状 | mode | 负责人待办 |
|---|---|---|---|---|
| `foray_platform` | 172 | `foray::platform::…`（层级已对，只换顶层） | `migrating` | 换顶层 + include 根 + 节点名 |
| `foray_localization` | 63 | `foray::localization_map/_ros/_relocalization`（**层级也要重排**） | `migrating` | 先合 PR #7/#8，再按 §4 顺序改 |
| `foray_simulation` | 0 | 无 C++ | `na`（仅对齐风格配置） | 换 `.clang-format` / `pyproject.toml` |
| `foray_interfaces` | 0 | 生成器 + 协议；生成代码跟包名走 | `na` | 无 C++ 改名 |
| `foray_navigation` | 0 | 骨架 | `na` | 建仓时直接按新规范 |
| `foray_vision` / `foray_auto_aim` / `foray_decision` / `foray_robots` | 0 | 空仓 | `na` | 首次提交即用 `diesel::` |
| `foray_ml` / `foray_ws` | 0 | 工具 / 元仓 | `na` | 无 |

### 6.2 五步迁移法（单仓）

```bash
# 1. 从 dev 切分支
git checkout dev && git pull && git checkout -b refactor/diesel-namespace

# 2. 目录先动（git mv 保留历史）
git mv src/*/include/foray/localization_map src/*/include/diesel/localization/map   # 逐包做

# 3. 代码替换（按 §4 顺序，先长后短）
grep -rl 'foray::localization_relocalization' --include='*.hpp' --include='*.cpp' . | xargs sed -i 's/foray::localization_relocalization/diesel::localization::relocalization/g'
# …同理处理 localization_map / localization_ros / localization / platform

# 4. include 与节点名
grep -rl '#include [<"]foray/' --include='*.hpp' --include='*.cpp' . | xargs sed -i 's|\(#include [<"]\)foray/|\1diesel/|g'

# 5. 验证（在容器里，不要在宿主上试）
docker exec foray_sim_run bash -lc 'source /opt/ros/humble/setup.bash; cd /ws && colcon build --base-paths src --packages-select <pkgs> && colcon test && colcon test-result --all'
```

全部通过后，把 `.diesel-naming` 的 `mode` 从 `migrating` 改成 `enforced`，再开第二个 PR 对齐风格配置。

---

## 7. 门禁与检查

组织 CI 模板 [`ci-template.yml`](https://github.com/ZQU-Foray/.github/blob/main/workflows/ci-template.yml) 新增两个作业：

| 作业 | 检查内容 | 失败条件 |
|---|---|---|
| `namespace-guard` | `namespace foray` / `foray::` / `#include "foray/` / 遗留 `include/foray/` 目录 / `Node("foray_*")` | 仅 `mode: enforced` 时失败；`migrating` 输出警告 |
| `style-config` | 仓内 `.clang-format`、`pyproject.toml` 与 `config/` canonical 逐字节比对 | 仅 `mode: enforced` 时失败；`migrating` 输出警告 |

本地自查（提交前跑一遍，比等 CI 快）：

```bash
diff -u <(curl -fsSL https://raw.githubusercontent.com/ZQU-Foray/.github/main/config/.clang-format) .clang-format
grep -rn 'namespace foray\|foray::' --include='*.hpp' --include='*.cpp' . | head
```

---

## 8. 变更流程

- 本文与 `config/` canonical 文件**同 PR 修改**，不允许只改一边。
- 改命名边界（§1 三层、§3 不变量）需要先在 Discussions 投票，并在本文件头部记录结论与日期。
- 单纯补一张映射表条目、修错别字、更新迁移状态：直接 PR 到 `main`（文档仓免三级分支，见 `repository_structure.md` §6.1）。

---

## 9. 现状速查

```text
组织/品牌     ZQU-Foray · foray_*        （不变）
系统名        Diesel                      （投票结果）
C++ namespace diesel::<domain>::<module>
include 根    include/diesel/<domain>/…
ROS 节点名    diesel_<module>
ROS 包名      foray_<repo>_<module>       （不变）
话题          /<module>/<entity>          （不变）
tf frame      map / odom / base_link …    （不变）
风格配置      .github/config/ 下的三个文件（唯一源）
```
