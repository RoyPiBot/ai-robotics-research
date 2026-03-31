# ROS 2 機械手臂進階指南

> 撰寫日期：2026-03-31
> 對象：已讀過入門指南，熟悉 ROS 2 基本概念（Node、Topic、Service）、會用 Python/C++ 寫簡單節點
> 前置要求：讀完 `ROS_robot_arm.md`（入門指南）

---

## 目錄

1. [進階運動學與動力學](#1-進階運動學與動力學)
2. [ros2_control 框架深入](#2-ros2_control-框架深入)
3. [MoveIt 2 進階應用](#3-moveit-2-進階應用)
4. [力控制與柔順控制](#4-力控制與柔順控制)
5. [視覺整合與手眼校正](#5-視覺整合與手眼校正)
6. [Sim-to-Real：從模擬到實機](#6-sim-to-real從模擬到實機)
7. [AI 驅動的機械手臂：VLA 與模仿學習](#7-ai-驅動的機械手臂vla-與模仿學習)
8. [進階實戰專案](#8-進階實戰專案)
9. [進階學習路線圖](#9-進階學習路線圖)
10. [參考資源](#10-參考資源)

---

## 1. 進階運動學與動力學

入門指南介紹了正向運動學（給角度 → 算位置），但實際開發中你更常面對的是反過來的問題。

### 1.1 逆向運動學（Inverse Kinematics, IK）

**問題**：我要機械手臂的末端移到 (x, y, z) 這個位置，每個關節應該轉幾度？

這比正向運動學難很多，因為：
- **可能無解** — 目標位置超出手臂可達範圍
- **可能有多組解** — 同一個末端位置，手臂姿態可以不同（想像你用手摸桌子，手肘可以朝上或朝下）
- **可能有無限多解** — 冗餘手臂（7+ DoF）有額外自由度

#### 常見 IK 求解方法

| 方法 | 原理 | 優點 | 缺點 | 適用場景 |
|------|------|------|------|----------|
| **解析解（Analytical）** | 用三角函數直接推導公式 | 速度極快、精確 | 只適用特定結構 | 6-DoF 標準工業手臂 |
| **Jacobian 迭代法** | 用 Jacobian 矩陣做數值逼近 | 通用性高 | 可能不收斂、奇異點問題 | 一般用途 |
| **TRAC-IK** | 結合 KDL 和 SQP 兩種求解器 | 成功率高、速度快 | 需安裝額外套件 | ROS 2 首選 IK 求解器 |
| **KDL** | 基於 Orocos KDL 函式庫 | ROS 內建 | 成功率較低 | 簡單場景 |
| **BioIK** | 基於進化演算法 | 處理約束能力強 | 速度較慢 | 高 DoF、多約束場景 |
| **IKFast** | OpenRAVE 離線生成解析解 | 速度極快 | 設定繁瑣 | 需要高頻 IK 的場景 |

#### Jacobian 矩陣 — 直覺理解

Jacobian 矩陣描述的是「關節速度」和「末端速度」之間的關係：

```
末端速度 = J(θ) × 關節速度
```

- J 是一個 6×n 的矩陣（6 = 3 個平移 + 3 個旋轉，n = 關節數）
- 知道 J 就能做速度控制：想讓末端沿直線移動，反推每個關節該轉多快
- **奇異點**：當 J 的行列式趨近 0 時，某些方向的運動會「卡住」——這是機械手臂控制最棘手的問題之一

#### 實作：用 MoveIt 2 做 IK

```python
# 使用 MoveIt 2 Python API (moveit_py)
from moveit.core.robot_state import RobotState
from moveit.core.kinematic_constraints import construct_joint_constraint

# 設定目標位姿
target_pose = Pose()
target_pose.position.x = 0.4
target_pose.position.y = 0.1
target_pose.position.z = 0.3
target_pose.orientation.w = 1.0

# 求解 IK
robot_state = RobotState(robot_model)
success = robot_state.set_from_ik(
    joint_model_group="arm",
    geometry_pose=target_pose,
    tip_name="end_effector_link",
    timeout=0.1  # 秒
)

if success:
    joint_values = robot_state.get_joint_group_positions("arm")
    print(f"IK 解: {joint_values}")
else:
    print("IK 無解！目標可能超出工作空間")
```

### 1.2 動力學（Dynamics）

運動學只管「位置和速度」，動力學還考慮「力和質量」：

- **正向動力學**：給定關節力矩 → 算出關節加速度（模擬用）
- **逆向動力學**：給定期望軌跡 → 算出需要的關節力矩（控制用）

#### 為什麼需要動力學？

1. **力矩前饋控制** — 計算重力補償，讓手臂在空中保持姿態不下垂
2. **碰撞偵測** — 比較預期力矩和實際力矩，差距過大 = 可能撞到東西
3. **力控制** — 精確控制末端施加的力（例如打磨、裝配）
4. **軌跡最佳化** — 考慮馬達極限，規劃最快或最省能的運動

#### 常見動力學演算法

| 演算法 | 用途 | 時間複雜度 |
|--------|------|------------|
| **RNEA**（遞迴 Newton-Euler） | 逆向動力學 | O(n) |
| **ABA**（Articulated Body Algorithm） | 正向動力學 | O(n) |
| **CRBA**（Composite Rigid Body Algorithm） | 質量矩陣計算 | O(n²) |

在 ROS 2 生態系中，**Pinocchio** 和 **KDL** 是最常用的動力學函式庫：

```python
# 使用 Pinocchio 做逆向動力學
import pinocchio as pin

# 載入 URDF 模型
model = pin.buildModelFromUrdf("robot.urdf")
data = model.createData()

# 給定 q（位置）、v（速度）、a（加速度）
# 計算所需力矩 τ
tau = pin.rnea(model, data, q, v, a)
print(f"所需力矩: {tau}")
```

### 1.3 工作空間分析

了解你的機械手臂「能到哪裡」和「到不了哪裡」至關重要：

- **可達工作空間（Reachable Workspace）**：末端至少能以一個姿態到達的所有點
- **靈巧工作空間（Dexterous Workspace）**：末端能以任意姿態到達的所有點（通常小很多）
- **奇異點邊界**：某些位置手臂雖然到得了，但會失去某些自由度

工具推薦：用 **MoveIt 2 + RViz** 可視覺化工作空間，或用 Python + Matplotlib 繪製 2D/3D 可達範圍圖。

---

## 2. ros2_control 框架深入

`ros2_control` 是 ROS 2 中控制真實硬體的核心框架。入門時你可能只用了簡單的 Topic 控制，但正式開發一定會用到它。

### 2.1 架構總覽

```
┌─────────────────────────────────────────────┐
│               ROS 2 應用層                   │
│  (MoveIt 2, Nav2, 你的應用程式)              │
└────────────────┬────────────────────────────┘
                 │ action/topic
┌────────────────▼────────────────────────────┐
│          Controller Manager                  │
│  ┌──────────┐ ┌──────────┐ ┌──────────────┐ │
│  │ Joint     │ │ Effort   │ │ Admittance   │ │
│  │ Trajectory│ │ Controller│ │ Controller   │ │
│  │ Controller│ │          │ │              │ │
│  └─────┬────┘ └────┬─────┘ └──────┬───────┘ │
└────────┼───────────┼──────────────┼─────────┘
         │ Hardware Interface (command/state)
┌────────▼───────────▼──────────────▼─────────┐
│          Hardware Abstraction Layer           │
│  ┌────────────┐  ┌────────────┐              │
│  │ System     │  │ Actuator   │              │
│  │ Interface  │  │ Interface  │              │
│  └─────┬──────┘  └─────┬──────┘              │
└────────┼───────────────┼────────────────────┘
         │               │
    ┌────▼────┐     ┌────▼────┐
    │ 真實硬體 │     │ 模擬器   │
    │ (馬達)   │     │ (Gazebo) │
    └─────────┘     └─────────┘
```

### 2.2 Hardware Interface：與硬體溝通

Hardware Interface 是你的程式和實體馬達之間的橋樑。你需要實作以下方法：

```cpp
// my_robot_hardware.hpp
#include "hardware_interface/system_interface.hpp"

class MyRobotHardware : public hardware_interface::SystemInterface
{
public:
  // 初始化：讀取 URDF 設定、開啟串口通訊
  CallbackReturn on_init(const HardwareInfo & info) override;

  // 啟動硬體（例如使能馬達驅動器）
  CallbackReturn on_activate(const rclcpp_lifecycle::State &) override;

  // 停止硬體（例如關閉馬達）
  CallbackReturn on_deactivate(const rclcpp_lifecycle::State &) override;

  // 從硬體讀取當前狀態（位置、速度、力矩）
  // 這個函式會以控制頻率被反覆呼叫（例如 500 Hz）
  return_type read(const rclcpp::Time &, const rclcpp::Duration &) override;

  // 把控制指令寫入硬體
  return_type write(const rclcpp::Time &, const rclcpp::Duration &) override;

  // 定義這個硬體提供的 state interfaces（可讀取的狀態）
  std::vector<StateInterface> export_state_interfaces() override;

  // 定義這個硬體提供的 command interfaces（可寫入的指令）
  std::vector<CommandInterface> export_command_interfaces() override;

private:
  // 硬體通訊（例如 serial port、CAN bus、EtherCAT）
  SerialPort serial_;

  // 關節狀態
  std::vector<double> hw_positions_;
  std::vector<double> hw_velocities_;
  std::vector<double> hw_efforts_;

  // 控制指令
  std::vector<double> hw_commands_;
};
```

### 2.3 可用的控制器（Controllers）

ros2_control 提供多種即用的控制器，依場景選用：

| 控制器 | 功能 | 典型用途 |
|--------|------|----------|
| **Joint Trajectory Controller** | 執行軌跡（一系列時間-位置點） | MoveIt 2 運動規劃輸出 |
| **Forward Command Controller** | 直接轉發原始指令到硬體 | 測試、簡單控制 |
| **Position Controllers** | 位置閉環控制 | 精確定位 |
| **Velocity Controllers** | 速度閉環控制 | 連續運動（如輸送帶） |
| **Effort Controllers** | 力矩/力控制 | 力控任務（打磨、裝配） |
| **Admittance Controller** | 導納控制（力→位移） | 人機協作、柔順操作 |
| **PID Controller** | 通用 PID 控制迴路 | 自訂控制策略基礎 |
| **Gripper Controller** | 夾爪開合控制 | 抓取任務 |
| **Joint State Broadcaster** | 發佈關節狀態到 `/joint_states` | 幾乎所有場景必備 |
| **Force Torque Sensor Broadcaster** | 發佈力/力矩感測器資料 | 力控場景 |

### 2.4 URDF 中的 ros2_control 設定

```xml
<ros2_control name="my_robot" type="system">
  <hardware>
    <plugin>my_robot_hardware/MyRobotHardware</plugin>
    <param name="serial_port">/dev/ttyUSB0</param>
    <param name="baud_rate">115200</param>
  </hardware>

  <!-- 第一個關節 -->
  <joint name="joint_1">
    <command_interface name="position">
      <param name="min">-3.14</param>
      <param name="max">3.14</param>
    </command_interface>
    <state_interface name="position"/>
    <state_interface name="velocity"/>
    <state_interface name="effort"/>
  </joint>

  <!-- 第二個關節 -->
  <joint name="joint_2">
    <command_interface name="position"/>
    <command_interface name="velocity"/>
    <state_interface name="position"/>
    <state_interface name="velocity"/>
  </joint>

  <!-- 更多關節... -->
</ros2_control>
```

### 2.5 即時性（Real-Time）考量

工業機器人控制迴路通常運行在 250Hz~1000Hz。要達到這個頻率：

1. **避免記憶體分配** — 在控制迴路中不要使用 `new`、`std::vector::push_back`
2. **使用 `realtime_tools`** — 提供 `RealtimePublisher`、`RealtimeBuffer` 等工具
3. **PREEMPT_RT 核心** — Linux 預設核心的排程器無法保證時限，需要 RT 核心補丁
4. **CPU 隔離** — 用 `isolcpus` 將一個核心專門給控制迴路使用

> **Pi 5 注意**：Raspberry Pi 可以跑 RT 核心，但效能和穩定性不如 x86 工業電腦。適合學習和原型開發，正式產品建議用工業等級硬體。

---

## 3. MoveIt 2 進階應用

MoveIt 2 是 ROS 2 的運動規劃框架，入門時你可能只用它拖動手臂到目標位置。進階用法包括：

### 3.1 運動規劃演算法

MoveIt 2 整合了 OMPL（Open Motion Planning Library），提供多種規劃演算法：

#### 取樣式規劃器（Sampling-based Planners）

| 演算法 | 特性 | 適用場景 |
|--------|------|----------|
| **RRT**（Rapidly-exploring Random Tree） | 快速探索空間 | 高維度、障礙多 |
| **RRT*** | RRT + 路徑最佳化 | 需要較短路徑 |
| **RRT-Connect** | 雙向 RRT，收斂快 | MoveIt 預設，一般用途 |
| **PRM**（Probabilistic Roadmap） | 預先建圖，多次查詢快 | 環境固定、重複規劃 |
| **BiTRRT** | 考慮成本函數 | 需要最佳化特定指標 |
| **LBKPIECE** | 適合窄通道場景 | 狹窄空間操作 |

#### 最佳化式規劃器

| 演算法 | 特性 | 適用場景 |
|--------|------|----------|
| **STOMP** | 隨機最佳化，生成平滑軌跡 | 需要自然動作 |
| **CHOMP** | 梯度下降最佳化 | 考慮平滑度和障礙 |
| **TrajOpt** | 凸最佳化 | 工業級軌跡最佳化 |
| **Pilz Industrial Motion Planner** | 線性/圓弧/PTP 動作 | 工業應用（如焊接路徑） |

#### 設定規劃器

```yaml
# moveit_config/config/ompl_planning.yaml
planner_configs:
  RRTConnect:
    type: geometric::RRTConnect
    range: 0.0  # 0 = 自動計算

  RRTstar:
    type: geometric::RRTstar
    range: 0.0
    goal_bias: 0.05
    delay_collision_checking: true

  PRMstar:
    type: geometric::PRMstar
    max_nearest_neighbors: 10

arm:
  default_planner_config: RRTConnect
  planner_configs:
    - RRTConnect
    - RRTstar
    - PRMstar
  projection_evaluator: joints(joint_1, joint_2)
  longest_valid_segment_fraction: 0.005  # 碰撞檢測精度
```

### 3.2 MoveIt Servo — 即時遙控操作

MoveIt Servo 讓你用搖桿、SpaceMouse 或任何輸入裝置即時控制手臂，就像玩遊戲一樣：

```yaml
# servo_config.yaml
moveit_servo:
  # 控制參數
  publish_period: 0.034  # 約 30 Hz
  incoming_command_timeout: 0.1

  # 運動模式
  command_in_type: "speed_units"  # 或 "unitless"
  scale:
    linear: 0.4   # 線速度縮放（m/s）
    rotational: 0.8  # 角速度縮放（rad/s）
    joint: 0.5     # 關節速度縮放

  # 安全設定
  check_collisions: true
  collision_check_rate: 10.0  # Hz
  self_collision_proximity_threshold: 0.01
  scene_collision_proximity_threshold: 0.02

  # 奇異點處理
  lower_singularity_threshold: 17.0
  hard_stop_singularity_threshold: 30.0
  leaving_singularity_threshold_multiplier: 2.0

  # 關節極限
  joint_limit_margins: [0.1, 0.1, 0.1, 0.1, 0.1, 0.1]
```

### 3.3 Pick and Place 流程

完整的抓取流程不只是「移過去 → 夾住 → 移回來」：

```
1. 物體偵測與定位（感知）
   └→ 相機拍照 → 物體識別 → 3D 位姿估計

2. 抓取姿態生成（Grasp Planning）
   └→ 根據物體形狀計算最佳抓取點和角度

3. 接近（Approach）
   └→ 沿抓取方向直線接近物體

4. 閉合夾爪（Grasp）
   └→ 用力控確認已經抓穩

5. 抬起（Lift）
   └→ 沿垂直方向抬起，確認物體沒掉

6. 移動到放置位置（Transport）
   └→ 規劃無碰撞路徑

7. 接近放置面（Approach Place）
   └→ 沿放置方向直線下降

8. 打開夾爪（Release）
   └→ 用力控確認物體已放穩

9. 退開（Retreat）
   └→ 沿安全方向退開
```

**MoveIt Task Constructor**（MTC）可以把這些步驟串成一個完整的 pipeline：

```cpp
// 使用 MoveIt Task Constructor 定義 Pick and Place
auto task = std::make_unique<moveit::task_constructor::Task>();

// Stage 1: 從當前狀態開始
auto current_state = std::make_unique<stages::CurrentState>("current");
task->add(std::move(current_state));

// Stage 2: 開啟夾爪
auto open_hand = std::make_unique<stages::MoveTo>("open hand", interpolation_planner);
open_hand->setGroup("hand");
open_hand->setGoal("open");
task->add(std::move(open_hand));

// Stage 3: 移動到抓取位置
auto move_to_pick = std::make_unique<stages::Connect>(
    "move to pick",
    stages::Connect::GroupPlannerVector{{"arm", sampling_planner}}
);
task->add(std::move(move_to_pick));

// Stage 4: 抓取（包含接近、夾取、抬起的子階段）
{
  auto grasp = std::make_unique<SerialContainer>("pick object");

  // 接近
  auto approach = std::make_unique<stages::MoveRelative>("approach", cartesian_planner);
  approach->setGroup("arm");
  approach->setDirection(/* 沿 z 軸下降 */);
  grasp->insert(std::move(approach));

  // 閉合夾爪
  auto close_hand = std::make_unique<stages::MoveTo>("close hand", interpolation_planner);
  close_hand->setGroup("hand");
  close_hand->setGoal("closed");
  grasp->insert(std::move(close_hand));

  // 抬起
  auto lift = std::make_unique<stages::MoveRelative>("lift", cartesian_planner);
  lift->setGroup("arm");
  lift->setDirection(/* 沿 z 軸上升 */);
  grasp->insert(std::move(lift));

  task->add(std::move(grasp));
}

// 執行
task->plan(10);  // 最多計畫 10 個方案
task->execute();  // 執行最佳方案
```

### 3.4 Cartesian Path Planning（笛卡爾路徑規劃）

讓末端沿直線或指定軌跡移動（如焊接、切割）：

```python
from moveit_msgs.msg import RobotTrajectory
from geometry_msgs.msg import Pose

# 定義路徑點（末端沿直線移動）
waypoints = []

# 第一段：向前移動 20cm
wpose = move_group.get_current_pose().pose
wpose.position.x += 0.2
waypoints.append(copy.deepcopy(wpose))

# 第二段：向下移動 10cm
wpose.position.z -= 0.1
waypoints.append(copy.deepcopy(wpose))

# 第三段：向右移動 15cm
wpose.position.y += 0.15
waypoints.append(copy.deepcopy(wpose))

# 規劃笛卡爾路徑
(plan, fraction) = move_group.compute_cartesian_path(
    waypoints,
    eef_step=0.01,      # 每步 1cm（越小越精確但越慢）
    jump_threshold=0.0,  # 0 = 不允許關節跳躍
    avoid_collisions=True
)

print(f"路徑完成度: {fraction * 100:.1f}%")

if fraction > 0.95:  # 至少 95% 的路徑可行
    move_group.execute(plan, wait=True)
```

---

## 4. 力控制與柔順控制

位置控制適合精確定位（如焊接），但有些任務需要控制「力」而非「位置」。

### 4.1 為什麼需要力控制？

| 場景 | 位置控制的問題 | 力控制的好處 |
|------|---------------|-------------|
| 打磨拋光 | 施力過大會損壞工件 | 保持恆定接觸力 |
| 裝配（插銷入孔） | 微小位置偏差會卡住 | 順著阻力方向調整 |
| 人機協作 | 碰到人時繼續推 → 危險 | 感受到外力就讓開 |
| 去毛邊 | 工件表面不規則 | 自動適應表面形狀 |

### 4.2 力控制策略

#### 阻抗控制（Impedance Control）

讓機械手臂像「彈簧阻尼系統」一樣運動：

```
F = M·ẍ + D·ẋ + K·(x - x_d)
```

- **K（剛度）**：越大 → 越硬，越精確追蹤目標位置
- **D（阻尼）**：越大 → 越不容易振盪
- **M（慣量）**：越大 → 反應越慢但越穩定

調整 K 和 D 就能讓手臂在「硬」（精確定位）和「軟」（柔順接觸）之間切換。

#### 導納控制（Admittance Control）

力 → 位移：感測到外力後，計算應該移動多少距離。

`ros2_control` 提供了內建的 **Admittance Controller**：

```yaml
# admittance_controller 設定
admittance_controller:
  ros__parameters:
    joints: [joint_1, joint_2, joint_3, joint_4, joint_5, joint_6]
    command_interfaces: [position]
    state_interfaces: [position, velocity]

    ft_sensor:
      name: tcp_fts_sensor
      frame:
        id: tool0

    # 導納參數
    admittance:
      mass: [10.0, 10.0, 10.0, 5.0, 5.0, 5.0]      # 虛擬質量
      damping: [200.0, 200.0, 200.0, 50.0, 50.0, 50.0]  # 阻尼
      stiffness: [500.0, 500.0, 500.0, 50.0, 50.0, 50.0]  # 剛度

    # 控制在哪些軸上做導納
    selected_axes: [true, true, true, true, true, true]
```

#### 混合力/位置控制

在不同軸上用不同策略：

- **X、Y 軸**：位置控制 → 沿表面移動
- **Z 軸**：力控制 → 保持恆定接觸力

這在打磨、擦拭等任務中非常實用。

### 4.3 力/力矩感測器

要做力控制，通常需要一個六軸力/力矩感測器（F/T Sensor），安裝在手臂末端和夾爪之間。

常見的力感測器品牌：
- **ATI Industrial Automation** — 業界標準，價格高
- **Robotiq FT 300-S** — 與 Universal Robots 搭配良好
- **OnRobot HEX** — 六軸，性價比高
- **自製方案** — 用應變片 + ADC，成本低但精度有限

在 ros2_control 中使用力感測器：

```xml
<!-- URDF 中定義力感測器 -->
<ros2_control name="ft_sensor" type="sensor">
  <hardware>
    <plugin>my_hardware/FTSensorInterface</plugin>
    <param name="serial_port">/dev/ttyUSB1</param>
  </hardware>
  <sensor name="tcp_fts_sensor">
    <state_interface name="force.x"/>
    <state_interface name="force.y"/>
    <state_interface name="force.z"/>
    <state_interface name="torque.x"/>
    <state_interface name="torque.y"/>
    <state_interface name="torque.z"/>
  </sensor>
</ros2_control>
```

---

## 5. 視覺整合與手眼校正

讓機械手臂「看到」並精確操作物體，需要相機和手臂之間的精確校正。

### 5.1 手眼校正（Hand-Eye Calibration）

兩種安裝方式：

| 方式 | 相機位置 | 優點 | 缺點 |
|------|---------|------|------|
| **Eye-in-Hand** | 裝在手臂末端 | 近距離高精度 | 視野隨手臂移動 |
| **Eye-to-Hand** | 固定在外部（如天花板） | 視野固定、穩定 | 遠距離精度較低 |

#### 校正原理

Eye-in-Hand 校正要解的方程式：

```
AX = XB
```

其中 A 和 B 分別是手臂和相機在不同姿態下的相對變換矩陣，X 是要求解的手眼變換。

#### 校正流程

1. 準備一個標定板（ArUco 板或棋盤格）放在固定位置
2. 移動手臂到 15~20 個不同姿態
3. 每個姿態記錄：
   - 手臂末端的位姿（從 robot state 讀取）
   - 相機看到的標定板位姿（從影像計算）
4. 用演算法求解 X

#### 使用 MoveIt 2 手眼校正

```bash
# 安裝 MoveIt 2 手眼校正套件
sudo apt install ros-jazzy-moveit-calibration

# 啟動校正工具（提供圖形化介面）
ros2 launch moveit_calibration_gui handeye_calibration.launch.py
```

### 5.2 物體偵測與位姿估計

抓取物體前需要知道物體在哪裡、朝哪個方向。常用方法：

| 方法 | 輸入 | 優點 | 缺點 | 工具 |
|------|------|------|------|------|
| **ArUco 標記** | 2D 相機 | 簡單可靠 | 物體上要貼標記 | OpenCV |
| **點雲匹配** | 深度相機 | 不需標記 | 計算量大 | PCL, Open3D |
| **深度學習偵測** | 2D/3D 相機 | 泛化能力強 | 需要訓練資料 | YOLO, DOPE |
| **FoundationPose** | RGBD | 無需訓練 | 需要 GPU | NVIDIA Isaac |
| **SAM + Depth** | RGBD | 通用物體分割 | 精度有限 | Segment Anything |

#### 範例：ArUco 標記偵測 + 位姿估計

```python
import cv2
import numpy as np

# 建立 ArUco 偵測器
aruco_dict = cv2.aruco.getPredefinedDictionary(cv2.aruco.DICT_6X6_250)
parameters = cv2.aruco.DetectorParameters()
detector = cv2.aruco.ArucoDetector(aruco_dict, parameters)

# 相機內參（從校正獲得）
camera_matrix = np.array([[fx, 0, cx],
                           [0, fy, cy],
                           [0,  0,  1]])
dist_coeffs = np.array([k1, k2, p1, p2, k3])

# 偵測與位姿估計
corners, ids, rejected = detector.detectMarkers(frame)
if ids is not None:
    rvecs, tvecs, _ = cv2.aruco.estimatePoseSingleMarkers(
        corners, marker_size, camera_matrix, dist_coeffs
    )
    # rvecs[i] = 旋轉向量, tvecs[i] = 平移向量
    # 這就是物體相對於相機的 6D 位姿
```

### 5.3 深度相機選擇

| 相機 | 技術 | 範圍 | 精度 | 價格 | 適用 |
|------|------|------|------|------|------|
| Intel RealSense D435i | 結構光 | 0.2-10m | ±2mm@2m | ~$300 | 一般抓取 |
| Intel RealSense L515 | LiDAR | 0.25-9m | ±5mm | ~$350 | 近距離高精度 |
| Orbbec Femto Bolt | ToF | 0.25-5.5m | ±2mm | ~$400 | 工業檢測 |
| Stereolabs ZED 2 | 雙目立體 | 0.3-20m | ±1mm@1.5m | ~$450 | 大範圍場景 |
| OAK-D Pro | 雙目+ToF | 0.15-10m | ±1.5mm | ~$250 | 高性價比 |

---

## 6. Sim-to-Real：從模擬到實機

在模擬器中訓練或測試，然後遷移到真實機器人。這是現代機器人開發的核心流程。

### 6.1 為什麼要用模擬？

1. **安全** — 不會撞壞真的機器人或環境
2. **快速** — 模擬可以加速到現實的 100~1000 倍
3. **大量資料** — 強化學習需要百萬次嘗試，真實世界做不到
4. **可重現** — 相同條件可以精確重複

### 6.2 模擬器比較（2026 年）

| 模擬器 | 物理引擎 | GPU 加速 | ROS 2 整合 | 適用場景 |
|--------|---------|---------|-----------|---------|
| **Gazebo Ionic** | 多引擎（DART, Bullet, TPE） | 部分 | 原生整合 | ROS 2 標準模擬 |
| **NVIDIA Isaac Sim** | PhysX 5 | 完整 | 透過 bridge | 大規模 RL 訓練 |
| **MuJoCo** | MuJoCo 引擎 | 完整 | 透過 plugin | 接觸模擬、RL |
| **PyBullet** | Bullet | 部分 | 需手動整合 | 快速原型 |
| **Webots** | ODE | 無 | 原生整合 | 教育、入門 |

> **Gazebo Kura**（2026 年 8 月預計發布）：下一代 Gazebo 版本，預期改善渲染品質和模擬速度。

### 6.3 Sim-to-Real Gap（模擬到現實的落差）

模擬永遠不完美，常見落差：

| 落差類型 | 原因 | 解決方法 |
|----------|------|----------|
| **物理參數** | 摩擦力、質量估計不準 | Domain Randomization |
| **感測器雜訊** | 模擬太乾淨 | 加入高斯雜訊 |
| **視覺差異** | 模擬渲染不夠逼真 | 隨機材質、光照 |
| **控制延遲** | 真實硬體有通訊延遲 | 模擬中加入隨機延遲 |
| **機構間隙** | 齒輪間隙、皮帶彈性 | 在模型中建模間隙 |

#### Domain Randomization 實作

在訓練時隨機化物理參數，讓模型學會在各種條件下都能工作：

```python
# 在 Isaac Sim 或 MuJoCo 中做 Domain Randomization
import numpy as np

def randomize_environment(env):
    # 隨機化摩擦係數
    env.set_friction(np.random.uniform(0.3, 1.2))

    # 隨機化物體質量 (±30%)
    nominal_mass = 0.5  # kg
    env.set_object_mass(nominal_mass * np.random.uniform(0.7, 1.3))

    # 隨機化相機位置 (±2cm)
    cam_noise = np.random.uniform(-0.02, 0.02, size=3)
    env.set_camera_offset(cam_noise)

    # 隨機化光照
    env.set_light_intensity(np.random.uniform(0.5, 1.5))
    env.set_light_direction(np.random.randn(3))

    # 隨機化控制延遲 (0-20ms)
    env.set_control_delay(np.random.uniform(0, 0.02))

    return env
```

### 6.4 NVIDIA Isaac Lab（原 Orbit）

目前最強大的機器人模擬訓練平台，特別適合大規模強化學習：

- 支援 **GPU 並行**：同時運行 4096+ 個環境
- 內建 **RL 訓練框架**：整合 RSL-RL, RL-Games, SKRL, Stable Baselines3
- 原生支援 **機械手臂操控任務**：抓取、推動、裝配
- 提供 **Domain Randomization** 模組

---

## 7. AI 驅動的機械手臂：VLA 與模仿學習

這是 2025-2026 年最熱門的領域——用 AI 讓機器人直接從示範中學習技能，而非手動編寫每個動作。

### 7.1 模仿學習（Imitation Learning）

**核心想法**：人類示範一次，機器人學會這個動作。

#### 傳統模仿學習 vs 現代方法

| 方法 | 年代 | 做法 | 優點 | 缺點 |
|------|------|------|------|------|
| 行為克隆（BC） | 經典 | 直接學 state → action 映射 | 簡單 | 分佈偏移嚴重 |
| DAgger | 2011 | 邊學邊讓專家修正 | 減少偏移 | 需要專家持續參與 |
| **ACT** | 2023 | Transformer + 分塊動作預測 | 平滑、精確 | 需要一定數量的示範 |
| **Diffusion Policy** | 2023 | 用擴散模型生成動作序列 | 處理多模態行為 | 推論速度較慢 |
| **VQ-BeT** | 2024 | VQ-VAE + BeT | 離散動作空間效果好 | 架構較複雜 |

#### ACT（Action Chunking with Transformers）

目前最實用的模仿學習方法之一：

- **Action Chunking**：一次預測未來 K 步動作（而非一步一步預測），避免不連貫
- **CVAE 架構**：用條件變分自編碼器處理行為的多樣性
- **只需 50~100 次示範** 就能學會簡單任務

#### Diffusion Policy

用 DDPM（去噪擴散概率模型）生成動作序列：

- 把「生成動作」當作「生成圖片」來做
- 能處理多模態行為（同一個狀態下可能有多種合理動作）
- 生成的動作序列比行為克隆更加平滑自然

### 7.2 Vision-Language-Action（VLA）模型

2025-2026 年的突破——讓機器人像理解語言一樣理解視覺和動作：

| 模型 | 開發者 | 特點 | 開源 |
|------|--------|------|------|
| **π0（pi-zero）** | Physical Intelligence | 基於 VLM + Flow Matching，通用機器人策略 | 部分 |
| **RT-2** | Google DeepMind | PaLM-E 為基礎，語言指令→動作 | 否 |
| **Octo** | UC Berkeley | 開源通用機器人策略，支援多機器人 | 是 |
| **OpenVLA** | Stanford | 開源 VLA，基於 Llama 2 + ViT | 是 |
| **SmolVLA** | Hugging Face | 輕量級 VLA，可在邊緣裝置運行 | 是 |
| **GR00T N1.5** | NVIDIA | 人形機器人專用 VLA | 部分 |

#### VLA 的工作原理

```
輸入：
  - 相機影像（RGB/RGBD）
  - 語言指令（如「把紅色杯子放到盤子上」）
  - 機器人當前狀態（關節角度、夾爪狀態）

                    ↓

  ┌─────────────────────────────┐
  │    Vision Encoder (ViT)     │ ← 理解視覺
  └──────────┬──────────────────┘
             │
  ┌──────────▼──────────────────┐
  │  Language Model (LLM/VLM)   │ ← 理解語言 + 推理
  └──────────┬──────────────────┘
             │
  ┌──────────▼──────────────────┐
  │    Action Head              │ ← 生成動作
  │  (MLP / Flow Matching /     │
  │   Diffusion Decoder)        │
  └──────────┬──────────────────┘
             │
輸出：
  - 未來 K 步的關節角度/末端位移
  - 夾爪開合指令
```

### 7.3 LeRobot 生態系統

Hugging Face 的 LeRobot 是目前最完整的開源機器人學習框架（ICLR 2026 接受）：

#### 支援的硬體

- **SO-100 / SO-ARM100**：低成本 6-DoF 手臂（~$300）
- **Koch v1.1**：開源手臂設計
- **Aloha**：雙臂遠端操作系統
- **Reachy2**：Pollen Robotics 人形機器人

#### 支援的模型

```python
# 使用 LeRobot 訓練 ACT 策略
# 安裝：pip install lerobot

# 1. 收集示範資料（用遠端操作器）
lerobot-record \
  --robot-type so100 \
  --dataset-repo-id my_user/my_task \
  --num-episodes 50

# 2. 訓練模型
lerobot-train \
  --policy.type=act \
  --dataset.repo_id=my_user/my_task \
  --output_dir=outputs/my_act_model

# 3. 在真實機器人上執行
lerobot-eval \
  --policy.path=outputs/my_act_model \
  --robot-type so100
```

#### SmolVLA — 在 Pi 上跑 VLA？

SmolVLA 是 Hugging Face 開發的輕量級 VLA 模型：
- 基於 SmolVLM（小型視覺語言模型）
- 參數量遠小於 OpenVLA / π0
- **目標**：在邊緣裝置（如 Jetson、甚至 Pi）上運行
- 目前仍在發展中，但代表了 VLA 輕量化的趨勢

### 7.4 強化學習（RL）用於機械手臂

模仿學習從示範學，強化學習從試錯學：

| 演算法 | 類型 | 適用場景 | 框架 |
|--------|------|----------|------|
| **PPO** | On-policy | 通用，穩定 | Stable Baselines3 |
| **SAC** | Off-policy | 連續動作空間 | 自訂 |
| **DDPG / TD3** | Off-policy | 連續控制 | 自訂 |
| **HIL-SERL** | 混合 | Human-in-the-loop | LeRobot |
| **TDMPC** | Model-based | 樣本效率高 | LeRobot |

> **實務建議**：先用模仿學習得到一個「差不多」的策略，再用強化學習微調，效率遠高於從零開始 RL。

---

## 8. 進階實戰專案

### 專案 1：桌面物品整理機器人

**難度**：★★★☆☆
**所需硬體**：6-DoF 手臂 + RGBD 相機 + 夾爪
**學習重點**：MoveIt 2 + 物體偵測 + Pick-and-Place

```
架構：
  USB Camera → YOLO 物體偵測 → 物體位姿估計
                                      ↓
  ros2_control ← MoveIt 2 軌跡規劃 ← 抓取規劃
       ↓
  真實手臂執行抓取 → 放到指定位置
```

### 專案 2：力控裝配系統

**難度**：★★★★☆
**所需硬體**：6-DoF 手臂 + F/T 感測器
**學習重點**：力控制 + 導納控制 + 策略切換

```
任務：把圓柱插入孔中（peg-in-hole）
  1. 位置控制：移動到孔的正上方
  2. 混合控制：Z 軸力控下壓 + XY 軸阻抗控制
  3. 螺旋搜索：如果插不進去，做微小圓周運動尋找孔位
  4. 插入：感測到 Z 軸力突然減小 = 對齊了，加大下壓力
  5. 確認：到達目標深度，任務完成
```

### 專案 3：模仿學習抓取

**難度**：★★★★★
**所需硬體**：SO-100 手臂 + 攝影機 + 遠端操作裝置
**學習重點**：LeRobot + ACT/Diffusion Policy + 資料收集

```
流程：
  1. 組裝 SO-100（3D 列印 + 伺服馬達）
  2. 用 LeRobot 的遠端操作介面收集 50~100 次抓取示範
  3. 訓練 ACT 策略
  4. 在真實手臂上測試
  5. 失敗案例 → 收集更多示範 → 重新訓練
```

### 專案 4：多臂協作

**難度**：★★★★★
**所需硬體**：2× 機械手臂
**學習重點**：多機器人協調、共享工作空間避碰

```
挑戰：
  - 兩隻手臂的工作空間重疊，需要互相避讓
  - 協作任務（如兩隻手一起搬大物體）需要同步
  - 可用 MoveIt 2 的 multi-robot planning 功能
```

---

## 9. 進階學習路線圖

### 階段 1：鞏固基礎（2~4 週）

```
目標：從「會用」到「理解原理」

□ 手寫正向/逆向運動學（2-DOF 或 3-DOF 平面手臂）
□ 理解 DH 參數（Denavit-Hartenberg）
□ 用 Python + Matplotlib 繪製工作空間
□ 閱讀 Craig《Introduction to Robotics》第 1-4 章
```

### 階段 2：ros2_control 實戰（3~4 週）

```
目標：能自己寫 Hardware Interface 和設定 Controller

□ 寫一個簡單的 Hardware Interface（控制 Arduino 上的伺服馬達）
□ 設定 Joint Trajectory Controller
□ 設定 Joint State Broadcaster
□ 理解 lifecycle node 和 controller manager
□ 嘗試不同的控制器（position → velocity → effort）
```

### 階段 3：MoveIt 2 深入（3~4 週）

```
目標：能設定和使用 MoveIt 2 的進階功能

□ 用 MoveIt Setup Assistant 設定自己的機器人
□ 嘗試不同規劃演算法（RRT-Connect, RRT*, PRM）
□ 實作 Pick and Place（用 MoveIt Task Constructor）
□ 實作 Cartesian Path Planning
□ 設定碰撞物體和規劃場景
```

### 階段 4：感知與視覺（4~6 週）

```
目標：讓機械手臂「看到」世界

□ 相機校正（內參 + 外參）
□ 手眼校正（Eye-in-Hand 或 Eye-to-Hand）
□ ArUco 標記偵測
□ 用 YOLO / SAM 做物體偵測
□ 點雲處理（PCL 或 Open3D）
□ 整合到 MoveIt 2 的 Planning Scene
```

### 階段 5：力控制（3~4 週）

```
目標：讓機械手臂「感受」力量

□ 理解阻抗控制和導納控制的理論
□ 設定 ros2_control 的 Admittance Controller
□ 用 F/T 感測器做簡單力控實驗
□ 實作 peg-in-hole 裝配任務
```

### 階段 6：AI + 機器人（持續學習）

```
目標：用最新 AI 技術增強機器人能力

□ 安裝和使用 LeRobot 框架
□ 收集示範資料、訓練 ACT 策略
□ 了解 Diffusion Policy 原理
□ 閱讀 VLA 論文（π0, OpenVLA, SmolVLA）
□ 嘗試 Sim-to-Real 遷移（Isaac Sim → 真實手臂）
□ 關注 Hugging Face 機器人學習課程（免費）
```

---

## 10. 參考資源

### 教科書

| 書名 | 作者 | 難度 | 涵蓋主題 |
|------|------|------|----------|
| *Introduction to Robotics* | John J. Craig | ★★★ | 運動學、動力學、控制 |
| *Robotics: Modelling, Planning and Control* | Siciliano et al. | ★★★★ | 全面涵蓋，偏理論 |
| *Modern Robotics* | Lynch & Park | ★★★★ | 現代方法，有免費線上版 |
| *A Mathematical Introduction to Robotic Manipulation* | Murray, Li, Sastry | ★★★★★ | 數學嚴謹，經典 |

### 線上課程

- **Modern Robotics（Northwestern）** — [Coursera](https://www.coursera.org/specializations/modernrobotics)，配合教科書免費觀看
- **Robotic Manipulation（MIT 6.881）** — Russ Tedrake 的開放課程，涵蓋抓取、力控、規劃
- **Hugging Face 機器人學習課程** — 免費，涵蓋 LeRobot、模仿學習、VLA
- **Stanford CS 237B** — 機器人原理 II，進階控制和規劃

### 重要論文（2024-2026）

| 論文 | 主題 | 重要性 |
|------|------|--------|
| π0 (Physical Intelligence, 2024) | 通用機器人 VLA 策略 | 定義了 VLA 方向 |
| Diffusion Policy (Chi et al., 2023) | 擴散模型做動作生成 | 模仿學習新範式 |
| ACT (Zhao et al., 2023) | Action Chunking + Transformer | 實用模仿學習 |
| OpenVLA (Kim et al., 2024) | 開源 VLA 基準 | 開源 VLA 標竿 |
| SmolVLA (Hugging Face, 2025) | 輕量級 VLA | 邊緣部署 VLA |
| LeRobot (Cadene et al., 2024) | 開源機器人學習框架 | ICLR 2026 接受 |
| Mobile ALOHA (Fu et al., 2024) | 低成本雙臂遠端操作 | 降低硬體門檻 |
| GR00T N1.5 (NVIDIA, 2025) | 人形機器人 VLA | 人形機器人專用 |

### 開源專案

| 專案 | 用途 | 連結 |
|------|------|------|
| LeRobot | 機器人學習框架 | github.com/huggingface/lerobot |
| MoveIt 2 | 運動規劃 | moveit.ai |
| ros2_control | 硬體控制框架 | control.ros.org |
| Pinocchio | 動力學計算 | github.com/stack-of-tasks/pinocchio |
| Open3D | 3D 視覺 | open3d.org |
| Isaac Lab | GPU 加速模擬 | github.com/isaac-sim/IsaacLab |
| SO-ARM100 | 低成本開源手臂 | github.com/TheRobotStudio/SO-ARM100 |

### 社群

- **ROS Discourse** — discourse.ros.org（官方論壇）
- **Hugging Face Discord** — #robotics 頻道
- **Robotics Stack Exchange** — robotics.stackexchange.com
- **r/robotics** — Reddit 機器人社群
- **ROSCon** — 年度 ROS 開發者大會

---

> 💡 **本文件與入門指南的關係**：入門指南（`ROS_robot_arm.md`）教你「如何開始」，本文件教你「如何深入」。建議先完成入門指南的階段一到三，再進入本文件的內容。

> 📅 最後更新：2026-03-31
