# 命名与代码风格

> 上位机软件系统的名字是 **Diesel**。代码里的顶层命名空间标识是 `diesel`。
> **仓库名、ROS 包名、include 根、节点名也统一为 `diesel_*` / `diesel::`**；
> 只有 ROS 话题与 tf frame 保持生态标准名——原因写在 §3。
> `foray` 只作为战队品牌留在组织名（`ZQU-Foray`）与非上位机仓（`foray_docs` / `foray_ml` / `foray_robots`）。
>
> 本文是组织内**唯一权威**的命名与风格规范。
> 配套：[`repository_structure.md`](repository_structure.md)（仓拓扑与协作）、
> [`ZQU-Foray/.github/blob/main/CONTRIBUTING.md`](https://github.com/ZQU-Foray/.github/blob/main/CONTRIBUTING.md)（提交与评审流程）。
>
> 决策来源：[ZQU-Foray/.github Discussion #6](https://github.com/orgs/ZQU-Foray/discussions/6)
> ——「是否将 Diesel 作为 Foray 战队上位机软件控制系统命名，并统一 namespace」，投票 2 票「是」/ 0 否，2026-11-03 生效。

---

## 0. 一句话

**系统叫 Diesel：仓名、ROS 包名、namespace、include 根、节点名全部用 `diesel_*` / `diesel::`；只有 ROS 话题与 tf frame 保持生态标准名。**
三层互不越界；任何一层想改，走 §8 的变更流程。

---

## 1. 三层命名边界

| 层 | 标识 | 取值 | 谁在用 | 能否随系统改名 |
|---|---|---|---|---|
| **品牌层** | 组织名与战队资产 | `ZQU-Foray` · `foray_docs` · `foray_ml` · `foray_robots` · `foray_ws` | GitHub 组织页、战队文档、训练产线 | ❌ 战队品牌，与软件系统无关 |
| **系统层** | 产品名 | **Diesel** | README、答辩材料、release tag 说明、日志头 | ✅ 就是它 |
| **代码层** | 仓库名 · ROS 包名 · C++ namespace · include 根 · ROS 节点名 | `diesel_<domain>` · `diesel_<domain>_<module>` · `diesel::…` · `include/diesel/…` · `diesel_*` | 源码、`package.xml`、launch、`.repos`、CI、`ros2 node list` | ✅ 跟随系统层 |
| **生态层** | ROS 话题 · tf frame | `/localization/state` · `map`/`odom`/`base_link` | Nav2、RViz、robot_state_publisher、第三方插件、外队共享接口 | ❌ 见 §3 |

一句话记法：**Diesel 的软件栈从仓名到节点名全部叫 `diesel_*`，只有 ROS 生态约定的话题与 tf frame 不动。**

---

## 2. 代码层标识规范

本章管五样东西：**仓库名、ROS 包名、C++ namespace、include 根、ROS 节点名**。它们的对应关系是死的：

```
仓库 diesel_<domain>  →  包 diesel_<domain>_<module>  →  namespace diesel::<domain>::<module>
                            include/diesel/<domain>/[<module>/]  ·  节点名 diesel_<module>
```

### 2.1 仓名 / 包名 / namespace 的对应关系

```
diesel::<domain>              ← domain = 仓的领域名（见下表），必填
diesel::<domain>::<module>    ← module = ROS 包名去掉 diesel_<domain>_ 前缀后剩下的部分
diesel::<domain>::<module>::detail       ← 实现细节（内部可见）
diesel::<domain>::<module>::test_support ← 测试辅助
```

| 旧仓名 | 目标仓名 | domain | ROS 包名前缀 | 目标 namespace 示例 |
|---|---|---|---|---|
| `foray_platform` | `diesel_platform` | `platform` | `diesel_platform_` | `diesel::platform` · `diesel::platform::hal` · `diesel::platform::driver::livox::detail` |
| `foray_localization` | `diesel_localization` | `localization` | `diesel_localization_` | `diesel::localization` · `diesel::localization::map` · `diesel::localization::ros::test_support` |
| `foray_interfaces` | `diesel_interfaces` | 协议名本身 | —（无 ROS 包） | `diesel::<protocol>`，如 `diesel::lower_link` |
| `foray_simulation` | `diesel_simulation` | `simulation` | `diesel_simulation_` | `diesel::simulation::bridge` |
| `foray_navigation` | `diesel_navigation` | `navigation` | `diesel_navigation_` | `diesel::navigation::planner` |
| `foray_vision` | `diesel_vision` | `vision` | `diesel_vision_` | `diesel::vision::detector` |
| `foray_auto_aim` | `diesel_auto_aim` | `auto_aim` | `diesel_auto_aim_` | `diesel::auto_aim::solver` |
| `foray_decision` | `diesel_decision` | `decision` | `diesel_decision_` | `diesel::decision::tactic` |

> **不参与本次改名的仓**（`foray` 作为战队品牌保留）：`foray_docs`（战队文档）、`foray_ml`（训练产线）、
> `foray_robots`（实体机器人与上场配置）、`foray_ws`（元仓）、`foray-HelloWorld`（招呼仓）。
> 判据是「是否属于 Diesel 上位机软件栈」——上面 8 个仓编译成一个系统，其余仓不参与编译装配。

**仓名与包名规则**

- 仓库名 = `diesel_<domain>`，domain 取值见上表，与 namespace 第二段**逐字一致**。
- ROS 包名 = `diesel_<domain>_<module>`，**只换前缀、保留原有结构**：`foray_localization_map` → `diesel_localization_map`，
  `foray_platform_driver_livox` → `diesel_platform_driver_livox`。
  后端两个包 `foray_rmul_2027_field` / `foray_simulation_bringup` 不以 domain 开头，同样只换前缀：
  `diesel_rmul_2027_field` / `diesel_simulation_bringup`。
- 消息/服务/动作包（若将来单独拆出）= `diesel_interfaces`，生成类型即 `diesel_interfaces::msg::X`。
- 校验口径：`package.xml` 的 `<name>`、`CMakeLists.txt` 的 `project()`、目录名 `src/<pkg>/`、
  以及 launch 里的 `package=`、`package.xml` 的 `<depend>`、`.repos` 的 pin，必须**同一 PR 内全部对齐**。

**硬规则**

- ✅ 第一段永远是 `diesel`，第二段永远是 domain。
- ❌ 禁止用**包名整体**当 namespace：包 `diesel_localization_map` 对应的是 `diesel::localization::map`，
  既不是 `diesel::localization_map`，也不是叠一层前缀的 `diesel::diesel_localization_map`。
- ❌ 禁止跳层：`diesel::hal`（缺 domain）、`diesel::platform::foray_platform_core`（缺 module 语义）。
- ❌ 禁止在 `diesel::` 下再挂组织名：`diesel::foray::…`。
- ⚠️ 在 `diesel::localization::map` 这类以标准库名结尾的命名空间内**禁用 `using namespace std;`**
  ——否则 `map` 的可读性会被 `std::map` 干扰，且 ADL 行为会变得难以推理。

> **生成代码的 namespace 只能通过生成器改。** `diesel_interfaces/generated/lower_link.hpp` 里的
> `diesel::lower_link` 是 `scripts/gen_lower_link.py` 照着 `protocol/lower_link.yaml` 产出的：
> 要改名就改生成器模板再重新生成，**不要手改 `generated/` 下的文件**（下次生成即被覆盖）。
> 这一点与 ROS 消息头不同：ROS 的命名空间由包名决定、我们无法选；protocol 的命名空间是我们自己写的模板。

### 2.2 include 根：`include/diesel/<domain>/…`

```cpp
// ✅
#include "diesel/localization/geometry.hpp"      // 原 foray/localization/（diesel_localization_core 包）
#include "diesel/localization/map/grid.hpp"      // 原 foray/localization_map/（diesel_localization_map 包）
#include "diesel/platform/hal/can.hpp"           // 原 foray/platform/hal/（diesel_platform_core 包）

// ❌
#include "foray/localization/geometry.hpp"
#include "localization/geometry.hpp"
```

- 每个 ROS 包的头文件根目录为 `<pkg>/include/diesel/<domain>/[<module>/]`。
  目录层级与 namespace 层级**一一对应**：`diesel::localization` → `diesel/localization/`，
  `diesel::localization::map` → `diesel/localization/map/`。有 module 层就不许省。
- `CMakeLists.txt` 的 `include_directories` 与 `install(DIRECTORY include/)` **不用改**（按目录整体安装）；
  但 `find_package()` / `ament_target_dependencies()` 里列的**包名**要随包改名同步（见 §4.4）。
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
只需两件事：对齐 §5 的 `ruff` / `black` 配置；把 launch 里引用的**包名**改成 `diesel_*`（那是 ROS 包名，不是 Python 模块名）。

---

## 3. 不变量：只留两样

| 对象 | 现状 | 为什么不动 |
|---|---|---|
| **ROS 话题** | `/localization/state`、`/localization/relocalize` 等 | 已有**模块域前缀**，语义清晰，与 [`repository_structure.md`](repository_structure.md) §6.6 的 scope 词表同构。加 `/diesel/` 只是重复一次系统名，收益为零，却会波及 Nav2 插件、`foray_robots` 的参数文件与跨队共享接口。 |
| **tf frame** | `map` / `odom` / `base_link` / 传感器 frame | ROS 生态约定名（REP-105），Nav2、robot_state_publisher、RViz 与第三方插件全部硬依赖。加前缀会直接打断 `tf2` 链路。 |

**仓名与包名为什么不在此列。** 改包名的成本是真的：`.repos` 的 pin、`package.xml` 的 `<depend>`、
launch 的 `package=`、每台机器 `install/` 下的目录名，都要在同一个同步窗口里对齐。
但这笔成本**只在改名当次付一次**，而「这个仓/这个包是不是 Diesel 的」这个问题每来一个新人、每读一次
`colcon list` 都要付一次。话题和 tf 的处境相反：它们已经有正确的语义（`/localization/state` 本来就说明
了是哪个模块的什么实体），改名只增加噪声，风险却是打断别人的 `tf2` 与 Nav2 配置。

> **ROS 消息/服务/动作生成的 C++ 类型**由包名直接决定：包名改成 `diesel_interfaces` 后，
> 生成类型就是 `diesel_interfaces::msg::X`。这部分是生成代码，不手写、不要套额外的 `diesel::` 前缀；
> 改包名后必须 `rm -rf build install log` 让它重新生成，否则 `install/` 里会残留旧包名的头文件。

---

## 4. 旧 → 新映射表

迁移时照抄。§4.1–4.3 是**代码内的字符串替换**，顺序自上而下（先长后短，避免 `localization` 先吃掉 `localization_map`）；
§4.4 是仓与包的重命名，顺序见 §6.3。

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

### 4.4 仓名与 ROS 包名

仓库名（GitHub 上重命名，旧 URL 会 302 重定向到新地址）：

| 旧仓名 | 新仓名 |
|---|---|
| `foray_platform` | `diesel_platform` |
| `foray_localization` | `diesel_localization` |
| `foray_interfaces` | `diesel_interfaces` |
| `foray_simulation` | `diesel_simulation` |
| `foray_navigation` | `diesel_navigation` |
| `foray_vision` | `diesel_vision` |
| `foray_auto_aim` | `diesel_auto_aim` |
| `foray_decision` | `diesel_decision` |

ROS 包名（现存 10 个包，只换前缀）：

| 旧包名 | 新包名 |
|---|---|
| `foray_localization_bringup` | `diesel_localization_bringup` |
| `foray_localization_core` | `diesel_localization_core` |
| `foray_localization_map` | `diesel_localization_map` |
| `foray_localization_relocalization` | `diesel_localization_relocalization` |
| `foray_localization_ros` | `diesel_localization_ros` |
| `foray_platform_core` | `diesel_platform_core` |
| `foray_platform_driver_hikvision` | `diesel_platform_driver_hikvision` |
| `foray_platform_driver_livox` | `diesel_platform_driver_livox` |
| `foray_rmul_2027_field` | `diesel_rmul_2027_field` |
| `foray_simulation_bringup` | `diesel_simulation_bringup` |

> 一个包名分布在**五个必须同步的位置**：`package.xml` 的 `<name>` 与 `<depend>`、`CMakeLists.txt` 的 `project()`、
> 目录名 `src/<pkg>/`、launch/py 里的 `package='<pkg>'`、`.repos` 的 pin（当前工作区没有 `.repos`，将来有就必须同改）。
> 缺一个就是 `colcon build` 找不到依赖。**改完必须 `rm -rf build install log`**——`install/` 按包名建目录，
> 不清掉会同时留着新旧两套头文件，编译器可能吃到旧那份。执行顺序见 §6.3。

### 4.5 迁移前先自查

```bash
# 待改项清单（结果必须为空才能把 mode 改成 enforced）
grep -rn 'namespace foray\|foray::' --include='*.hpp' --include='*.cpp' --include='*.h' --include='*.cc' .
grep -rn '#include [<"]foray/' --include='*.hpp' --include='*.cpp' --include='*.h' .
find . -type d -name foray -path '*/include/*'
grep -rn 'Node("foray_\|"foray_[a-z_]*"' --include='*.cpp' --include='*.py' .
# 包名与目录（这一组容易被忘）
grep -rn '<name>foray_' --include='package.xml' .
grep -rn 'project(foray_' --include='CMakeLists.txt' .
grep -rn "package=['\"]foray_" --include='*.py' --include='*.xml' --include='*.yaml' .
find . -maxdepth 3 -type d -name 'foray_*' -path '*/src/*'
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

> 表内用的是 2026-11-03 的现状仓名，对应新仓名见 §4.4（`foray_x` → `diesel_x`）。

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

| 仓（现 → 目标） | C++ 文件数 | 现状 | mode | 负责人待办 |
|---|---|---|---|---|
| `foray_platform` → `diesel_platform` | 172 | `foray::platform::…`（层级已对，只换顶层） | `migrating` | 仓改名 + 3 个包改名 + 顶层 namespace / include 根 / 节点名 |
| `foray_localization` → `diesel_localization` | 63 | `foray::localization_map/_ros/_relocalization`（**层级也要重排**） | `migrating` | 先合 PR #7/#8；仓改名 + 5 个包改名 + 按 §4 顺序改 |
| `foray_interfaces` → `diesel_interfaces` | 0 | 协议生成器，生成代码跟模板走 | `na` | 仓改名 + 改 `scripts/gen_lower_link.py` 模板后重新生成 |
| `foray_simulation` → `diesel_simulation` | 0 | 无 C++，2 个包（bringup / field） | `na`（仅对齐风格配置） | 仓改名 + 包改名 + 换 `.clang-format` / `pyproject.toml` |
| `foray_navigation` → `diesel_navigation` | 0 | 骨架 | `na` | 仓改名；建包时直接用 `diesel_*` |
| `foray_vision` / `foray_auto_aim` / `foray_decision` → `diesel_*` | 0 | 空仓 | `na` | 仓改名；首次提交即用 `diesel::` |
| `foray_docs` / `foray_ml` / `foray_robots` / `foray_ws` | — | 非上位机软件栈 | `na` | **不改名**（`foray` 是战队品牌） |

### 6.2 五步迁移法（单仓，namespace 部分）

> 本节只改 namespace / include 根 / 节点名；**包名与仓名见 §6.3**，包名必须在同一个 PR 内一起改，
> 否则 `colcon build` 会在中途编不过。

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

### 6.3 仓名与包名的改名顺序

**前提：先等 open PR 合完**（`foray_localization` #7 #8、`foray_platform` #2、`foray_navigation` #1、`foray_docs` #2）。
改名会重写大量文件路径，未合并的 PR 必然冲突，rebase 的成本高于等几天。

单仓顺序：**1–5 在同一个 PR 里完成，6–7 合并后做。**

1. **先记基线**：在容器里跑一次 `colcon build` + `colcon test`，确认改之前是绿的（否则改完分不清是谁的锅）。
2. **包名与目录**：`git mv src/<old_pkg> src/<new_pkg>`，改 `package.xml` 的 `<name>` / `<depend>`、
   `CMakeLists.txt` 的 `project()`、launch 里的 `package=`。
3. **namespace / include 根 / 节点名**：按 §4.1–4.3 替换（先长后短）。
4. **清干净重建**：`rm -rf build install log` 再 `colcon build`。
   包名变了，`install/` 下旧包的目录不会自动消失，会和新包同时存在。
5. **收紧门禁**：`.diesel-naming` 改成 `mode: enforced`，推 PR、等 CI 绿、合并。
6. **GitHub 仓改名**：`gh repo rename diesel_platform --repo ZQU-Foray/foray_platform`。
   旧 URL 会自动 302 到新地址，但本地 clone 要跟上：`git remote set-url origin https://github.com/ZQU-Foray/diesel_platform.git`，
   目录名建议同时 `mv foray_platform diesel_platform`（否则容器挂载与文档路径全是旧名）。
7. **同步仓外引用**：组织 `.github/CODEOWNERS`、`foray_docs` 里的仓库链接、`foray_robots` 的启动脚本与参数、
   以及 `foray_sim_run` 容器——它的 `/ws/src` 挂载路径是按仓名写死的，仓目录改名后要么更新启动脚本，要么重建容器。

> **为什么先改包名再改仓名**：仓改名只是 GitHub 侧的一次操作，随时能补，不影响编译；
> 而包名/目录不对齐，代码根本编不过。反过来做，会让每个仓在最长的窗口里处于编不过的状态。

---

## 7. 门禁与检查

组织 CI 模板 [`ci-template.yml`](https://github.com/ZQU-Foray/.github/blob/main/workflows/ci-template.yml) 新增两个作业：

| 作业 | 检查内容 | 失败条件 |
|---|---|---|
| `namespace-guard` | `namespace foray` / `foray::` / `#include "foray/` / 遗留 `include/foray/` 目录 / `Node("foray_*")` / `package.xml` 的 `<name>foray_*` / `CMakeLists.txt` 的 `project(foray_*` | 仅 `mode: enforced` 时失败；`migrating` 输出警告 |
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
- **包名与仓名按仓整体改**（§6.3），不要为了"顺手统一"单独改某一个包——装配链会断，别人拉下来就编不过。

---

## 9. 现状速查

```text
组织/品牌     ZQU-Foray · foray_docs / foray_ml / foray_robots   （不变）
系统名        Diesel                                             （投票结果）
仓库名        diesel_<domain>
ROS 包名      diesel_<domain>_<module>
C++ namespace diesel::<domain>::<module>
include 根    include/diesel/<domain>/…
ROS 节点名    diesel_<module>
协议 namespace diesel::<protocol>
话题          /<module>/<entity>                                 （不变）
tf frame      map / odom / base_link …                           （不变）
风格配置      .github/config/ 下的三个文件（唯一源）
```
