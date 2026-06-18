---

# 🐾 Legged Robot SLAM & Navigation (Unitree A1)

本项目基于廖教授的四足机器人框架，实现了 Unitree A1 机器狗在 Gazebo 仿真环境下的 **SLAM地图构建** 与 **自主路径规划（Navigation）**。

---

## 🛠️ 环境准备与编译规范

在运行任何节点前，请确保工作空间已正确编译。

> ⚠️ **编译避坑指南**：本项目工作空间由 `catkin build` 进行管理，**严禁混用 `catkin_make**`，否则会导致动态库冲突并引发 RViz `Segmentation fault` 崩溃。

```bash
cd ~/legged_ws
# 清理可能存在的冲突缓存
catkin clean --yes
# 使用原生工具链完整编译
catkin build -j2 -p2 --cmake-args -DCMAKE_BUILD_TYPE=Release
# 刷新环境变量
source devel/setup.bash

```

---

## 🗺️ 第一阶段：SLAM 建图与扫描

请严格按照以下步骤依次在**不同的终端**中执行：

### 1. 启动仿真环境、算法与 RViz

```bash
cd ~/legged_ws
source devel/setup.bash
export ROBOT_TYPE=a1
roslaunch legged_nav legged_slam.launch

```

### 2. 加载控制器（输入 list 查看步幅）

```bash
cd ~/legged_ws
source devel/setup.bash
rosparam set /use_sim_time true
export ROBOT_TYPE=a1
roslaunch legged_controllers load_controller.launch cheater:=false

```

### 3. 激活电机控制器

```bash
rosrun rqt_controller_manager rqt_controller_manager

```

> 💡 *操作提示：在弹出的 GUI 界面中，右键将对应的控制器集群状态切换为 `Start`（变为绿色亮起）。*

### 4. 启动键盘遥控（操纵狗子扫描地图）

```bash
source ~/legged_ws/devel/setup.bash
rosrun teleop_twist_keyboard teleop_twist_keyboard.py

```

> 🎮 *控制提示：使用键盘的 `i`, `j`, `k`, `l` 键控制机器狗前后左右移动，使其在环境中走动以完成全景雷达扫描。*

### 5. 保存建好的地图

当 RViz 里的地图边界清晰完整后，运行以下命令固化地图：

```bash
source ~/legged_ws/devel/setup.bash
rosrun map_server map_saver -f ~/legged_ws/src/legged_nav/maps/my_map

```

---

## 🚀 第二阶段：自主路径规划与导航

在确保第一阶段的地图已成功保存后，关闭之前的所有终端，重新开始导航流程：

### 1. 启动导航后台算法（不加载默认 RViz）

```bash
cd ~/legged_ws
source devel/setup.bash
export ROBOT_TYPE=a1
roslaunch legged_nav legged_navigation.launch use_rviz:=false

```

### 2. 加载控制器

```bash
cd ~/legged_ws
source devel/setup.bash
rosparam set /use_sim_time true
export ROBOT_TYPE=a1
roslaunch legged_controllers load_controller.launch cheater:=false

```

### 3. 激活电机控制器

```bash
rosrun rqt_controller_manager rqt_controller_manager

```

### 4. 打开定制导航 RViz 界面

```bash
source ~/legged_ws/devel/setup.bash
rosrun rviz rviz -d $(rospack find legged_nav)/rviz/nav.rviz

```

---

## 🎮 导航操控指南

1. 打开 RViz 界面后，如果机器狗在地图上的初始位置有偏置，点击上方工具栏的 **`2D Pose Estimate`** 按钮，在地图上点击并拖动，为机器狗赋予正确的初始姿态。
2. 确认位姿匹配后，点击上方工具栏的 **`2D Nav Goal`**。
3. 在目标点点击并拖动一条代表朝向的箭头，机器狗将自动规划出一条全局绿色的最优路径，并自主起步移动前往目的地。
