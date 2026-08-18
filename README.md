# ROS 2 + MoveIt 2 六自由度机械臂控制保姆级教程


本项目围绕“如何让一个 ROS 2 机械臂从模型描述走到可规划、可执行、可通信控制”的完整过程，搭建了六自由度机械臂与夹爪的 MoveIt 2 仿真控制系统。重点在于理解建模、状态发布、运动规划、轨迹执行和外部任务指令之间的连接关系。

URDF 与 Xacro 描述机械臂和夹爪的连杆、关节、运动范围、碰撞模型与安装关系；`robot_state_publisher` 根据机器人模型和关节状态发布 TF，RViz 据此展示当前姿态。MoveIt 负责基于模型、目标状态和碰撞约束规划可行轨迹；`ros2_control` 中的控制器负责接收并执行轨迹。

上层的 `Commander` 节点基于 MoveIt C++ API，封装了命名位姿、关节角、末端位姿、笛卡尔路径和夹爪开闭控制；其他 ROS 2 节点或终端只需发布任务目标即可调用这些能力。

通过本项目可以学习 ROS 2 功能包组织、URDF/Xacro 建模、TF、RViz、MoveIt 规划配置、`ros2_control`、`FakeSystem`、Launch 文件、ROS 2 话题、自定义消息和 MoveIt C++ API，并建立对机器人控制系统分层结构的整体认识。

## 功能范围

- 六自由度机械臂 URDF/Xacro 建模
- 夹爪建模与 MoveIt 配置
- RViz 和 TF 可视化
- MoveIt 运动规划
- `ros2_control` 控制器
- MoveIt C++ API Commander 节点
- 关节、位姿和夹爪控制话题
- 自定义消息接口

## 环境信息

- Ubuntu 22.04
- ROS 2 Humble
- MoveIt 2 Humble
- C++
- RViz 2
- `ros2_control`

## 快速开始


克隆并构建工作空间：

```bash
git clone https://github.com/XushuZou/ros2_moveit2_arm_control.git
cd ros2-moveit2-arm-control-tutorial/ros2_ws
colcon build
source install/setup.bash
```

### 测试命名位姿

在第一个终端启动机器人、MoveIt 和 RViz：

```bash
cd ros2-moveit2-arm-control-tutorial/ros2_ws
source install/setup.bash
ros2 launch my_robot_bringup my_robot.launch.xml
```

在第二个终端运行测试节点：

```bash
cd ros2-moveit2-arm-control-tutorial/ros2_ws
source install/setup.bash
ros2 run my_robot_commander_cpp test_moveit
```

两个命令需在两个终端中依次运行。执行第二个命令后，机械臂会自动运动到 `pose_1`，无需在 RViz 中拖拽。


### 运行 Commander 并通过话题控制

构建并运行 Commander：

```bash
cd ros2-moveit2-arm-control-tutorial/ros2_ws
colcon build --packages-select my_robot_commander_cpp
source install/setup.bash
ros2 run my_robot_commander_cpp commander_template
```

运行 Commander 前，需在另一个终端启动 bringup：

```bash
cd ros2-moveit2-arm-control-tutorial/ros2_ws
source install/setup.bash
ros2 launch my_robot_bringup my_robot.launch.xml
```

打开夹爪：

```bash
ros2 topic pub --once open_gripper example_interfaces/msg/Bool "{data: true}"
```

关闭夹爪：

```bash
ros2 topic pub --once open_gripper example_interfaces/msg/Bool "{data: false}"
```

运行节点后，可在第三个终端发送六个关节目标：

```bash
ros2 topic pub --once /joint_command \
example_interfaces/msg/Float64MultiArray \
"{data: [1.5, 0.5, 0.0, 1.5, 0.0, -0.7]}"
```
 
定义消息包必须先成功构建，控制节点才能包含和使用它。保持 bringup 和 Commander 分别在两个终端中运行后，可在第三个终端测试末端位姿控制：

```bash
ros2 topic pub --once /pose_command \
my_robot_interfaces/msg/PoseCommand \
"{x: 0.0, y: -0.7, z: 0.4, roll: 3.14, pitch: 0.0, yaw: 0.0, cartesian_path: false}"
```

其中 `cartesian_path: false` 使用普通 MoveIt 位姿规划；改为 `true` 时会调用笛卡尔路径规划。

