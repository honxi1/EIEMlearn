# Endfield Poser 合并变更摘要

此次合并整体新增了一个独立的《明日方舟：终末地》游戏内摄影摆姿插件"Endfield Poser"项目，复用 EIEM 的注入链路（代理 DLL + Applepie 插件宿主 + IL2CPP 解析 + MinHook），实现角色冻结、FK/2-bone IK 摆姿、形态键调节、双模式（摆姿/镜头）自由相机与 ImGui 覆盖层 UI。所有文件均为全新添加，覆盖基础注入层、纯 C++ 数学模块（含可跨平台单测）、游戏逻辑层、编辑器界面层以及构建脚本与实现计划文档。

| 文件 | 变更 |
|------|---------|
| .gitignore | - 新增忽略规则，排除 build/ 目录与 plugin/poser.dll 构建产物 |
| CMakeLists.txt | - 新增 CMake 构建配置，Windows/MSVC 下编译 poser 插件 DLL<br>- 新增跨平台数学单测目标 test_quat/test_ik/test_pose_file 并启用 CTest |
| build.bat | - 新增 Windows 一键构建脚本，执行 CMake 配置/编译并拷贝 poser.dll 与代理 DLL 到 plugin/ 目录 |
| deps/json/nlohmann/json.hpp | - 引入 nlohmann/json 单头文件库，用于姿态文件 JSON 序列化 |
| docs/superpowers/plans/2026-08-26-endfield-poser.md | - 新增 Endfield Poser 完整实现计划文档，定义架构分层、范围决策与分阶段任务 |
| src/config.h | - 新增 poser_config.txt 配置文件读写，支持 GUI 切换/截图快捷键与相机速度等 key=value 解析 |
| src/core/base.h | - 新增基础工具：版本号宏、线程安全日志、IL2CPP 对象布局常量与字段偏移回退机制 |
| src/core/game_hooks.h | - 新增 MinHook 的 SetMainCharacter hook，角色切换时捕获 Animator/Entity<br>- 新增 Humanoid 55 根骨骼句柄封装（GetHumanoidBone/GetBoneLocalRot 等）供上层复用 |
| src/core/gui_overlay.h | - 新增 D3D11 + DComp 透明覆盖窗与 ImGui 渲染循环，业务面板通过 DrawPoserGui 挂接 |
| src/core/il2cpp_api.h | - 新增 IL2CPP 运行时解析（Resolve）与 FindClass/FindMethod/Invoke 等封装 |
| src/core/proxy_d3dcompiler.cpp | - 新增 d3dcompiler_47 代理 DLL，通过链接器导出转发系统 DLL 的 D3D 编译接口 |
| src/editor/gizmo.h | - 新增 ImGuizmo 3D 拖拽手柄集成，将选中骨世界矩阵换算回 localRotation/localPosition 写回 |
| src/editor/panel_camera.h | - 新增相机面板：接管主相机（禁用 CinemachineBrain）、FOV 滑条与机位预设记录/恢复 |
| src/editor/panel_library.h | - 新增姿态预设库面板，扫描 plugin/poses/*.poser.json 并支持加载、另存与删除姿态 |
| src/editor/panel_mode.h | - 新增摆姿模式/镜头模式双模式切换（Tab 键）与自由相机移动（WSAD/右键/滚轮） |
| src/editor/panel_morph.h | - 新增面部 BlendShape 形态键面板，按网格分组展示滑条，支持搜索过滤与恢复原始值 |
| src/editor/panel_pose.h | - 新增 FK 姿态编辑面板：按部位分组的骨骼树、三轴 Euler 滑条微调与骨骼锁定开关 |
| src/game/accessory.h | - 新增从骨（头发/配饰/衣角）采集：识别非 Humanoid 骨链并按连续链分组<br>- 新增逐链/逐骨物理组件禁用与锁定功能，锁定骨在快照恢复/FK/镜像中被跳过 |
| src/game/freeze.h | - 新增角色冻结系统：关闭 Animator 使骨骼停驻当前帧，固化姿势基线并抑制从骨物理等写者<br>- 新增解冻逻辑，恢复 Animator 与物理状态 |
| src/game/ik_driver.h | - 新增四肢链（左右臂/左右腿）的解析式 2-bone IK 驱动，支持目标由 gizmo 拖拽更新<br>- 预留驱动原生 BipedIK 的开关与接入点 |
| src/game/morph.h | - 新增 BlendShape 采集与读写：递归遍历角色根收集 SkinnedMeshRenderer 形态键，冻结态实时写入权重并支持解冻恢复 |
| src/game/skeleton.h | - 新增 Humanoid 骨骼列表维护、姿势快照捕获/恢复、T-pose 复位与姿态镜像操作 |
| src/math/ik_two_bone.h | - 新增纯 C++ 解析式 2-bone IK 求解器，含不可达/退化 pole 情况处理 |
| src/math/pose_file.h | - 新增纯 C++ 姿态文件模型与 JSON 序列化/反序列化（骨骼+形态键） |
| src/math/quat_math.h | - 新增纯 C++ Vec3/Quat 数学库：Slerp、Euler 换算、FromTo 方向旋转等 |
| src/poser.cpp | - 新增 DLL 入口与 Applepie 插件协议导出（Enable/Disable/Config/Hotkeys）<br>- 新增每帧 GameFrameTick 逻辑与主面板/姿态/姿态库/形态键/相机等 ImGui 窗口调度 |
| tests/test_ik.cpp | - 新增 2-bone IK 单元测试，覆盖目标命中、不可达伸直、pole 退化与驱动旋转差链路 |
| tests/test_pose_file.cpp | - 新增姿态文件 JSON 序列化往返单元测试 |
| tests/test_quat.cpp | - 新增四元数数学单元测试，覆盖轴角旋转、Slerp、FromTo 与 Euler 往返 |
