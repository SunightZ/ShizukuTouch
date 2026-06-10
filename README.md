# 项目说明

**SunightTouch(ShizukuTouch)** 是一款针对 Android 平台开发的高级触控增强引擎。它利用 **Shizuku** 框架获取系统级权限，实现了对物理触控事件的底层截获、实时可视化渲染以及高精度的仿真触控注入。

---

本项目采用分层架构设计。

### 1. 核心层：Shizuku User Service
- **进程模型**：应用通过 `Shizuku.bindUserService` 将 `TouchUserService` 实例化在具有 Shell 权限的独立进程中。
- **权限边界**：该层直接与系统 `IInputManager` 交互，绕过应用层 `MotionEvent` 的分发限制，能够全局监听并注入触控事件。
- **协议适配**：内置对 Linux Multi-touch Protocol (Type A/B) 的识别逻辑。针对特殊设备（如采用 Protocol A 的华为/荣耀设备）设有安全熔断机制，防止底层注入导致触控死锁。

### 2. 注入引擎：双模驱动
项目实现了两种互补的触控注入方案：

*   **Native Mode (高性能内核级)**
    - **实现原理**：通过 JNI 调用 C++ 编写的 `libtouch.so`。
    - **技术细节**：直接操作 `/dev/uinput` 设备节点，在内核空间模拟一个虚拟的硬件输入设备。
    - **优势**：极低延迟（<1ms）、绕过所有应用层对虚拟按键的检测、支持完全的物理覆盖。
*   **Java Mode (高兼容性系统级)**
    - **实现原理**：利用反射获取 `InputManager` 实例。
    - **核心逻辑**：通过 `ShizukuBinderWrapper` 代理 `IInputManager` 接口，调用 `injectInputEvent` 方法。

### 3. 渲染层：可视化引擎
- **高性能绘制**：基于自定义 `PointerOverlayView` 实现。使用 `android.graphics.Canvas` 结合硬件加速。
- **数据流**：`TouchUserService` 通过 `IPhysicalTouchCallback` (AIDL) 异步回调触控状态，UI 进程接收 `PhysicalFingerState` 列表并触发重绘。
- **轨迹算法**：采用队列管理触控点轨迹，支持动态消失延迟（Fade Delay）与样条曲线平滑（Trails）。

---

## 功能特性

| 功能 | 技术实现描述 |
| :--- | :--- |
| **实时可视化** | 提供多种指针样式（Dot, Arrow, Iris）及霓虹、渐变等轨迹特效。 |
| **坐标监控** | 实时显示物理触控点的精确 X/Y 轴映射坐标。 |
| **自动点击器** | 独立 `ClickerService`，通过控制 `TouchUserService` 实现非 Root 环境下的坐标级自动化点击。 |
| **动态参数调节** | 运行时调整轨迹粗细、透明度、消失间隔（0-2000ms）及指针缩放比例。 |
| **多点追踪** | 支持最高 32 点同步追踪与仿真注入，完美适配多指操作场景。 |

---

## 使用流程

### 1. 环境准备
- Android 8.0+ 设备。
- 已激活的 **Shizuku** 环境。

### 2. 启动逻辑
1. **Shizuku 鉴权**：应用启动时检查 `pingBinder()`，并通过 `requestPermission` 获取用户授权。
2. **服务绑定**：调用 `bindService` 建立与 `TouchUserService` 的 AIDL 连接。
3. **悬浮窗授权**：引导用户开启 `ACTION_MANAGE_OVERLAY_PERMISSION`，用于 `OverlayService` 的视觉呈现。
4. **引擎激活**：点击主界面 Play 按钮，选择注入模式（推荐优先尝试 Native 模式以获取最佳体验）。

---


---

**项目作者**：Sunight  
**仓库地址**：[https://github.com/SunightZ/ShizukuTouch](https://github.com/SunightZ/ShizukuTouch)  
