## 框架结构

![Flutter Framework](https://github.com/LiangLuDev/flutter_tutorial/blob/master/images/framework_basic.png?raw=true)

### 框架（Framework）

Dart语言编写。

####  UI层（Foundation、Animation、Painting、Gestures）

UI 层，提供动画、手势及绘制

####  渲染层（Rendering）

构建一个 Widget 树，当有变化时，会根据一定的算法计算出有变化的部分，然后更新 Widget 树。

#### 组件（Widgets、Materail、Cupertino）

基础组件库，Android风格和iOS风格组件

### 引擎（Engine）

C++语言编写。Android由NDK，iOS由LLVM，把Dart代码AOT/JIT模式编译为相应平台代码。


## 启动流程

![Flutter Activity](https://raw.githubusercontent.com/LiangLuDev/flutter_tutorial/master/images/flutter_activity.jpeg)

FlutterApplication：初始化配置文件/加载libflutter.so/注册JNI方法

MainActivity：应用主页面，继承 FlutterActivity

FlutterActivity：继承Activity，初始化 FlutterActivityDelegate 绑定生命周期

FlutterActivityDelegate：

- FlutterMain.ensureInitializationComplete
- create FlutterNativeView
- create FlutterView：SurfaceView
- setContentView : 把FlutterView加载到Activity 
- ......


相关文章：

[Flutter引擎启动](http://gityuan.com/2019/06/22/flutter_booting/)

[Flutter应用启动](http://gityuan.com/2019/06/29/flutter_run_app/)


## 渲染流程

VSync信号到来，状态发生变化，会走完整的渲染流水线动作：**动画（Animate）、构建（Build）、布局（Layout）和绘制（Paint）**，最终输出layer tree被送入engine，通过engine调度GPU绘制到屏幕上。

相关文章：

[Flutter渲染机制—GPU线程](http://gityuan.com/2019/06/16/flutter_gpu_draw/)

[Flutter渲染机制—UI线程](http://gityuan.com/2019/06/15/flutter_ui_draw/)
