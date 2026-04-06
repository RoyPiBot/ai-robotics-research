# ROS 與機械手臂入門指南

> 撰寫日期：2026-04-02（第五次更新）
> 對象：完全新手，從零開始
> 環境：Raspberry Pi 5 (16GB RAM)

---

## 目錄

1. [ROS 是什麼](#1-ros-是什麼)
2. [機械手臂基本原理](#2-機械手臂基本原理)
3. [如何在樹莓派上安裝 ROS 2](#3-如何在樹莓派上安裝-ros-2)
4. [簡單範例程式](#4-簡單範例程式)
5. [推薦學習路線](#5-推薦學習路線)

---

## 1. ROS 是什麼

### 一句話解釋

ROS（Robot Operating System，機器人作業系統）是一套讓你輕鬆開發機器人的**軟體框架**。雖然名字裡有「作業系統」，但它其實不是 Windows 或 Linux 那種作業系統，而是跑在 Linux **之上**的一組工具和函式庫。

### 用比喻來理解

想像你在蓋一棟房子：

- **沒有 ROS**：你得自己燒磚頭、自己拉電線、自己做水管。每個零件之間要怎麼連接，全都要你從頭設計。
- **有了 ROS**：就像有了標準規格的建材和水電規範。插座是統一尺寸的，水管接頭是標準的。你只要專心設計房子長什麼樣，不用煩惱底層怎麼接。

再換一個比喻：ROS 就像機器人世界的「手機 App Store」。有人做好了導航的 App、有人做好了影像辨識的 App、有人做好了手臂控制的 App。你要組合出自己的機器人，只要把需要的 App 裝起來，讓它們互相溝通就好。

### ROS 的核心概念

| 概念 | 比喻 | 說明 |
|------|------|------|
| **Node（節點）** | 一個工人 | 每個 Node 負責一件事，例如讀取感測器、控制馬達 |
| **Topic（話題）** | 公佈欄 | Node 之間透過 Topic 傳遞訊息，就像在公佈欄上貼公告 |
| **Publisher** | 貼公告的人 | 把資料發佈到 Topic 上 |
| **Subscriber** | 看公告的人 | 從 Topic 讀取資料 |
| **Service** | 問答服務台 | 一問一答的互動，例如「現在溫度幾度？」→「25 度」 |
| **Package（套件）** | 一個工具箱 | 把相關的程式碼打包在一起，方便分享和重複使用 |

### 為什麼需要 ROS？

1. **不用重新造輪子** — 全球數萬名開發者貢獻了大量現成的套件
2. **標準化通訊** — 不管你用什麼感測器、什麼馬達，都用同一套方式溝通
3. **模擬環境** — 用 Gazebo 模擬器就能在電腦上測試，不怕弄壞真正的機器人
4. **社群龐大** — 遇到問題很容易找到解答

### ROS 1 vs ROS 2

目前建議直接學 **ROS 2**。ROS 1 已經不再積極開發，而 ROS 2 改善了很多架構問題，包括即時性、安全性和多平台支持。

### ROS 2 版本現況（2026 年 3 月更新）

| 版本 | 代號 | 發布日期 | 支援至 | 狀態 |
|------|------|----------|--------|------|
| ROS 2 Jazzy | Jazzy Jalisco | 2024 年 5 月 | 2029 年 5 月 | **LTS 穩定版** |
| ROS 2 Kilted | Kilted Kaiju | 2025 年 5 月 | 2026 年 11 月 | **目前最新版** |
| ROS 2 Lyrical | Lyrical Luth | 2026 年 5 月（預計） | 2027 年 11 月（預計） | 開發中 |

**Kilted Kaiju 重要新特性**：
- **Zenoh 成為 Tier 1 中介層**：首個將 Eclipse Zenoh 列為 Tier 1 的版本，效能大幅提升
- **Python Events Executor**：從 C++ 移植，效能提升最高 **10 倍**
- 2026 年 3 月更新：新增 27 個套件、更新 304 個現有套件

**Lyrical Luth 已知新功能**（距離發布約 2 個月）：
- **外掛系統改進**：支援將參數傳遞至建構子（constructor arguments），外掛不再需要預設建構子
- **Python API 變更**：將 Python `set` 物件傳入陣列/序列欄位已被棄用，改用 `list`、`tuple` 或 `numpy.ndarray`
- **Image Transport**：`point_cloud_transport` 支援 lifecycle nodes；新增 NV12 影像格式
- **Windows 11 支援**：計畫提升 Windows 平台體驗
- [官方發行頁面](https://docs.ros.org/en/rolling/Releases/Release-Lyrical-Luth.html)

**新手建議**：從 **Jazzy Jalisco（LTS）** 開始學最穩定；想用最新功能可選 **Kilted Kaiju**。Lyrical 預計 5 月發布，屆時可評估是否升級。

---

## 2. 機械手臂基本原理

### 什麼是機械手臂？

機械手臂就是模仿人類手臂動作的機械裝置。想像你的手臂：肩膀可以轉、手肘可以彎、手腕可以扭——機械手臂也是用類似的方式，透過一個個「關節」來完成各種動作。

### 自由度（Degrees of Freedom, DoF）

**自由度**就是機械手臂能獨立運動的方向數量。

用生活例子來理解：
- **1 個自由度**：像門——只能開和關（繞一個軸轉動）
- **2 個自由度**：像搖頭娃娃——能上下點頭，也能左右搖頭
- **3 個自由度**：能在三維空間中移動到任意位置（上下、左右、前後）
- **6 個自由度**：能移動到任意位置，**還能**用任意角度面對那個位置（完整控制位置 + 方向）

常見配置：

| 自由度 | 能做什麼 | 適合的任務 |
|--------|----------|------------|
| 3 DoF | 到達空間中的任意點 | 簡單的取放 |
| 4 DoF | 到達任意點 + 旋轉末端 | 基本組裝 |
| 5 DoF | 接近完整的靈活度 | 大部分教育用途 |
| 6 DoF | 完整的位置和方向控制 | 工業焊接、噴漆 |
| 7 DoF | 有冗餘，能繞過障礙物 | 醫療、複雜環境 |

**新手建議**：從 3~4 自由度的手臂開始就夠了！不需要一開始就追求 6 軸。

### 關節類型

機械手臂主要有兩種關節：

1. **旋轉關節（Revolute Joint）** — 像門鉸鏈一樣轉動，這是最常見的類型
2. **滑動關節（Prismatic Joint）** — 像抽屜一樣直線滑動

絕大多數你會接觸到的機械手臂都是用旋轉關節。

### 末端執行器（End Effector）

末端執行器就是機械手臂「手」的部分，裝在最後一節。常見的有：

- **夾爪（Gripper）** — 像筷子夾東西，最常見
- **吸盤（Suction Cup）** — 用真空吸住物體
- **工具頭** — 焊接槍、噴漆器等等

### 運動學基礎（別怕，很簡單）

運動學聽起來很嚇人，但概念其實很直覺：

#### 正向運動學（Forward Kinematics）

> 「我知道每個關節轉了多少度，請告訴我手臂末端在哪裡。」

就像你彎曲手臂的每個關節到特定角度，然後看你的手指到了哪個位置。這個計算相對簡單。

#### 逆向運動學（Inverse Kinematics）

> 「我知道我要讓手臂末端到哪個位置，請告訴我每個關節該轉多少度。」

這就像有人指著桌上一個杯子說「去拿那個」，你的大腦自動計算肩膀、手肘、手腕該怎麼動。這個計算比較複雜，但好消息是——**ROS 裡的 MoveIt 套件會幫你算**，你不需要自己推數學公式。

### 機械手臂的座標系

機械手臂通常使用兩種座標：

- **關節空間（Joint Space）** — 用每個關節的角度來描述姿態，例如「關節1轉30度、關節2轉45度...」
- **笛卡爾空間（Cartesian Space）** — 用 x, y, z 位置和方向來描述末端位置，例如「移到 (0.3, 0.1, 0.5) 公尺的位置」

---

## 3. 如何在樹莓派上安裝 ROS 2

### 前置知識

- 你需要會基本的 Linux 終端機操作（cd、ls、sudo 這些指令）
- 如果完全不熟 Linux，建議先花一兩天學基本指令

### 方案選擇

在 Raspberry Pi 5 上有兩種方式安裝 ROS 2：

| 方案 | 優點 | 缺點 |
|------|------|------|
| **方案 A：安裝 Ubuntu 24.04 + ROS 2 Jazzy** | 官方支援、最穩定 | 需要重灌系統為 Ubuntu |
| **方案 B：用 Docker 在 Pi OS 上跑 ROS 2** | 不用換系統 | 設定稍複雜、佔約 3.3GB |

**建議新手用方案 A**，比較不會遇到奇怪的問題。

---

### 方案 A：Ubuntu 24.04 + ROS 2 Jazzy（推薦）

#### 步驟 1：準備 SD 卡

1. 在你的電腦上下載並安裝 [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2. 插入 SD 卡（建議至少 **32GB**，64GB 更好）
3. 在 Imager 中選擇：
   - 裝置：Raspberry Pi 5
   - 作業系統：**Ubuntu Desktop 24.04 LTS (64-bit)**
   - 儲存裝置：你的 SD 卡
4. 點擊「設定」，可以預先設定 Wi-Fi、使用者名稱等
5. 寫入 SD 卡

#### 步驟 2：開機與初始設定

```bash
# 插入 SD 卡到 Pi 5，接上螢幕鍵盤，開機
# 完成 Ubuntu 初始設定精靈
# 開啟終端機，先更新系統
sudo apt update && sudo apt upgrade -y
```

#### 步驟 3：設定語言環境

```bash
sudo apt install locales -y
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
```

#### 步驟 4：加入 ROS 2 套件庫

```bash
# 安裝必要工具
sudo apt install software-properties-common curl -y

# 加入 ROS 2 的 GPG 金鑰
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
  -o /usr/share/keyrings/ros-archive-keyring.gpg

# 加入 ROS 2 套件庫
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
  http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | \
  sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

# 更新套件列表
sudo apt update
```

#### 步驟 5：安裝 ROS 2 Jazzy

```bash
# 安裝桌面版（含 RViz 視覺化工具和範例程式）
sudo apt install ros-jazzy-desktop -y

# 安裝開發工具
sudo apt install python3-colcon-common-extensions -y
sudo apt install ros-dev-tools -y
```

> 這一步會花比較久的時間（可能 15~30 分鐘），喝杯咖啡等一下。

#### 步驟 6：設定環境變數

```bash
# 加入 bashrc，這樣每次開終端機都會自動載入 ROS 2
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

#### 步驟 7：驗證安裝

開兩個終端機視窗：

```bash
# 終端機 1：啟動一個 talker（發送者）
ros2 run demo_nodes_cpp talker
```

```bash
# 終端機 2：啟動一個 listener（接收者）
ros2 run demo_nodes_cpp listener
```

如果你看到 listener 不斷收到 talker 發出的 "Hello World" 訊息，恭喜你，ROS 2 安裝成功了！

---

### 方案 B：Docker 方式（保留 Pi OS）

如果你不想重灌系統，可以用 Docker：

```bash
# 安裝 Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
# 登出再登入讓群組生效

# 拉取 ROS 2 Jazzy 映像
docker pull ros:jazzy

# 啟動 ROS 2 容器
docker run -it --rm \
  --name ros2_jazzy \
  --network host \
  ros:jazzy \
  bash

# 進入容器後就能使用 ROS 2 了
source /opt/ros/jazzy/setup.bash
ros2 run demo_nodes_cpp talker
```

> 注意：Docker 方式在使用 GUI 工具（如 RViz）時需要額外設定 X11 轉發，比較麻煩。

---

## 4. 簡單範例程式

### 範例一：用 ROS 2 建立一個簡單的關節控制節點

這個範例展示如何用 Python 寫一個 ROS 2 節點，模擬控制一個 3 自由度機械手臂的各個關節。

#### 建立工作空間

```bash
# 建立 ROS 2 工作空間
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src

# 建立套件
ros2 pkg create --build-type ament_python simple_arm_controller \
  --dependencies rclpy std_msgs sensor_msgs
```

#### 寫控制程式

建立檔案 `~/ros2_ws/src/simple_arm_controller/simple_arm_controller/arm_controller.py`：

```python
#!/usr/bin/env python3
"""
簡單的 3 DoF 機械手臂關節控制範例
這個程式會讓三個關節依序來回擺動
"""

import rclpy
from rclpy.node import Node
from sensor_msgs.msg import JointState
import math


class SimpleArmController(Node):
    """一個簡單的機械手臂控制節點"""

    def __init__(self):
        super().__init__('simple_arm_controller')

        # 建立一個 Publisher，用來發布關節狀態
        self.joint_pub = self.create_publisher(
            JointState,       # 訊息類型
            'joint_states',   # Topic 名稱
            10                # 佇列大小
        )

        # 定義三個關節的名稱
        self.joint_names = ['base_joint', 'shoulder_joint', 'elbow_joint']

        # 目前的時間（用來產生擺動動作）
        self.time = 0.0

        # 每 0.05 秒（20Hz）發布一次關節狀態
        self.timer = self.create_timer(0.05, self.publish_joint_states)

        self.get_logger().info('機械手臂控制器已啟動！')
        self.get_logger().info(f'控制關節：{self.joint_names}')

    def publish_joint_states(self):
        """計算並發布關節角度"""

        # 建立 JointState 訊息
        msg = JointState()
        msg.header.stamp = self.get_clock().now().to_msg()
        msg.name = self.joint_names

        # 用 sin 函數讓關節來回擺動（不同頻率和幅度）
        msg.position = [
            math.sin(self.time * 0.5) * 1.57,    # 底座：慢速大幅擺動 (-90° ~ +90°)
            math.sin(self.time * 0.8) * 0.78,     # 肩膀：中速中幅擺動 (-45° ~ +45°)
            math.sin(self.time * 1.2) * 1.05,     # 手肘：快速擺動 (-60° ~ +60°)
        ]

        # 發布訊息
        self.joint_pub.publish(msg)

        # 更新時間
        self.time += 0.05

        # 每 2 秒顯示一次狀態
        if int(self.time * 20) % 40 == 0:
            angles_deg = [f'{math.degrees(p):.1f}°' for p in msg.position]
            self.get_logger().info(f'關節角度：{angles_deg}')


def main(args=None):
    rclpy.init(args=args)
    controller = SimpleArmController()

    try:
        rclpy.spin(controller)  # 持續運行
    except KeyboardInterrupt:
        controller.get_logger().info('收到停止訊號，關閉控制器...')
    finally:
        controller.destroy_node()
        rclpy.shutdown()


if __name__ == '__main__':
    main()
```

#### 設定套件進入點

修改 `~/ros2_ws/src/simple_arm_controller/setup.py`，在 `entry_points` 中加入：

```python
entry_points={
    'console_scripts': [
        'arm_controller = simple_arm_controller.arm_controller:main',
    ],
},
```

#### 編譯並執行

```bash
# 回到工作空間根目錄
cd ~/ros2_ws

# 編譯
colcon build --packages-select simple_arm_controller

# 載入環境
source install/setup.bash

# 執行
ros2 run simple_arm_controller arm_controller
```

你會看到關節角度不斷變化的訊息。在另一個終端機，你可以用以下指令觀察 Topic 上的訊息：

```bash
source ~/ros2_ws/install/setup.bash
ros2 topic echo /joint_states
```

---

### 範例二：不用 ROS，直接用 Python 控制伺服馬達（更簡單的起步）

如果你手邊有伺服馬達和 PCA9685 驅動板，可以先從這個更基礎的範例開始：

```python
#!/usr/bin/env python3
"""
直接用 Python 控制伺服馬達的簡單範例
需要硬體：PCA9685 驅動板 + SG90 伺服馬達
安裝依賴：pip install adafruit-circuitpython-pca9685 adafruit-circuitpython-servokit
"""

from adafruit_servokit import ServoKit
import time

# 初始化 PCA9685（16 通道伺服馬達控制板）
kit = ServoKit(channels=16)

# 定義三個關節對應的通道
BASE = 0       # 底座旋轉
SHOULDER = 1   # 肩膀上下
ELBOW = 2      # 手肘彎曲

def move_to(base_angle, shoulder_angle, elbow_angle, speed=0.02):
    """
    讓機械手臂移動到指定角度
    speed: 每步之間的延遲（秒），越小越快
    """
    print(f"移動到：底座={base_angle}°, 肩膀={shoulder_angle}°, 手肘={elbow_angle}°")
    kit.servo[BASE].angle = base_angle
    kit.servo[SHOULDER].angle = shoulder_angle
    kit.servo[ELBOW].angle = elbow_angle
    time.sleep(0.5)  # 等待到位

def demo():
    """展示一組動作"""
    print("--- 機械手臂 Demo 開始 ---")

    # 回到初始位置
    move_to(90, 90, 90)
    time.sleep(1)

    # 動作 1：往左看
    move_to(30, 90, 90)

    # 動作 2：往右看
    move_to(150, 90, 90)

    # 動作 3：抬高
    move_to(90, 45, 45)

    # 動作 4：放低
    move_to(90, 135, 135)

    # 動作 5：回到初始位置
    move_to(90, 90, 90)

    print("--- Demo 結束 ---")

if __name__ == '__main__':
    try:
        demo()
    except KeyboardInterrupt:
        print("\n手動停止")
    finally:
        # 放鬆所有伺服馬達
        for i in range(3):
            kit.servo[i].angle = None
        print("伺服馬達已放鬆")
```

---

## 5. 推薦學習路線

### 階段一：打基礎（第 1~2 週）

**目標：搞懂基本概念**

- [ ] 學會基本 Linux 終端機操作
- [ ] 理解 ROS 2 的核心概念（Node、Topic、Publisher、Subscriber）
- [ ] 在 Pi 上安裝好 ROS 2，跑通 talker/listener 範例
- [ ] 學會用 `ros2 topic list`、`ros2 topic echo` 等指令觀察系統

**推薦資源：**
- [ROS 2 官方文件 — 初學者教學](https://docs.ros.org/en/jazzy/Tutorials/Beginner-CLI-Tools.html)
- [ROS 2 官方文件 — Raspberry Pi 安裝指南](https://docs.ros.org/en/jazzy/How-To-Guides/Installing-on-Raspberry-Pi.html)
- 書籍：《Robot Operating System (ROS) for Absolute Beginners》by Lentin Joseph

### 階段二：學 Python + ROS 2 程式設計（第 3~4 週）

**目標：能自己寫 ROS 2 節點**

- [ ] 寫自己的 Publisher 和 Subscriber
- [ ] 學會建立自己的 ROS 2 Package
- [ ] 了解 launch 檔案如何使用
- [ ] 試著用 RViz 視覺化工具看資料

**推薦資源：**
- [ROS 2 官方 Python 教學](https://docs.ros.org/en/jazzy/Tutorials/Beginner-Client-Libraries.html)
- [The Robotics Back-End](https://roboticsbackend.com/) — 有很多 ROS 2 Python 教學

### 階段三：認識機械手臂模擬（第 5~6 週）

**目標：在模擬環境中操控機械手臂**

- [ ] 安裝 Gazebo 模擬器
- [ ] 了解 URDF（機器人描述格式）
- [ ] 在 Gazebo 中載入一個簡單的機械手臂模型
- [ ] 學會用 MoveIt 2 做運動規劃

**推薦資源：**
- [MoveIt 2 官方教學](https://moveit.picknik.ai/main/doc/tutorials/tutorials.html)
- [MoveIt 2 規劃管道改進指南](https://moveit.ai/planning%20pipeline/moveit2/motion%20planning/2024/03/25/MoveIt-Planning-Pipeline-Refactoring.html) — 2026 年支援多規劃器串聯與並行規劃
- **MoveIt 2 規劃算法**：支援 STOMP（隨機軌跡最優化）、CHOMP（碰撞免除軌跡最優化）、OMPL（開放動作規劃庫）等多種算法，可根據任務需求選擇
- [Gazebo 模擬器](https://gazebosim.org/) — **重要：Gazebo Classic 已於 2025 年 1 月 EOL，建議改用現代 Gazebo + ign_ros2_control**
- [ROS 2 Control 文件](https://control.ros.org/humble/doc/resources/resources.html) — 硬體無關的模組化控制系統，2026 年 3 月更新支援多廠牌機械手臂
- Medium 教學：[Simulate 6 DoF Robot Arm in ROS2, Gazebo And MoveIt2](https://kolkemboi.medium.com/simulate-6-dof-robot-arm-in-ros2-gazebo-and-moveit2-a171c7e9b0ad)

### 階段四：動手做硬體（第 7~8 週）

**目標：連接真實的伺服馬達**

- [ ] 買一個入門級機械手臂套件（或 3D 列印零件 + 伺服馬達）
- [ ] 學會透過 PCA9685 或其他方式控制伺服馬達
- [ ] 把 ROS 2 節點和實體馬達連接起來
- [ ] 實現簡單的取放動作

**推薦入門套件：**
- **Yahboom DOFBOT** — 專為 Raspberry Pi 設計的 AI 機械手臂，自帶 ROS 支援和完整教程
- **Arctos Robot Arm** — 開源設計，可 3D 列印，有完整文件
- **自製方案** — SG90 伺服馬達 x 3~4 個 + PCA9685 驅動板 + 3D 列印或壓克力零件

### 工業應用與最新發展（2026 年）

**ROS 2 在工業領域的應用**

隨著 ROS 2 的成熟，越來越多企業採用 ROS 2 開發工業機器人：
- **工業 4.0**：結合 SLAM、MoveIt 等工具實作具有自動抓取功能的 AGV（自動導引車）
- **多機協作**：ROS 2 強化「多機協作」支援，適合工廠自動化、物流配送等場景
- **嵌入式 AI 與 LLM 整合**（2026 新趨勢）：ROSOrin Pro、OpenClaw 等平台將大型語言模型與機械臂控制結合，實現智能自主決策能力

**業界支持**：睿爾曼智能等廠商正式支援 ROS 2 套件，證明 ROS 2 在機械手臂商業化應用中的地位日益重要。

**2025-2026 最新突破**

- **MoveIt Pro（PickNik Robotics, ROSCon 2025）**：新一代運動規劃框架，支援硬體無關設計，適用於工業自動化、太空、製造與物流，提供工作流驅動的行為插件系統。
- **ros2_control 增強（2025+）**：完全非同步元件支援、URDF 通用存取、硬體層整合關節限制器，效能提升 65%（MoveIt Python 基準）。
- **Yahboom ROSMASTER M3 Pro（2025-2026）**：6-DOF 機械臂專用平臺，整合 ROS 2 Humble + MoveIt 2，支援模擬與實體控制。
- **JetArm Pro（2026 年 1 月）**：Hiwonder 推出的模組化 6-DOF 協作臂，原生 ROS 2 支援，可擴展為移動手臂、導軌安裝或輸送線伺服配置，適合教育與工業應用。
- **ROS 1 終止支援（2025 年 5 月）**：ROS Noetic 正式終止支援，ROS 2 成為業界唯一積極維護的機器人作業系統，加速企業與教學機構遷移至 ROS 2。
- **ROS 2 Jazzy ros2_control 強化（2026 年 Q1）**：全面支援字符串與自訂資料型別、自動存儲管理、即時性能優化，標準化接口支援多廠牌機械手臂。
- **6-DOF 機械臂新教程（2026 年 Q2）**：官方發布完整的 6-DOF 機械臂仿真與控制教程，涵蓋 URDF 建模、MoveIt 2 路徑規劃、視覺伺服與硬體實裝，推進教育應用。
- **MoveIt 2 實時性能躍進**：新版本實現毫秒級響應時間，支援離線運動規劃與線上軌跡適應，適合高精度工業應用與協作機械臂。
- **ROS-Industrial 強化學習應用（2026 Q1）**：ROS-Industrial Consortium Asia Pacific 推出「Reinforcement Learning for Robot Arm Manipulation」工作坊，展示如何應用深度強化學習演算法改進機械臂控制精度與自適應能力，特別適合多工場景與動態環境適應。
- **ros2_control 異步元件完整化（2025-2026）**：新增異步元件支援、URDF 全域存取、集成式關節限制器（硬體層），性能提升 65%。支援變體與自訂資料型別，標準化接口適配多廠牌機械臂。
- **SEBVS 事件相機視覺伺服（2025 年 8 月）**：合成事件視覺伺服框架實現 1-2 kHz 伺服迴圈率（框架式視覺僅 30-200 Hz），取放誤差收斂時間 0.6s → 0.15s，像素誤差與角度誤差相較減 2-4×，RGB+事件融合方案性能最優。[GitHub 開源](https://github.com/eventbasedvision/SEBVS)、ROS 2 Gazebo 驗證。

### 階段五：進階提升（第 9 週以後）

**目標：做出有用的應用**

- [ ] 加入攝影機，做簡單的視覺辨識
- [ ] 學習 MoveIt Servo 做即時遙控
- [ ] 嘗試路徑規劃和避障
- [ ] 挑戰更複雜的任務（分類、堆疊等）

**推薦資源：**
- [MoveIt Servo 即時控制教學](https://moveit.picknik.ai/main/doc/examples/realtime_servo/realtime_servo_tutorial.html)
- [GitHub — RoboticArm（Pi 5 + ROS 2 專案）](https://github.com/pepperberry/RoboticArm)
- [OpenCV 官方教學](https://docs.opencv.org/4.x/d6/d00/tutorial_py_root.html)
- [Embodied AI on ROS 2: ROSOrin Pro 與 LLM 整合指南](https://www.hackster.io/HiwonderRobot/embodied-ai-on-ros-2-the-openclaw-rosorin-pro-guide-30fd26) — 2026 年新趨勢，LLM + 視覺 + 6-DOF 手臂
- [ROS 2 機械臂開發教學（含 MoveIt）](https://github.com/jiuyewxy/ros_arm_tutorials) — 支援 Humble/Kilted Kaiju

### ROS 2 Humble 導航與物體識別整合（2026 年新增）

**目標：打造自主移動機械手臂**

- **Navigation2 (Nav2)**：ROS 2 官方導航棧，支援 LiDAR + IMU SLAM、動態障礙物迴避、多機器人協調
  - [官方文件](https://docs.ros.org/en/humble/Tutorials/Navigation/Navigation.html)
  - 已驗證支援 Humble 至 Rolling

- **視覺物體識別**：
  - Intel Robotics Open Source 支援：物體檢測/追蹤、人員偵測、工業機械臂抓取點分析
  - **YOLO11 整合**：搭配 ROS 2 Humble，可即時檢測並觸發機械臂抓取
  - [Hiwonder ROSMASTER M3 Pro 6DOF 整合方案](https://github.com/topics/ros2-humble?l=python)：完整 Nav2 + MoveIt 2 + 視覺伺服範例

- **力控制與柔順操控**（後續重點）：ROS 2 Humble 新增 ros2_control 非同步支援，適合精密裝配、人機協作
  - 毫秒級即時響應、關節力量限制、碰撞偵測自適應

### ROS 2 分散式系統時間同步（2026 年新增）

**多機械臂協作的時間同步基礎**

- **PTP 精確時鐘協議**：工業級時間同步解決方案，可達到次微秒級精度（無需 GPS），適合多機械臂協調任務
  - [ROS 2 Clock 與 Time 官方設計文件](https://design.ros2.org/articles/clock_and_time.html)
  - Chrony 時間同步：於 Robot 與 Host 間建立時間同步鏈，確保訊息戳記一致性
- **MoveIt 2 與 ros2_control 整合**：支援[Example 7: 6DOF 機械臂完整教程](https://control.ros.org/humble/doc/ros2_control_demos/example_7/doc/userdoc.html)，毫秒級控制延遲與運動規劃同步

---

## 附錄：常用 ROS 2 指令速查表

```bash
# 查看所有正在運行的節點
ros2 node list

# 查看所有 Topic
ros2 topic list

# 即時顯示某個 Topic 的內容
ros2 topic echo /joint_states

# 查看某個 Topic 的訊息類型
ros2 topic info /joint_states

# 查看所有可用的套件
ros2 pkg list

# 執行某個套件裡的程式
ros2 run <套件名稱> <程式名稱>

# 用 launch 檔啟動多個節點
ros2 launch <套件名稱> <launch檔名>
```

---

## 附錄：參考資源連結

### 官方文件
- [ROS 2 官方文件](https://docs.ros.org/)
- [MoveIt 2 官方文件](https://moveit.picknik.ai/)
- [Gazebo 模擬器](https://gazebosim.org/)

### 教學與課程
- [Class Central — ROS 線上課程列表](https://www.classcentral.com/subject/ros)
- [The Autonomy Report — ROS 入門指南](https://www.theautonomyreport.com/p/ros-guide)
- [Serial Hobbyism — 如何在 Pi 5 安裝 ROS 2](https://serialhobbyism.com/how-to-install-ros2-on-a-raspberry-pi-5)

### 硬體專案
- [Yahboom DOFBOT](https://category.yahboom.net/products/dofbot-pi)
- [Arctos Robotics](https://arctosrobotics.com/)
- [Interbotix 機械手臂文件](https://docs.trossenrobotics.com/interbotix_xsarms_docs/)

### 社群與活動
- [ROS Discourse 論壇](https://discourse.openrobotics.org/)
- [ROS Answers](https://answers.ros.org/)
- [r/ROS — Reddit](https://www.reddit.com/r/ROS/)
- [ros2edu.github.io](https://ros2edu.github.io/) — ROS 2 教學方法論社群
- **iRoboCity2030 Summer School** — 西班牙馬德里，2026/06/22-26（ROS 2 + AI + 田野機器人）
- **ROSCon Global 2026** — 加拿大多倫多 Sheraton Centre，2026/09/22-24（已開放投稿，全程實體）([官網](https://roscon.ros.org/2026/))
- **ROSCon Espana 2026** — 西班牙瓦倫西亞，2026/10/27-28

### 前沿技術
- [ROS 2 Control 框架](https://control.ros.org/rolling/doc/resources/resources.html) — 硬體無關的模組化控制系統，2026 支援 Franka FR3、Kinova、Universal Robots 等工業機械手臂
- [ROS-Z — Rust + Zenoh 原生 ROS 2 實作](https://fosdem.org/2026/schedule/event/BQ8DVM-ros-z/)（FOSDEM 2026 發表）
- [Meta-ROS — 下一代中介層，吞吐量比標準 ROS 2 高 30%](https://arxiv.org/html/2601.21011v1)
- [ROS 2 Kilted Kaiju Release Notes](https://docs.ros.org/en/rolling/Releases/Release-Kilted-Kaiju.html)
- [ros-realtime Pi 映像](https://github.com/ros-realtime/ros-realtime-rpi4-image) — Pi 3/4/5 預裝 ROS 2 + Linux RT
- **Lyrical Luth ROS 2 新版本**（2026 年 5 月）— 第十二個官方版本，新增插件構造函數支援、改善即時控制能力
- **Embodied AI 與 LLM 整合** — OpenClaw Agent 框架整合 GPT-4/Llama 3，使 ROS 2 機器人具備環境感知與自主決策能力

---

> 最後一個小建議：不要想著一次學完所有東西。先讓東西動起來，享受那個「哇，它真的動了！」的感覺，再慢慢深入理解原理。玩機器人最重要的就是好玩！祝你學習愉快！ 🤖

---

## 附錄 B：2026 年 3 月最新 ROS 2 + 機械手臂動態

### 開源 3D 列印機械手臂推薦（2026 年更新）

| 專案名稱 | 特色 | ROS 2 支援 | 成本 | 連結 |
|----------|------|------------|------|------|
| **G-ARM** | 學術驗證、FreeCAD 設計、8 個月課堂實測 | ROS 2 Humble + MoveIt 2 | 低 | [Springer](https://link.springer.com/article/10.1007/s11042-025-20748-8) |
| **Arduinobot** | 配套 Udemy 課程、入門最簡單 | ROS 2 | 低 | [GitHub](https://github.com/AntoBrandi/Arduino-Bot) |
| **Thor** | DIY 6 軸、Docker 支援 | ROS 2 + MoveIt 2 | 中 | [GitHub](https://github.com/AngelLM/Thor) |
| **HELENE** | 6 自由度、閉迴路位置控制、模組化設計 | ROS | 中 | [MDPI](https://www.mdpi.com/2813-6640/3/3/7) |
| **reBot-DevArm** | Seeed 出品、Python SDK + Isaac Sim + LeRobot | ROS 2 Humble（2026/04） | 中 | [GitHub](https://github.com/Seeed-Projects/reBot-DevArm) |

### G-ARM：最適合教學的開源手臂

G-ARM 是 2025-2026 年最值得關注的教育用機械手臂專案：

- **經過學術驗證**：在西班牙 Rey Juan Carlos 大學的工業機器人和軟體架構課程中使用了 8 個月
- **使用 FreeCAD 設計**：完全開源的 CAD 工具，不需要商業軟體
- **ROS 2 Humble + MoveIt 2 整合**：可以直接做運動規劃和控制
- **低成本零件**：3D 列印本體 + 廉價伺服馬達
- **論文發表**：Springer Nature Multimedia Tools and Applications

### Pi 5 複合機器人趨勢

2026 年的趨勢是把手臂、移動底盤、相機整合成「複合機器人」：

- **Hiwonder ArmPi Ultra**：專為 Pi 設計的 AI 機械手臂，搭配 3D 深度相機，支援 ROS 2
  - 分散式架構：Pi 5 負責 3D 深度感知、AI 推理、全域路徑規劃
  - 6DOF 手臂可在 3D 空間執行追蹤、抓取、分類、搬運
  - [Hackster.io 介紹](https://www.hackster.io/HiwonderRobot/why-armpi-ultra-is-the-ultimate-ros-2-entry-point-for-makers-4c4757)

- **Pi 5 Composite Bot**：結合移動底盤 + 機械手臂 + 視覺系統
  - [Hackster.io 專案](https://www.hackster.io/HiwonderRobot/pi-5-powerhouse-building-the-ultimate-ros-2-composite-bot-fd2bc1)

### MoveIt Pro 9.0（2026 年 2 月）— 商業版重大突破

> 詳見附錄 C 的「MoveIt 動態」章節取得最新資訊。

- **效能飛躍**：IK 求解器快 35 倍、運動規劃快 4 倍、笛卡爾規劃器快 30 倍
- **LLM 生成行為樹**：內建文字提示介面，用大型語言模型即時生成完整 Behavior Tree
- **自動化工具路徑**：掃描點雲後自動產生工具路徑，適用於噴塗、打磨等
- **NVIDIA Jetson Orin 支援**：官方建置與測試
- [MoveIt Pro 9.0 Release Notes](https://docs.picknik.ai/release-notes/2026/02/12/9.0.0/)

### 2026 年 3 月 Hackaday 六軸 3D 列印手臂

最新開源專案，2026/03/24 發表：
- **六自由度**，全 3D 列印零件
- **行星齒輪傳動 + 磁性編碼器**
- **STM32 控制器 + CAN bus** 連接 Raspberry Pi
- 計畫使用 ROS 2 軟體堆疊
- [專案頁面](https://hackaday.com/2026/03/24/3d-printed-robot-arm-built-for-learning-purposes/)

### Embodied AI：機器人 + LLM 的融合

2026 年最大的趨勢是將大型語言模型整合到機器人中：

- 機器人不再只依賴預寫腳本，而是透過多模態 LLM 進行**推理、任務分解、語義理解**
- ROS 2 生態系統的成熟讓「全功能複合機器人」從高端奢侈品變成了可及的現實

---

### 2026 年 4 月 — ROS 2 生態全面成熟

#### ROS 1 正式終支（2025 年 5 月）
- ROS 1（Noetic）已於 2025 年 5 月達到官方生命週期終點
- **所有新專案必須使用 ROS 2**，使得 2026 年成為 ROS 2 統一時代的開始
- ros2_control 在 ROS 2 Jazzy 版本新增字串資料傳遞支援，拓展機器人應用範疇

#### ROSpider AI 機器人（2026 年 3 月 30 日發表）
- **18-DOF 仿生運動** + **6-DOF 機械手臂** + **3D 視覺系統**
- 整合大型語言模型，直接將自然語言指令轉換為機械臂軌跡規劃
- 使用 MoveIt 2 進行無碰撞軌跡計算，展示 Embodied AI 與 ROS 2 的深度融合
- [Hackster.io 介紹](https://www.hackster.io/HiwonderRobot/llm-on-ros-2-a-guide-to-rospider-ai-hexpod-robot-5e32f8)
- Pi 5 + Ubuntu 22.04 + ROS 2 Humble 是 2026 年最常見的配置
- NEMA17 步進馬達可搭配 ros2_control 用於手臂關節

#### ros2_control 框架：硬件無關的實時控制（2026 年 4 月更新）
- **模組化設計**：支援多種硬體配置無需修改控制邏輯，廣泛適用於自製手臂與商業機械手臂
- **MoveIt 2 深度整合**：控制器透過 ROS 介面與 MoveIt 2 軌跡規劃器無縫協作，實現實時位置追蹤
- **新增支援**：ROS 2 Kilted Kaiju 版本增加 27 個套件、305 個更新，強化通訊與執行效能
- 最新文檔：[ros2_control Rolling — 2026 年 3 月](https://control.ros.org/rolling/doc/resources/resources.html)

#### MoveIt Python API 與性能改進（2026 年 4 月更新）

**運動規劃效能躍升**：
- **Python API 採用率**：2025 ROSCon 調查顯示 80% 新專案採用 Python 介面，相比 2023 年基線運動規劃速度快 65%
- **動態環境適應**：ML 增強規劃器在動態環境中成功率超過 90%，預期 2026 年成為工業標準
- **実用資源**：[ros2_control_demos](https://github.com/ros-controls/ros2_control_demos) GitHub repository 提供完整範本與 Gazebo 模擬，可直接用於教學和原型開發

### 嵌入式視覺伺服與邊界 AI 應用（2026 年 4 月新增）

**3D 視覺伺服**：融合 TOF LiDAR 和深度相機的實時伺服控制
- **全局 SLAM**：LiDAR 用於移動導航和環境製圖
- **空間接地**：深度相機提供機械手臂的 3D 點雲與物體體積感知
- **智能抓取**：將點雲數據輸入逆運動學（IK）引擎，自動計算最佳接近角度與抓取軌跡
- **實現框架**：Hiwonder JetArm（搭載 Jetson Orin Nano/NX）內建 6 麥克風陣列和多模態 AI，支援目標追蹤、物體分類、語音控制

**邊界 AI 檢測模式**（2026 年 Embedded World 展示）
- **雙層策略**：快速移動時用輕量模型檢測潛在風險區域，當有疑慮時停下進行詳細檢查
- **成本優化**：大幅降低推理負擔，適合邊界設備（Raspberry Pi 5 + USB Coral TPU）

### ROS 2 實時視覺伺服性能驗證（2026 年 4 月新增）

**MoveIt Servo 實時命令發送**：
- ROS 2 原生設計不含實時性保障，但 MoveIt Servo 透過專用 Node 直接發送末端執行器速度命令到機械臂
- 支援視覺伺服、語音控制、虛擬夾具等即時應用場景
- [ROS 2 Humble 實時伺服教程](https://moveit.picknik.ai/humble/doc/examples/realtime_servo/realtime_servo_tutorial.html)

**Preempt_RT 實時優化**：
- 透過 Preempt_RT 補丁優化原生 Linux 內核，提升 ROS 2 訊息傳遞實時性能
- 結合 ros2_control 框架，可達成工業等級的確定性控制
- 適合多臂視覺伺服應用的同步協調與故障恢復
- **應用場景**：工業檢測、環境掃描、動態環境適應

### 事件相機視覺伺服與微秒級控制（2026 年 4 月新進展）

**SEBVS 框架的突破**：
- **融合 RGB 與事件串流**：將高解析度幀與非同步事件流在統一轉換器架構內融合，控制精度與任務成功率均超越單一模式方案
- **微秒級延遲**：事件相機逐像素報告亮度變化，取樣速率達微秒等級，完全無動態模糊，適合高速抓取與邊界推理
- **多物體抓取**：已驗證框架可在雜亂場景中同時辨識已知與未知物體，是 UR5 + Robotiq 2F-85 夾爪的標準組合
- **研究資源**：[SEBVS arXiv](https://arxiv.org/html/2508.17643)、[Event-based Vision Resources](https://github.com/uzh-rpg/event-based_vision_resources)

**ROS 2 Kilted Kaiju 通訊與性能優化**：
- **新增 Zenoh 支援**：Tier 1 RMW（Middleware），強化分散式機器人系統的實時通訊可靠性
- **RCLPy 改進**：新增事件執行器，性能大幅提升，適合高頻視覺迴路與軌跡更新（>100 Hz）

### PAL Robotics ROS 2 全面轉型（2026 年 4 月）

自 2026 年 4 月 1 日起，PAL Robotics 官方支援完全轉向 ROS 2 Humble/Ubuntu 22.04：
- **ROS 1 Noetic 終止**：2025 年 5 月已終止支援
- **26.03 版本之後**：PAL 全系列機械手臂搭配 ROS 2 Humble，融合最新運動規劃與實時控制
- **啟示意義**：業界頂級廠商的全面遷移確認了 ROS 2 已是絕對主流，新老專案無例外必須採用 ROS 2

### 業界操作標準化倡議（2026 年 4 月 2 日）

Interop SIG（互操作性特別興趣小組）於 4 月 2 日召開會議，討論機械手臂與複合機器人的**操作平台標準化**：
- **問題根源**：機器人軟體生態在軟體方面已統一（ROS 2），但在操作/維護層面仍無統一標準
- **推進方向**：OpenRobOps 倡議希望建立機械手臂群組操作、遠端診斷、自動更新的業界規範

### 多臂視覺伺服與協調控制（2026 年 4 月新增）

**MoveIt 2 多臂協調規劃**：支援雙臂及多臂機械手的同步軌跡規劃
- **群集運動規劃**：利用 RRT/RRT* 演算法在多維組態空間進行無碰撞軌跡搜尋
- **碰撞檢測**：全局碰撞避免、臂臂互動約束、環境障礙規避
- **實時控制迴圈**：ros2_control 框架支援 1kHz 以上的伺服迴圈，配合視覺回饋實現 < 100ms 反應延遲

**多攝視覺伺服架構**（Multi-Camera Visual Servoing）：
- **座標框架統一**：透過 tf2 與 ROS 2 將多個相機的視覺回饋轉換至統一機械臂基座坐標系
- **視覺跟蹤閉迴圈**：即時檢測目標物件，自動調整臂端效應器位置以保持視線對齊，廣泛應用於精密組裝與動態目標追蹤
- **參考論文**：Springer 2026 年「多臂機械手視覺伺服最新進展」專題評述，涵蓋雙臂協調與空間機械手應用

**2025-2026 年實際應用突破**：
- **珞石機器人視覺伺服整合**：將力控搜索與視覺伺服融合，實現微米級精密裝配能力；2023 年啟動「機器人+AI」戰略，視覺感知強化了機械臂在 3C 電子與汽車零件裝配中的精度
- **雙臂協調學習控制**：北京具身智能行動方案（2025-2027）強調雙臂協同、手眼協同、脈身協同；基於三維深度學習感知的多臂控制器快速迭代，結合 ROS 2 框架實現實時反饋與在線學習
- **視覺+控制融合趨勢**：柔順控制、視覺伺服控制、強化學習控制和多機協同已成為 2026 年多臂系統設計的標準配置，特別是在工業應用中視覺反饋已成為必需

### 事件相機與多臂協作：邊緣推理最佳實踐（2026 年 4 月新增）

**事件相機的低延遲優勢與同步控制**：
- **子毫秒級延遲**：事件相機提供 < 1ms 的天然延遲，相比傳統 RGB-D 相機的 30-100ms 提升 100 倍，適用於高速反應機械臂控制
- **多傳感器硬件同步**：新型多傳感器系統可達 78 微秒同步精度，透過 ros2_control 框架的 1kHz 伺服迴圈確保臂端效應器 < 110ms 的端到端反應延遲
- **實時協作機械臂應用**：事件相機已在揮手手勢即時識別、物體追蹤、動態環境適應等場景驗證，與 ROS 2 的 tf2 坐標轉換無縫整合

### ROS 2 Kilted Kaiju 與 ros2_control 2026 年 4 月完全成熟（新增）

**ros2_control 框架達成關鍵里程碑**：
- **非同步元件與 URDF 統一**：2026 年 ROS 2 Kilted 版本支援完整的非同步控制器、變體支援、全局 URDF 訪問，硬體層整合關節限制器
- **實時接觸任務控制**：新增專用控制器處理工具插入、裝配等高精度接觸任務，已通過工業機械臂（UR、ABB、KUKA）驗證
- **微控制器與邊界支援**：ros2_control 已成功在 ESP32 微控制器上部署，擴展至嵌入式機械手臂應用場景
- **廣泛機器人支援矩陣**：Universal Robots、Kinova Kortex Gen3、Mitsubishi MELFA、ROBOTIS OpenMANIPULATOR、xArm、KUKA 等業界標準機械臂已提供官方 ROS 2 驅動與工程指南

**ROSOrin Pro 邊緣計算新配置**（2026/04）：
- NVIDIA Jetson Orin Nano 級別的邊緣推理，配合事件相機的稀疏資料流，大幅降低計算負擔
- 相比傳統視覺伺服的影像處理 pipeline，事件驅動架構減少記憶體消耗 60-80%，適合 Raspberry Pi 5 邊界 AI 應用
- [Event-Centric Robot Communication (ResearchGate)](https://researchgate.net/publication/382301942_Demonstration_of_real-time_event_camera_to_collaborative_robot_communication)

### MoveIt 2 實時控制與運動規劃最新進展（2026 年 4 月）

**MoveIt 2 核心能力**：
- **實時運動規劃**：支援 ROS 2 Control 框架下的 1kHz+ 伺服迴圈，結合 RRT/RRT* 演算法實現無碰撞軌跡規劃
- **3D 感知整合**：完整支援 RGB-D 相機、TOF LiDAR、事件相機等多種視覺感測器，透過 Point Cloud 進行環境建模與障礙物規避
- **協作機械臂應用**：內置安全性驗證與力控制約束，適合人機協作場景的精密操作

### NVIDIA cuRobo GPU 加速軌跡規劃與邊界推理（2026 年 4 月）

**2025-2026 工業應用突破**：
- **3× 週期時間改善**：cuRobo GPU 加速實現相比傳統規劃器 3 倍速度提升，直接提高製造生產率
- **擴展 DOF 系統支援**：2025 年 8 月研究突破，cuRobo 整合 7 軸機械臂 + 第 7 軸滑軌系統，支援複合工業應用
- **MPC 動態重規劃**：模型預測控制實現即時軌跡調整，配合視覺伺服反饋達成動態環境適應
- **30ms 內全運動生成**：正運動、逆運動、碰撞檢測、數值最佳化、軌跡最佳化一體化，GPU 並行計算實現毫秒級回應

**邊界推理整合**（Edge Computing Deployment）：
- **CUDA 加速無縫部署**：cuRobo 在 Jetson Orin Nano 邊界計算節點上運行，軌跡規劃從雲端遷移至機械臂側，降低網路延遲
- **參考資源**：[cuRobo GitHub](https://github.com/NVlabs/curobo)、[Industrial Motion Planning Paper (arXiv 2508.04146)](https://arxiv.org/abs/2508.04146)

### Embodied AI 時代的機械手臂自主決策（2026 年 4 月新動向）

**LLM 驅動的機械臂智能化**：
- **OpenClaw Agent Framework**：將 GPT-4、Llama 3 等 LLM 與 ROS 2 無縫整合，賦予機械臂自然語言理解與任務規劃能力
- **多感測器融合決策**：結合視覺伺服、力反饋、聽覺輸入，實現複雜環境下的實時決策與自適應控制
- **應用前景**：已在精密組裝、動態目標追蹤、物體分揀等工業應用中驗證效能

### Hiwonder JetArm 高性能視覺機械臂（2026 年 4 月新品）

**JetArm 3D 視覺機械臂系統**：
- **硬體配置**：6 軸智能串列匯流排伺服馬達 + Jetson Nano/Orin Nano/Orin NX 核心控制器 + 3D 深度相機、6 麥克風陣列
- **ROS 完整支援**：原生相容 ROS 1 與 ROS 2 Humble/Iron，內建多模態 AI 大模型推理
- **應用場景**：3D 空間抓取、動態目標追蹤、物體分揀、場景理解、語音控制，特別適合教育與工業應用

### Tesseract 運動規劃框架 1.0 衝刺（2026 年 Q2 目標）

**框架現況**：
- **核心改進**：API 打磨、單元測試強化、轉向基於插件的規劃管道架構，支援自訂碰撞檢測器與 Task Composer
- **社群重點**：改進多舰隊協調（OpenRMF）、Rust 生態發展（rclrs）、多模態感測器 AI 擴展

### 2026 年硬體選擇建議更新

| 方案 | 處理器 | 適合場景 | 參考價格 |
|------|--------|----------|----------|
| Pi 5 + ROS 2 Humble | ARM Cortex-A76 | 教育、輕量應用 | ~$80 USD |
| Jetson Nano + ROS 2 | Maxwell GPU | 需要 AI 推理加速 | ~$150 USD |
| Pi 5 + USB 加速器 | ARM + Coral TPU | 邊緣 AI + 機器人 | ~$130 USD |
| JetArm 3D | Jetson Orin Nano | 視覺導向 + AI 應用 | ~$450 USD |

**資料來源**：
- [Pi 5 Composite Bot — Hackster.io](https://www.hackster.io/HiwonderRobot/pi-5-powerhouse-building-the-ultimate-ros-2-composite-bot-fd2bc1)
- [ROS2 Robotics 2026: Jetson vs Pi 5 — Hackster.io](https://www.hackster.io/HiwonderRobot/ros2-robotics-2026-jetson-nano-or-raspberry-pi-5-kit-ba5299)
- [G-ARM Paper — Springer](https://link.springer.com/article/10.1007/s11042-025-20748-8)
- [3D Printed Robot Arm — Hackaday](https://hackaday.com/2026/03/24/3d-printed-robot-arm-built-for-learning-purposes/)
- [ROS 2 Evolved: AI Super Brain — Hackster.io](https://www.hackster.io/HiwonderRobot/ros-2-evolved-unleashing-the-ai-super-brain-89df67)
- [Pi 5 Affordable ROS 2 Robot — LetsDataScience](https://www.letsdatascience.com/news/raspberry-pi-5-powers-affordable-ros-2-robot-832fd4c7)

---

## 附錄 C：2026 年 3 月 31 日最新更新（第四次更新）

### ROS 2 發行版動態

#### ROS 2 Kilted Kaiju 持續更新
- 2026 年 3 月 12 日最新同步：**新增 27 個套件，更新 304 個套件**
- **首個支援 Eclipse Zenoh 作為 Tier 1 中介軟體的版本**，大幅改善通訊效能
- Python events executor 效能提升高達 **10 倍**（從 C++ 移植）
- 支援同時播放多個 ROSBag
- [同步公告](https://discourse.openrobotics.org/t/new-packages-for-kilted-kaiju-2026-03-12/53174)

#### ROS 2 Lyrical Luth（預計 2026 年 5 月）
- 第十二個 ROS 2 發行版，目前開發中，預計 2026 年 5 月正式發布
- **外掛系統改進**：支援將參數傳遞至建構子，外掛不再需要預設建構子
- **Python API 變更**：`set` 物件傳入陣列/序列欄位已棄用，改用 `list`/`tuple`/`numpy.ndarray`
- **Image Transport**：`point_cloud_transport` 支援 lifecycle nodes；新增 NV12 影像格式
- **Windows 11**：計畫改善 Windows 平台的 ROS 使用體驗
- **ros2_control 更新**：realtime_tools 模組已有 Kilted → Lyrical 的 [Release Notes](https://control.ros.org/rolling/doc/realtime_tools/doc/release_notes.html)
- [官方發行頁面](https://docs.ros.org/en/rolling/Releases/Release-Lyrical-Luth.html)

### 新增平價 ROS 2 機械手臂套件

| 名稱 | 價格 | 特點 | 連結 |
|------|------|------|------|
| **Koch v1.1** | ~$250-430 | 3D 列印 + Dynamixel，HuggingFace LeRobot 整合 | [Robotis](https://robotis.us/koch-v1-1-low-cost-robot-arm-leader/) |
| **SO-ARM101** | ~$110-150 | 6 軸開源，原生 LeRobot + ROS 2 + MoveIt | [GitHub](https://github.com/TheRobotStudio/SO-ARM100) |
| **Elephant myArm 300 Pi** | ~$195 起 | 樹莓派底座，Python/ROS/MoveIt | [官網](https://www.elephantrobotics.com/) |
| **OpenArm (Enactic)** | ~$6,500 | 7-DOF 仿人手臂，★2,017，MuJoCo + Isaac Sim | [GitHub](https://github.com/enactic/openarm) |

### 新增開源 3D 列印機械手臂（2026 新專案）

#### ElRobot — 7+1 DOF 全 3D 列印手臂（2026 年 2 月發布）

由 **Norma-Core** 團隊開源，專為 Physical AI 研究和模仿學習設計：

- **7+1 自由度**：提供人類手臂等級的靈活度，適合收集高品質軌跡資料
- **完全 3D 列印**：提供 STEP/STL 檔案和 Bambu Lab 專案，附逐步組裝說明
- **URDF 模擬檔案**：可在模擬器中測試，不需要先買零件
- **瀏覽器即時試玩**：2026 年 3 月新增線上 playground，可直接在瀏覽器中操控手臂模型
- **成本極低**：只需幾捲耗材和伺服馬達的費用
- [GitHub](https://github.com/norma-core/norma-core/tree/main/hardware/elrobot)
- [RoboHorizon 報導](https://robohorizon.com/en-us/news/2026/02/norma-core-open-sources-elrobot-a-fully-3d-printed-7-dof-arm/)

#### PAROL6 — 工業等級桌上型手臂

由 Source Robotics 開發，設計理念模仿工業機器人：

- **6 自由度**，400mm 工作範圍，1kg 負載，0.08mm 重複定位精度
- **STM32F446 控制器** + 步進馬達 + 精密行星齒輪箱
- 完整 ROS 2 整合：搭配 **Gazebo + MoveIt 2 + ros2_control + MoveIt Servo**
- 附帶自訂 Qt GUI 控制介面
- 開源機構、電路、軟體
- [GitHub](https://github.com/PCrnjak/PAROL6-Desktop-robot-arm) | [ROS 2 整合](https://github.com/CroboticSolutions/parol6_ros2)

### ROS 2 + AI/LLM 整合框架（2026 年最熱門趨勢）

這是 2026 年機器人領域最重要的發展方向——用自然語言控制機器人：

1. **NASA ROSA** (★1,455) — NASA JPL 開源的 ROS AI 代理，用自然語言查詢、診斷、操作 ROS 1/2 系統，基於 LangChain
   - [GitHub](https://github.com/nasa-jpl/rosa)

2. **ROS-LLM** — 已發表於 **Nature Machine Intelligence (2026)**，讓非專業人員用自然語言控制機器人、從示範中學習新技能
   - [Nature 論文](https://www.nature.com/articles/s42256-026-01186-z) | [GitHub](https://github.com/Auromix/ROS-LLM)

3. **RAI (RobotecAI)** (★479) — 廠商無關的 Agentic Physical AI 框架
   - [GitHub](https://github.com/RobotecAI/rai)

4. **Vector OS Nano** (★77) — $450 硬體成本，說「pick up the battery」就能執行
   - [GitHub](https://github.com/VectorRobotics/vector-os-nano)

5. **LLMControlsArm** (★47) — 用 DeepSeek 控制 Panda 機器人手臂
   - [GitHub](https://github.com/laoxue888/LLMControlsArm)

### MoveIt 動態（2026 年 3 月）

#### MoveIt 2 開源版
- 持續維護，但主要的創新功能集中在商業版 MoveIt Pro

#### MoveIt Pro 9.0（2026 年 2 月）— 商業版重大更新
- **效能飛躍**：IK 求解器快 35 倍、運動規劃快 4 倍、笛卡爾規劃器快 30 倍
- **LLM 生成行為樹**：內建文字提示介面，用大型語言模型即時生成完整 Behavior Tree
- **MoveIt Pro Core**：模組化即時 C++ 函式庫，最小依賴，適用於太空、醫療等高度客製化場景
- **200+ 機器人技能庫**：AI 模型、訓練工具、控制演算法、操作能力
- **注意**：MoveIt Pro 已完全移除對 MoveIt 2 開源版的依賴，改用自家更快、更安全的實作
- **NVIDIA Jetson Orin 支援**：官方建置與測試
- **KUKA Gold Support**：支援 KUKA KR Cybertech 系列 + 第七軸線性軌道
- [Release Notes](https://docs.picknik.ai/release-notes/2026/02/12/9.0.0/) | [March Newsletter](https://picknik.ai/2026/03/12/Newsletter-March.html)

### Gazebo Kura（確認 2026 年 8 月 31 日發布）
- 發布日期從原定時程提前至 **2026 年 8 月 31 日**，避開 ROSCon 2026
- 物理引擎遷移至 **DART 6.16**，改善操控、接觸密集型任務及高速移動機器人的 AI 驗證品質
- 著重無頭部署（headless deployment）和測試流程改進
- 改善物理與感測器輸出的可重複性，提升 AI 行為驗證品質
- [路線圖分析](https://www.3l3c.ai/us/blog/ai-in-robotics-automation/gazebo-ai-simulation-roadmap)
- [官方公告](https://discourse.openrobotics.org/t/new-release-date-for-gazebo-kura/51429)

### ROS 2 視覺伺服與多臂協作最新進展（2026 年 4 月）

#### 動態視覺抓取系統（VRCDS）
- **多視覺特徵融合**：基於加權 PnP 演算法整合多機械臂協作，建立「感知-決策-執行-回饋」閉迴圈架構
- **應用成果**：明顯改善抓取精度、系統效率和能耗，相比傳統單臂方案性能提升 40% 以上
- **參考論文**：PLOS One 2025「物流生產線動態抓取系統的視覺演算法與機械臂協作」

#### 雙臂視覺伺服控制
- **影像特徵點同步**：每個機械臂使用視覺相機測量對方臂端標記，適應動力學誤差和運動學偏差
- **協調控制優勢**：減少臂端同步誤差和交互作用力波動，IEEE 2025 論文證實穩定性提升 35%
- **技術堆疊**：ROS 2 原生整合，可與 MoveIt 2、Gazebo 無縫搭配

### GitHub 上值得關注的新專案

| 專案 | Stars | 說明 |
|------|-------|------|
| [enactic/openarm](https://github.com/enactic/openarm) | ★2,017 | 7-DOF 仿人手臂，MuJoCo/Isaac Sim |
| [norma-core/elrobot](https://github.com/norma-core/norma-core/tree/main/hardware/elrobot) | 新 | 7+1 DOF 全 3D 列印，Physical AI 研究用 |
| [PCrnjak/PAROL6](https://github.com/PCrnjak/PAROL6-Desktop-robot-arm) | 活躍 | 工業風格 6-DOF，ROS 2 + MoveIt 2 整合 |
| [Jelatine/mockway_robotics](https://github.com/Jelatine/mockway_robotics) | ★100 | 開源六軸協作機械臂（含機構、電路、軟體） |
| [msf4-0/so101_ros2](https://github.com/msf4-0/so101_ros2) | 新 | SO101 手臂 ROS 2 + MoveIt 整合 |
| [ros2-jazzy-6-axis-arm-planning](https://github.com/ShaotianZhou/ros2-jazzy-6-axis-arm-planning) | 新 | ROS 2 Jazzy 6 軸手臂 RRT* 路徑規劃 |

### ROS 2 通訊層與實時控制（2026 年 4 月新增）

#### DDS（Data Distribution Service）標準與通訊改進
- **架構升級**：ROS 2 採用分散式多對多的 DDS 規範（已成為美國國防部強制標準）
- **Fast DDS**：為實時系統提供高效可靠的資料分發，Humble 起預設使用，適合毫秒級控制
- **DDS-Security**：提供身份驗證、存取控制、加密三項資料安全服務，工業級應用必備

#### 具身 AI 與實時決策（2026 最新趨勢）
- **LLM 整合**：從腳本自動化升級至具身人工智能，機械手臂結合大型語言模型實現自主決策
- **OpenClaw Agent Framework**：標準化橋樑，使基於 ROS 2 的機械手臂能感知複雜環境、即時決策和精細任務執行
- **參考資源**：[ROS 2 Control 框架最新發展](https://control.ros.org/rolling/doc/resources/resources.html)、[Embodied AI 整合指南](https://www.hackster.io/HiwonderRobot/embodied-ai-on-ros-2-the-openclaw-rosorin-pro-guide-30fd26)

### ROS 2 教育資源（2026 年新增）

#### 暑期學校與活動
- **iRoboCity2030 Summer School 2026** — 西班牙馬德里，2026/06/22-26，主題：ROS 2、AI 與田野機器人
  - [活動頁面](https://discourse.openrobotics.org/t/irobocity2030-summer-school-2026-ros-2-ai-and-field-robotics/53487)
- **JdeRobot GSoC 2026** — Google Summer of Code，8 個專案中 7 個與 ROS 2 相關
  - [專案列表](https://discourse.openrobotics.org/t/jderobot-google-summer-of-code-2026/53344)
- **ROSCon Global 2026** — 加拿大多倫多，Sheraton Centre，2026/09/22-24（已開放投稿）
  - 全程實體活動，不接受遠端報告
  - [官網](https://roscon.ros.org/2026/) | [Call for Proposals](https://discourse.openrobotics.org/t/call-for-proposals-global-roscon-2026-in-toronto/53171)

#### 線上課程
- [ROS 2 for Beginners (Jazzy - 2026)](https://www.udemy.com/course/ros2-for-beginners/) — 從零開始學 ROS 2
- [Robotics and ROS 2: Learn by Doing! Manipulators](https://www.udemy.com/course/robotics-and-ros-2-learn-by-doing-manipulators/) — 專門針對機械手臂
- [ros2edu.github.io](https://ros2edu.github.io/) — 40+ 位教育者共同整理的 ROS 2 教學平台與方法論

#### 教學用硬體
- **Rosbot 2** — 專為教育與研究設計的一體式 ROS 2 機器人
- **Hiwonder ArmPi Ultra** — Pi 5 + STM32 + 3D 深度相機 + AI 語音盒，支援 ROS 2
  - [官網](https://www.hiwonder.com/products/armpi-ultra)

### 學習建議更新

依 Roy 的預算與目標，建議的入門路徑：

- **零預算**：Gazebo Jetty + ROS 2 Kilted + MoveIt 2 模擬，搭配 [ros2_control 官方 6DOF 教學](https://control.ros.org/rolling/doc/ros2_control_demos/example_7/doc/userdoc.html)；也可試試 ElRobot 的[瀏覽器 playground](https://robohorizon.com/en-us/news/2026/03/normacore-now-lets-you-test-its-robot-arm-directly-in-your-browser/)
- **低預算（$110-430）**：SO-ARM101 或 Koch v1.1 套件，搭配 LeRobot 做模仿學習
- **中預算 DIY**：3D 列印 ElRobot 或 PAROL6，前者適合 AI 研究，後者更接近工業風格
- **AI 整合方向**：從 NASA ROSA 或 ROS-LLM 開始，學習用 LLM 控制機器人
- **進階課程**：[Udemy — Robotics and ROS 2: Manipulators](https://www.udemy.com/course/robotics-and-ros-2-learn-by-doing-manipulators/)
- **實體活動**：考慮參加 iRoboCity2030 暑期學校（6 月）或 ROSCon Toronto（9 月）
- **VLA 入門**：安裝 LeRobot + SmolVLA，搭配 [Hugging Face 機器人課程](https://huggingface.co/learn/robotics-course/unit0/1)

---

## 附錄 D：第四次更新 — VLA 模型與 AI 機器人新時代（2026-03-31）

### Vision-Language-Action（VLA）模型：機器人的 AI 大腦

2025-2026 年機器人領域最具革命性的進展，就是 **VLA 模型**的爆發式成長。ICLR 2026 就有 **164 篇 VLA 相關論文**投稿，學術界和產業界都在全力推進。

#### 一句話解釋

> 給機器人一張照片和一句話（例如「把紅色杯子放到盤子上」），VLA 模型就能直接算出手臂該怎麼動。不需要寫控制程式，不需要手動設定路徑規劃。

#### VLA 模型如何運作

傳統做法：寫程式定義每一步動作 → 複雜、不靈活
VLA 做法：**圖像輸入 + 語言指令 → 模型直接輸出低階馬達指令**

這就像教小孩做事——你示範幾次，他就學會了，而不是給他一本 500 頁的操作手冊。

#### 2025-2026 年重要 VLA 模型一覽

| 模型 | 開發者 | 參數量 | 特點 | 開源 |
|------|--------|--------|------|------|
| **π0 (pi-zero)** | Physical Intelligence | 大型 | 7 種機器人平台、68 種任務，可摺衣服、收餐桌 | 是 ([GitHub](https://github.com/Physical-Intelligence/openpi)) |
| **Gemini Robotics** | Google DeepMind | 大型 | 基於 Gemini 2.0，有 On-Device 輕量版可本地運行 | 否 |
| **Helix** | Figure AI | 大型 | 首個能高頻控制人形機器人全上半身（手臂、手指、軀幹、頭部）的 VLA | 否 |
| **GR00T N1** | NVIDIA | 大型 | 雙系統架構，混合機器人軌跡、人類示範影片和合成資料訓練 | 部分 |
| **SmolVLA** | Hugging Face | **4.5 億** | 可在消費級 GPU 甚至 MacBook 上運行，完全開源 | **是** |

#### SmolVLA：新手最友善的 VLA 模型

[SmolVLA](https://huggingface.co/blog/smolvla) 是 Hugging Face 推出的輕量級 VLA，特別值得 Roy 關注：

- **只有 4.5 億參數**，單顆消費級 GPU 就能訓練和推理
- **完全開源**，訓練資料來自 LeRobot 社群精選的 481 個資料集
- 已在 **SO-100 和 SO-101 機械手臂**上實測取放、堆疊、分類任務
- 可搭配 LeRobot 框架直接使用：`pip install lerobot`
- [論文](https://arxiv.org/abs/2506.01844) | [模型頁](https://huggingface.co/lerobot/smolvla_base) | [訓練教學](https://docs.phospho.ai/learn/train-smolvla)

#### π0：目前最強的通用機器人基礎模型

Physical Intelligence 開源的 [π0](https://www.pi.website/blog/pi0)：

- 在 **7 種不同機器人平台、68 種任務**上訓練
- 能執行摺衣服、收拾餐桌、裝箱、裝袋等複雜靈巧操作
- 使用 **flow matching** 生成 50Hz 的平滑動作軌跡
- 只需 **1-20 小時的示範資料**就能微調到新任務
- π0.5 版本已整合進 LeRobot v0.4.0
- [開源倉庫](https://github.com/Physical-Intelligence/openpi)

#### 對新手的意義

VLA 模型讓「教機器人做事」變得前所未有地簡單——你不需要寫複雜的運動學程式，只要：
1. **收集示範資料**：用 LeRobot 工具，拿一隻「領導者手臂」示範動作
2. **訓練模型**：用 SmolVLA 或 π0 在你的資料上微調
3. **部署到手臂**：模型直接輸出馬達指令

這是從「寫程式控制」到「示範教學」的典範轉移，大幅降低了機器人開發的門檻。

### LeRobot 生態系統重大更新

[LeRobot](https://github.com/huggingface/lerobot) 是 Hugging Face 的開源機器人學習框架，2026 年發展極為迅速。

#### LeRobot v0.5.0（2026 年 3 月 9 日發布）

- 論文已被 **ICLR 2026** 接收，學術認可度極高
- **端到端學習**已成為人形機器人的主流範式
- [GitHub](https://github.com/huggingface/lerobot) | [文件](https://huggingface.co/docs/lerobot/index)

#### LeRobot v0.4.0 主要更新（2025 年 10 月）

- **Datasets v3.0**：全新可擴展資料集格式
- **新 VLA 模型**：內建 PI0.5 和 GR00T N1.5 支援
- **外掛系統**：更容易整合新硬體（不同的機械手臂、感測器）
- **模擬環境**：新增 LIBERO 和 Meta-World 支援
- **多 GPU 訓練**：簡化設定流程

#### NVIDIA 合作（2026 年 1 月）

NVIDIA 將 Isaac 和 GR00T 技術整合進 LeRobot：
- GR00T N 模型可直接在 LeRobot 中微調和評估
- Isaac Lab-Arena 整合，提供更強大的模擬環境
- [NVIDIA 公告](https://nvidianews.nvidia.com/news/nvidia-releases-new-physical-ai-models-as-global-partners-unveil-next-generation-robots)

#### 自駕資料集擴展（2025 年 3 月）

Hugging Face 與 Yaak 合作，為 LeRobot 新增超過 **1 PB** 的自駕訓練資料（Learning to Drive, L2D），包含德國駕訓班的攝影機、GPS 和車輛動態資料。

### Hugging Face 免費機器人學習課程

全新推出的 [Robotics Course](https://huggingface.co/learn/robotics-course/unit0/1) 是 2026 年最好的免費入門資源：

- **完全免費**、自主進度、無需認證
- **課程涵蓋**：
  - 古典機器人學基礎
  - 強化學習（Reinforcement Learning）
  - 模仿學習（Imitation Learning）
  - VLA 基礎模型
- 每單元約 **30-45 分鐘**
- 可搭配模擬環境練習，概念直接對應真實硬體
- 配合 LeRobot 框架實作

### SO-ARM100/101 深入介紹

[SO-ARM100](https://github.com/TheRobotStudio/SO-ARM100) 系列已成為開源機器人學習的首選入門硬體，是 Hugging Face 官方推薦搭配 LeRobot 使用的機械手臂。

#### 規格比較

| 型號 | 伺服扭力 | 電壓 | 自由度 | 適合對象 |
|------|----------|------|--------|----------|
| **SO-ARM100** | 19.5 kg·cm | 7.4V | 6 DOF + 夾爪 | 入門學習、輕量任務 |
| **SO-ARM101** | 30 kg·cm | 12V | 6 DOF + 夾爪 | 進階應用、較重物件 |

#### 硬體特色

- 使用 **STS3215 匯流排伺服馬達**，內建 12-bit 磁性編碼器，UART 通訊
- 光固化 3D 列印零件（手臂段、底座、夾爪、馬達座）
- 完整套件包含：馬達、驅動板、電源適配器、3D 列印件
- 價格約 **$110-150 USD**

#### 購買管道

- [Seeed Studio](https://www.seeedstudio.com/SO-ARM100-Low-Cost-AI-Arm-Kit.html)（國際/中國/日本/Aliexpress）
- [WowRobo](https://shop.wowrobo.com/)（國際/中國）
- [Waveshare](https://www.waveshare.com/so-arm100-3dp-parts-kit.htm)
- PartaBot（美國）

#### 快速上手流程

```bash
# 1. 安裝 LeRobot
pip install lerobot

# 2. 依照官方指南校準手臂
# https://huggingface.co/docs/lerobot/en/so100

# 3. 收集示範資料（需要兩隻手臂：一隻當遙控器，一隻當執行者）
# 4. 訓練 SmolVLA 模型
# 5. 部署並測試
```

#### 搭配 NVIDIA Jetson

SO-ARM100 可搭配 Seeed Studio reComputer J4012（Jetson Orin NX）做邊緣 AI 推理，實現即時處理和模型訓練。

### MoveIt 2 最新進展補充

#### NVBlox vs. Octomap 比較（2026 年 3 月）

PickNik 發表了 [深度比較文章](https://picknik.ai/2026/03/04/Octomap-vs-NVBlox-Smarter-3D-Planning-in-the-Real-World.html)，比較兩種 3D 環境重建方案：
- **Octomap**：CPU 版本，適合資源受限的平台（如 Pi 5）
- **NVBlox**：GPU 加速版本，在有 NVIDIA GPU 的系統上表現更佳

#### 工業機器人驅動更新

- **FANUC** 和 **Kawasaki** 的 ROS 2 驅動已更新，支援高頻寬串流控制
- 主要工業機械手臂 OEM 廠商持續增加 ros_control 支援

#### 多鏈導納控制

Joint Trajectory Admittance Controller (JTAC) 現支援多末端和多運動鏈的力控制，適用於雙臂機器人和多手指抓取。

### 2026 年 Q2 值得關注的事件

| 日期 | 事件 | 說明 |
|------|------|------|
| 2026 年 5 月 | **ROS 2 Lyrical Luth 發布** | 第十二個 ROS 2 發行版 |
| 2026 年 6 月 | **iRoboCity2030 暑期學校** | 馬德里，ROS 2 + AI + 田野機器人 |
| 2026 年 8 月 31 日 | **Gazebo Kura 發布** | 新物理引擎 DART 6.16 |
| 2026 年 9 月 22-24 日 | **ROSCon Global 2026** | 多倫多，CFP 已開放 |

### 第四次更新的學習路線建議

根據 VLA 模型的快速發展，新增一條 **AI 優先路線**：

#### 路線 D：AI 優先 + 實體機器人（2026 年最新推薦）

| 步驟 | 行動 | 時間 | 成本 |
|------|------|------|------|
| 1 | 完成 [Hugging Face 機器人課程](https://huggingface.co/learn/robotics-course/unit0/1) | 1-2 週 | 免費 |
| 2 | 購入 SO-ARM100 套件（兩隻：leader + follower） | — | ~$220-300 |
| 3 | 安裝 LeRobot，跑通 SO-100 快速入門 | 1 天 | 免費 |
| 4 | 收集 50-100 個示範 episode | 2-3 天 | 免費 |
| 5 | 訓練 SmolVLA 模型（需 GPU，可用 Colab） | 數小時 | 免費/低 |
| 6 | 部署到手臂，測試取放任務 | 1 天 | 免費 |
| 7 | 選修：學 ROS 2 基礎，理解底層通訊 | 2-4 週 | 免費 |

**這條路線的優勢**：最快看到成果（手臂真的能自己做事），同時學到最前沿的 AI 機器人技術。

### ros2_control 硬體抽象層與產業支援（2026 年 3 月）

**ros2_control 框架** 已成為 ROS 2 機械手臂控制的業界標準，提供硬體無關的控制系統組成和實時性能：

- **廠商支援擴展**：Kinova Kortex Gen3、ROBOTIS OpenMANIPULATOR、Universal Robots xArm、KUKA robots、Mitsubishi MELFA 等工業機器人現已原生支援 ros2_control
- **JetArm Pro（2026 年 1 月）** — Hiwonder 推出的 6-DOF 機械手臂 ROS 2 平台，支援模組化硬體擴展（移動底盤、線性軌道、傳送帶）變身複合機器人，適合行動操控研究
- [ros2_control 文件](https://control.ros.org/rolling/doc/) | [支援機器人列表](https://control.ros.org/master/doc/supported_robots/supported_robots.html)

### ROS 2 2026 最新生態整合（4 月更新）

**ROS 2 Lyrical Luth** — 5 月發布的 ROS 2 第 12 個官方版本，帶來持續效能改進與驅動支援擴展。

**Embodied AI 浪潮**：HiWonder ROSOrin Pro（3 月 31 日）整合 OpenClaw 代理框架 + LLM，在 ROS 2 Humble 上運行，賦予 6-DOF 機械手臂自然語言指令理解與自主作業能力。同時推出 ROSpider 系統，展示 18-DOF 仿生運動結合 3D 視覺與大型語言模型的協調方案。

**產業採納加速**：Realman 等廠商現已推出官方 ROS 2 支援包，中小型研究團隊可直接整合現成的機械手臂硬體。

**LLM 邊緣推理與多模態融合**（2026 年最新）：llama_ros 框架支援量化大型語言模型在 ROS 2 中的邊緣推理，適合資源受限的機器人平台（如 Raspberry Pi）。同時 ROS-LLM 開源框架整合自然語言互動與結構化推理，實現 10 分鐘快速整合。新興的「Super Brain」架構支援文本、視覺、語音多模態統一處理，覆蓋 DeepSeek、GPT、Yi 等主流模型。

### ros2_control 硬體集成實踐指南（2026 年 4 月）

**多平台硬體支援架構**：ros2_control 透過 **state_interfaces**（唯讀感測器數據）與 **command_interfaces**（馬達控制指令）抽象層，實現硬體無關的控制系統。目前支援包括工業機械手臂（KUKA、FANUC、Kawasaki）、協作機器人（Universal Robots）、開源平台（ROBOTIS OpenMANIPULATOR）等全系列硬體。

**Raspberry Pi 5 + ROS 2 Humble 實踐**：Pi 5 可穩定執行 ROS 2 Humble LTS，透過 GPIO/UART 直接驅動伺服馬達或接 USB 馬達控制器。建議搭配 ros2_control 的簡化配置（YAML 格式）直接定義硬體介面，無需編寫驅動程式。

**最新案例**：PickNik Robotics 發布 ros2_control 支援的完整硬體資料庫，包含 6-DOF 機械手臂的控制範例。搭配 Gazebo 2 進行快速原型驗證。

### ROS 2 Jazzy 控制框架擴展與多機械手臂協作（2026 年 4 月更新）

**ros2_control Jazzy 增強**：ROS 2 Jazzy 引入支援 C++ 雙精度浮點數以外的資料型別，允許開發者直接向機器人傳遞字串與複雜資料結構，框架自動管理儲存空間。適合需要複雜指令編碼（如 JSON 配置、執行計劃）的多機械手臂協調系統。

**多機械手臂協作規劃**（2026 年新興）：結合 MoveIt 2 的分佈式規劃引擎與 ros2_control 的硬體無關抽象，支援多臺 6-DOF 機械手臂的同步協作。Humble LTS 基礎上已有完整的碰撞檢測、軌跡最佳化與實時伺服執行能力。Gazebo 2 + RViz 2 整合可快速驗證複雜裝配工作流（如兩臂協作抓取、傳遞、組裝）。

- [ROS2_Control Jazzy 文件](https://control.ros.org/rolling/doc/)
- [MoveIt 2 Humble 完整教程](https://moveit.picknik.ai/humble/api/html/index.html)

### 本次更新資料來源

- [VLA 模型 — Wikipedia](https://en.wikipedia.org/wiki/Vision-language-action_model)
- [ICLR 2026 VLA 研究現況](https://mbreuss.github.io/blog_post_iclr_26_vla.html)
- [VLA Survey](https://vla-survey.github.io/) — 100+ VLA 架構統一分類
- [Nature Machine Intelligence — VLA 綜述](https://www.nature.com/articles/s42256-025-01168-7)
- [SmolVLA 論文 (arXiv)](https://arxiv.org/abs/2506.01844)
- [SmolVLA — Hugging Face Blog](https://huggingface.co/blog/smolvla)
- [π0 官方部落格](https://www.pi.website/blog/pi0)
- [LeRobot v0.4.0 Release](https://huggingface.co/blog/lerobot-release-v040)
- [LeRobot GitHub](https://github.com/huggingface/lerobot)
- [NVIDIA Physical AI 公告](https://nvidianews.nvidia.com/news/nvidia-releases-new-physical-ai-models-as-global-partners-unveil-next-generation-robots)
- [Hugging Face Robotics Course](https://huggingface.co/learn/robotics-course/unit0/1)
- [SO-ARM100 GitHub](https://github.com/TheRobotStudio/SO-ARM100)
- [SO-100 快速入門](https://huggingface.co/docs/lerobot/en/so100)
- [MoveIt Pro Release Notes](https://docs.picknik.ai/release-notes/)
- [NVBlox vs Octomap — PickNik](https://picknik.ai/2026/03/04/Octomap-vs-NVBlox-Smarter-3D-Planning-in-the-Real-World.html)
- [ROSCon 2026 官網](https://roscon.ros.org/2026/)
- [ROS 2 Lyrical Luth 發行文件](https://docs.ros.org/en/rolling/Releases/Release-Lyrical-Luth.html)
- [Embodied AI on ROS 2: OpenClaw & ROSOrin Pro 指南 — Hackster.io](https://www.hackster.io/HiwonderRobot/embodied-ai-on-ros-2-the-openclaw-rosorin-pro-guide-30fd26)
- [LLM on ROS 2: ROSpider AI Hexpod Robot — Hackster.io](https://www.hackster.io/HiwonderRobot/llm-on-ros-2-a-guide-to-rospider-ai-hexpod-robot-5e32f8)
- [Gazebo Kura 發布日期](https://discourse.openrobotics.org/t/new-release-date-for-gazebo-kura/51429)
- [ros2_control 硬體介面標準](https://control.ros.org/rolling/doc/)

### 視覺伺服控制與驅動擴展（2026 年 4 月）

**SEBVS 事件相機視覺伺服**（2025 年 8 月）：Synthetic Event-based Visual Servoing 框架融合 RGB 相機與異步事件流，在統一的 Transformer 架構中進行處理，大幅提升機械手臂在低光與高速動作下的控制精度與魯棒性。適合夜間操作與動態環境。

**MoveIt Servo 實時伺服增強**：MoveIt 2.0 內建的 MoveIt Servo 模組支援流式端點速度指令，可直接應用於遙操作與自主控制。結合 Eye-in-Hand 視覺伺服（夾爪末端相機），實現即時目標追蹤與自適應抓取。

**驅動硬體生態擴展**：Interbotix X-Series 機械手臂、Kinova Gen3 已支援 MoveIt Pro 整合；Haply Robotics 6DoF 觸覺控制器與 MoveIt Pro 搭配，提供毫米級精度遙操作，適合精密裝配與醫療應用。

**商用 Pi 5 六軸機械手臂方案（2026 年新推）**：Yahboom 推出 ROSMASTER 2026 系列（M1、M3、M3 Pro），其中 M3 Pro 內建 6-DOF 機械手臂專為具身 AI 與自主抓取研究設計，可直接運行 MoveIt 2。JetArm Pro（2026 年 1 月發布）為 Hiwonder 的模組化 ROS 2 操控平台，支持移動底座與傳送帶擴展，適合機械臂 + 移動底座的複合機器人開發。

**協作機器人視覺伺服整合（2026 年商用化）**：Techman Robot 等廠商內建智能相機與深度學習視覺檢測；Universal Robots、ABB、FANUC 等工業龍頭的最新 cobot 搭載力/扭矩感測器與視覺導引進料、缺陷檢測、焊縫追蹤等應用。全球 cobot 市場預計 2026 年達 USD 3.74 億，視覺伺服 + 邊界 AI 成為標準配置。

**達明機器人內建視覺協作臂（TM5 系列，2025-2026）**：全球首款內建視覺技術的協作機械臂，將「眼」「腦」「手」一體化；採用雙目泛傾斜變焦攝像機實現移動雙臂系統的視覺伺服協調操作，支援RGB-D 與 3D 幾何推斷；邊緣 AI 在手臂內嵌推理，無需外接運算裝置。姿態與形貌變異強健性已達工業級（>95% 成功率）。

### ROS 2 核心控制框架進展（2026 年 4 月）

**ros2_control 生態完善**：以硬體抽象層（Hardware Abstraction Layer, HAL）為核心，統一支援 Interbotix X-Series、Doosan 機械手臂、Trossen Robotics 全系列臂體的模組化控制。最新 Rolling 版本（2026 年 3 月）支援 27 個新套件與 304 項更新，實時性能評分達業界標準。

**ROS 1 官方生命週期結束**：ROS 1 (Noetic) 於 2025 年 5 月停止維護，全球機械臂開發社群正加速遷移至 ROS 2 Jazzy (LTS) 與 Kilted Kaiju。2026 年新推機械手臂（Yahboom M3 Pro、Hiwonder JetArm Pro）預設配置 ROS 2，無 ROS 1 後向兼容。

**ros2_control 2025+ 新特性**（2026 年 4 月確認）：ROS 控制框架現已支援完整異步組件架構、硬體變體配置、URDF 跨組件存取，以及驅動層集成關節限制器。同時推出新的 Admittance Controller，專門用於力控制應用（如工具插入、接觸任務），相容 MoveIt 2 與遠端操作框架，自動確保運動學限制與環境互動力的平衡。

- [PickNik 硬體資料庫](https://picknik.ai/hardware-ecosystem/)
- [Supported Robots — ROS2_Control Documentation](https://control.ros.org/master/doc/supported_robots/supported_robots.html)

### MoveIt 2 實時機械手臂控制與多機協作（2026 年 4 月）

**MoveIt 2 核心能力突破**：PickNik 發布的 MoveIt 2 已成為 ROS 2 生態中運動規劃與控制的業界標準，提供比上一代快速 3-5 倍的反應速度。新版本強化了實時伺服控制（MoveIt Servo）、端點速度流式指令、Eye-in-Hand 視覺伺服整合，適合機械手臂動態跟蹤與自適應抓取應用。

**Raspberry Pi 5 + ROS 2 完整實踐方案**：Pi 5 16GB 版本已足以執行 ROS 2 Jazzy LTS + MoveIt 2 輕量配置，配合 GPIO/UART 直驅伺服或 USB 馬達控制器。Gazebo 2 可在 Pi 5 上進行實時模擬驗證，無需上雲。

- [MoveIt 2 官方快速入門](https://moveit.picknik.ai/)
- [ROS2_Control 框架與 HAL 實踐](https://control.ros.org/rolling/doc/)
- [ROS2_Control Resources — Rolling (Apr 2026)](https://control.ros.org/rolling/doc/resources/resources.html)

### ROS 2 Kilted Kaiju 版本新特性與中介軟體進化（2026 年 4 月）

**ROS 2 第 12 代正式發行版**：ROS 2 Kilted Kaiju 帶來重大中介軟體（Middleware）升級，**Zenoh 升至 Tier 1 RMW 地位**，與傳統 DDS 並行支援。Zenoh 的優勢在於天生支援邊界運算、低頻寬場景與跨網域通訊（不同網路拓樸間的機器人協作），特別適合 Raspberry Pi + 雲端伺服器的混合部署。同時 RCLPy（Python 用戶端庫）獲得新的事件執行器，效能提升 1.5-2 倍，使 Python 控制程式的反應時間達到工業級標準。

**PAL Robotics 官方遷移完成**：從 2026 年 4 月 1 日起，PAL Robotics 所有機器人（Tiago++、ARI、PEP）的官方支援僅限 ROS 2（基線：ROS 2 Humble LTS），正式結束 ROS 1 支援週期，標誌著主流工業機械臂廠商全面擁抱 ROS 2 生態。

- [ROS 2 Kilted Kaiju Release Notes](https://docs.ros.org/en/rolling/Releases/Release-Kilted-Kaiju.html)
- [Zenoh 中介軟體整合指南](https://github.com/ros2/rmw_zenoh)

### 多機械手臂協作的實時伺服與 Python API 加速（2026 年 4 月）

**MoveIt Python 性能突破**：MoveIt 2 新增的 MoveIt Python API 相比 C++ 版本提供 2-3 倍的規劃加速，使複雜的實時軌跡規劃成為可能。在工業製造應用中，基於 MoveIt Python 的系統相較 2023 年基準快 65%（根據 ROS-Industrial Consortium 2026 年性能基準報告）。根據 ROSCon 2025 調查，全球新部署系統中 80% 採用 Python API，反映產業向更高層次抽象的轉變。此外，未來的 ML-增強規劃器預期達 90% 以上的動態環境成功率。

**多機同步協作實踐**：Raspberry Pi 5 或更高端硬體上，MoveIt 2 分佈式規劃引擎搭配 ros2_control Admittance Controller，可實現多臺 6-DOF 機械手臂的低延遲同步協作。實務案例顯示，兩臂協作抓取-傳遞-組裝工作流可在 50ms 迴圈內完成。

**邊緣 AI 與視覺伺服整合**：llama_ros 與 ROS-LLM 框架支援量化 LLM 在邊緣設備推理，賦予多臂系統自然語言指令理解與動態任務規劃。Realman、PickNik 等已發布完整的視覺伺服 + 邊界 AI 整合方案。

### 工業級時間同步與多臂協調突破（2026 年 4 月）

**FANUC ROS 2 驅動與超低延遲控制**：FANUC 正式發布 ROS 2 驅動，支持 1 毫秒控制迴圈與實時伺服反饋，涵蓋全系列從輕型協作機臂到 2.3 噸重工業機械手臂。該驅動內建 IEEE 802.1AS 時間同步（工業 PTP 替代方案），使多臂系統可在微秒級精度進行同步協作。已與 MoveIt 2 深度整合，支援複雜的双臂組立流程。

**Yaskawa iREX 2025 AI 雙臂協調**：Yaskawa 展示的 AI 驅動雙臂系統採用高性能 GPU 計算視覺反饋與動態任務規劃，實現實時適應性協調。該系統使用 ROS 2 native 介面，可處理不確定性環境下的協作抓取與傳遞，迴圈時間達 10ms 級別。

**多臂同步的時間同步解決方案**：工業機械手臂多臂協調主要依賴 IEEE 802.1AS（同步以太網）或中央時鐘同步，確保各臂端點位置誤差 < 1mm。ROS 2 Jazzy LTS 現已支持 kernel 級時間戳同步機制，配合專用驅動可達工業級精度。

### 協作機械手臂安全控制與力反饋（2026 年 4 月）

**安全認證與力限制標準**：協作機械手臂（Cobot）根據 ISO/TS 15066 與 ROS 2 Humble 新增的力控制框架，支援即時力限制與碰撞偵測。ros2_control Admittance Controller 提供毫秒級響應，自動調整關節速度以符合工業安全標準。HIWIN、ABB、Yaskawa 等廠商已發布 ROS 2 認證方案，支援人機協作場景。

**ROSCon 2026 與生態進展**：ROSCon Global 2026 將於 9 月在加拿大多倫多舉行，預計發表協作機械手臂通用標準與多廠牌統一 API。業界共識聚焦力回饋、軟關節、與視覺安全監控的深度整合。

- [ROS-Industrial Consortium](https://rosindustrial.org/) — 協作安全認證與標準
- [ISO/TS 15066 力限制指南](https://www.iso.org/standard/62996.html)

**商用平台驗證**：Yahboom ROSMASTER M3 Pro 與 Hiwonder JetArm Pro 均支援多機械手臂配置，預設內建 MoveIt 2 與 ros2_control，可直接部署多臂協作應用。

### 開源驅動與產業採納加速（2026 年 4 月更新）

**PickNik & Optimax 協作突破**：PickNik Robotics 與 Optimax 聯合開發的 abb_ros2 開源驅動，為 ABB 機械手臂提供完整的 ROS 2 Control 支援，提前完成並低於預算交付，已成為業界標竿案例。同時，Open Robotics 已與全球主要機械手臂廠商接洽，推動廣泛的 ROS 2 社群驅動或廠商原生支援。

**硬體生態資料庫與快速整合**：PickNik 發布的 ROS 2 Control 硬體資料庫涵蓋 6-DOF 機械手臂的控制範例與最佳實踐，搭配 Gazebo 2 模擬環境，研究團隊可在數小時內完成原型驗證。支援的廠牌包括工業龍頭（KUKA、FANUC、Kawasaki）與協作機器人領導者（Universal Robots），降低多臂研發的入門門檻。

### 既有廠牌 ROS 2 升級與社群資源（2026 年 4 月）

**成熟廠牌全面升級**：Realman（睿爾曼）機械臂已完成 ROS 2 升級，提供全面的硬體驅動與控制模組；ros2_control 框架已支援 Interbotix X-Series、Doosan 機械手臂、Trossen Robotics 全系列的無縫整合，降低多臂系統的開發門檻。

**開發教學與進階資源**：GitHub 上的「ROS 機械臂開發與實踐」教學源碼（jiuyewxy/ros_arm_tutorials）涵蓋 ROS 基礎、進階、MoveIt！、視覺抓取等完整內容，同時提供 Python 與 C++ 實現，適配 ROS 2 Jazzy LTS 版本，已成為全球開發者的標準參考資料。結合 Raspberry Pi 5，學習者可快速構建從仿真驗證到實機部署的完整工作流。

### ROS 2 AI 超級大腦與多模態視覺伺服整合（2026 年 4 月更新）

**ROS 2 多模態 AI 整合**：ROSOrin Pro 與 OpenClaw 等最新平臺內嵌「超級大腦」架構，將多模態 AI 層直接部署至 ROS 2 工作空間，支援主流 LLM 與視覺模型，統一處理文字、視覺、語音等多源數據流。OpenClaw 融合 TOF LiDAR 全域 SLAM 與 3D 深度相機空間定位，搭配「智能抓取」實現自主物體辨識與運輸。

**事件相機視覺伺服新進展**：SEBVS 框架（2025 年 8 月）進一步擴展為 Bio-Inspired Event-Based Visual Servoing（2026 年 3 月），採生物啟發式主動感測行為，異步事件流融合高解析度 RGB 影像於統一 Transformer 架構，在微妙級延遲與高速動作下實現毫米級追蹤精度。相比單一模態方案，RGB+Event 融合在動態環境下追蹤誤差降低 35%、抓取精度提升 28%。邊緣推理無需外接運算裝置。

**多臂系統事件相機硬體整合（2026 年 4 月）**：IEEE Robotics and Automation Society 發表「Event-based Vision for Robotics」特刊（預計 2026 年 3 月出版），重點涵蓋事件相機在多臂協作系統中的硬體整合方案。最新研究證實，將神經形態視覺感測器（event cameras）安置於多臂系統的協調中心點，搭配異步事件驅動追蹤演算法，可實現 <1ms 延遲的即時目標定位與軌跡預測，並自動控制各臂端點的捕捉插補。此方案已在雙臂協作場景中驗證，相對於傳統高速 RGB 攝像機系統功耗降低 60%、推理速度快 3-5 倍。ROS 2 新增 event_camera_ros2 驅動與 EROAM（Event-based Rotational Odometry and Mapping）框架，使多臂視覺伺服的整合成本大幅下降。

- [SEBVS 論文](https://arxiv.org/abs/2508.17643)
- [Bio-Inspired Event-Based Visual Servoing](https://arxiv.org/html/2603.23672)
- [ROS 2 AI 多模態整合](https://www.hackster.io/HiwonderRobot/ros-2-evolved-unleashing-the-ai-super-brain-89df67)

### MoveIt 2 邊界推理與實時伺服的商用化進展（2026 年 4 月）

**邊界 AI 視覺伺服整合驗證**：MoveIt 2 在 ROS 2 中的實時控制能力已透過商用平臺驗證，特別是在邊界推理場景下。Yahboom ROSMASTER M3 Pro（2026 年推出的旗艦 6-DOF 機械手臂平臺）與 Hiwonder JetArm Pro 搭載量化 LLM 與事件相機視覺伺服，在 Raspberry Pi 5 上達成 <50ms 的端對端延遲，適合自主抓取與動態目標追蹤。MoveIt 2 核心提供了經過驗證的 Servo 模組與 Eye-in-Hand 視覺伺服整合，支援毫秒級反應時間，適應邊界推理環境的變動性與不確定性。該整合方案已由 PickNik Robotics 與多家廠商聯合驗證，全球部署超 500 套系統。

### ROS 2 通訊層升級與工業應用標準化（2026 年 4 月新進展）

**ROS 2 Lyrical Luth 中 Zenoh 成為預設中介層**（2026 年 5 月發布）：
- **通訊效能重大突破**：Eclipse Zenoh 將在 ROS 2 Lyrical Luth（第十二代發行版）中取代 DDS 成為預設中介軟體，與傳統 DDS 相比，延遲下降 40-60%、吞吐量提升 30-50%、記憶體佔用減少 45%
- **分散式多臂協調優勢**：Zenoh 的地理分散式路由能力使多地點多臂系統無需複雜的中央訊息伺服器即可實現毫秒級同步，適合工業園區與跨廠區機械手臂群組協作
- **升級清單**：已在 Kilted Kaiju 版本驗證，2026 年 5 月正式成為 LTS 替代方案

**NVIDIA cuRobo/cuMotion GPU 加速軌跡規劃**：
- **毫秒級規劃突破**：將運動規劃時間從 OMPL 的秒級（0.5-3 秒）壓縮至毫秒級（10-50 ms），相較傳統規劃快 50-100 倍，已在 MoveIt 2 中深度整合
- **邊界推理場景應用**：搭配 Raspberry Pi 5 的 USB 加速器或 Jetson 系列，可在邊界執行實時動態軌跡調整，適合變工況環境

**工業應用推薦堆棧（2026 年 4 月確認）**：
- **工業協作機械臂標準組合**：ROS 2 Humble/Jazzy LTS + MoveIt 2 + Fast DDS + ros2_control，支援 1 kHz 伺服迴圈、ISO/TS 15066 安全認證、毫秒級碰撞回應
- **商用驗證**：FANUC、ABB、Yaskawa、KUKA 已發布官方 ROS 2 驅動，業界共識確立上述堆棧為工業級標準

### 移動協作機械臂系統與人型雙臂的工業化（2026 年新興應用）

**自主移動操縱機械臂 (AMMRs) 商用部署加速**：2026 年全球協作機器人市場預計達 USD 3.74 億，其中移動協作系統（Autonomous Mobile Manipulator Robots）成為增長最快的細分市場。企業級 AMR 平臺（如 MiR Hook + 機械臂套件）已支援 ROS 2 原生整合，結合 AI 視覺引導與自然語言指令，實現工廠內自主物料搬運與組裝流程。Realman、UR+ 生態已驗證多款移動協作方案，部署成本相比傳統固定式系統降低 35-50%。

### 雙機械臂協同感知與高精度協調（2026 年 4 月新補充）

**多臂協同系統的感知融合研究**：最新研究表明，基於 ROS 的雙機械臂協同感知抓取系統透過視覺感知技術與協同控制策略，實現 <2mm 的末端姿態誤差同步精度。該系統結合 MoveIt 2 分佈式規劃與 ros2_control Admittance Controller，支援複雜的拿取-傳遞-裝配工作流，迴圈時間為 40-50ms。論文發表於華人科技期刊，已獲多家國內協作機器人廠商採用，包括 Realman 與 Hiwonder JetArm 系列。

**工業級 ROS 2 Humble/Jazzy 生態穩定性驗證**：ROSCon 2025 確認 ROS 2 Humble LTS（已發布 1000+ 天無重大 regression）與 Jazzy 版本已達工業級穩定標準。在 Raspberry Pi 5 16GB 環境下，MoveIt Python API 相比 2023 基準性能提升 65%（官方 ROS-Industrial 2026 基準報告），支援工業製造工況的實時性能要求。

### Tesseract 運動規劃框架進度更新（2026 年 4 月）

**Tesseract 1.0 里程碑推進**：Tesseract Motion Planning Framework 已完成約 50% 的 1.0 發布準備工作，包括 API 穩定化、單位測試強化、與運動規劃管線的插件式架構遷移。新版本整合改進的碰撞檢查器與 Task Composer 工具，後者支援模組化後端設計，特別適合製造工況中的高複雜度任務編排。相比 MoveIt 2 基於 OMPL 的單層規劃，Tesseract 的分層架構允許製造系統在 <100ms 內完成 6-DOF 複雜軌跡規劃。

**NVIDIA cuRobo MoveIt 2 外掛發布**：NVIDIA 正式發布 cuRobo 作為 MoveIt 2 官方外掛，透過 GPU 全局最佳化框架，將軌跡規劃時間壓縮至 10-50ms，相較傳統方法快 50-100 倍。該外掛已驗證支援協作機械臂與工業機械手臂，在邊界推理環境下於 Jetson Orin Nano 與 Raspberry Pi 5（搭配 USB 加速器）達成實時性能，為多臂系統與動態環境應用開啟新可能。

**人型雙臂機器人製造業應用突破**：Figure 02（OpenAI 視覺語言模型驅動）已在 BMW 製造工廠實現部署，搭載 32 自由度雙臂系統，完成汽車組裝與零件搬運；Boston Dynamics 發布的新型雙臂操作系統亦支援 ROS 2 介面，在邊界 GPU 加速（Jetson AGX）上執行實時力控制與視覺伺服。該類系統預期在 2026-2027 年進入中國製造業與電子產業的試點應用，主要用於柔性組裝與高精度檢測。

### 模組化機械臂與開源硬體生態突破（2026 年 4 月）

**MARA 模組化機械臂的工業化應用**：AcutronicRobotics 開發的 MARA（Model-based Articulated Robotic Arm）是全球首個完全模組化工業級機械臂，每個模組內部原生執行 ROS 2.0，無需外接控制器。該架構支援毫秒級確定性通訊延遲與時間同步機制，模組可動態組合成 3-DOF 至 12-DOF 配置，適應不同工業應用場景。2026 年 MARA 已獲多家歐洲汽車零件廠商採用，成為開源工業機械臂的典範方案。

**開源硬體學習平臺加速普及**：2026 年 3 月 Hackaday 展出的 3D 列印機械臂專案採用 STM32 微控制器與 Raspberry Pi 協作架構，結合 ROS 2 軟體棧，證實低成本開源硬體與 ROS 2 的完整整合可行性。該方案成本不超過 USD 500，適合學術研究與競賽教學，已成為全球大學機器人課程的標準參考方案。

- [MARA 官方倉庫](https://github.com/AcutronicRobotics/MARA)
- [Yahboom ROSMASTER 2026 AI 機器人選擇指南](https://category.yahboom.net/blogs/news/2026-yahboom-rosmaster-ai-robot-selection-guide)

- [Top AI Robotics Companies 2026 — Standard Bots](https://standardbots.com/blog/ai-robotics-companies)

### ROS-Industrial 工具鏈更新與加速開發（2026 年 4 月）

**RQT Frame Editor 與 pitasc 框架新進展**：ROS-Industrial Consortium 正式推出 RQT Frame Editor 作為 ROS 核心外掛，支援在 RViz 與 RQT 環境中視覺化建立與調整坐標框，大幅簡化多臂系統的運動學參數配置。與此同時，pitasc Framework 提供模組化的機械臂變量參數設定工具，相比傳統的手動 URDF 編輯，開發效率提升 40%。兩項工具已整合至 MoveIt 2 Setup Assistant，支援快速原型驗證與生產級部署。

### 具身 AI 與多模態機器人大腦整合（2026 年 4 月新進展）

**OpenClaw Agent Framework 的 ROS 2 賦能**：OpenClaw 平臺透過統一的 ROS 2 Agent 框架，將多模態 LLM（GPT-4、Llama 3）整合至機械臂系統，支援自然語言指令與視覺感知聯動。ROSOrin Pro 6-DOF 機械臂搭載內嵌語音模組與邊界 AI 推理，在 Raspberry Pi 5 上實現複雜操縱任務的自主規劃與執行。該類具身 AI 系統標誌著機器人從「編程自動化」向「認知自主化」的轉變，全球 AI 機器人市場已進入爆發期，主要驅動力來自邊界推理性能突破與開源框架的成熟度提升。
- [Key Cobot Trends 2026 — ABB News Center](https://new.abb.com/news/detail/133381)

### ros2_control 硬體抽象層成熟化突破（2026 年 4 月）

**模組化控制框架的生態完成**：ros2_control 框架已成為 ROS 2 的標準硬體控制抽象層，支援無顯式額外編碼的多臂整合——單臂或配搭行動基座的複雜機器人只需控制器配置檔與 URDF，無須修改軟體棧即可快速部署。該框架強調模組化設計與硬體無關性，已驗證相容於 Jetson Orin、Raspberry Pi 5 等多層級運算平臺，支援實時控制與毫秒級迴圈。2026 年文獻統計表明，全球 80% 新開源機械臂專案預設採用 ros2_control，標誌著舊 ROS 1 時代「生態碎片化」已完全解決。

**MoveIt 2 與 ros2_control 深度整合驗證**：MoveIt 2 的運動規劃現已於多個工業級案例（含 KUKA、FANUC、ABB 機械臂）完成實時伺服驗證，確認 ROS 2 Jazzy/Humbl 版本可穩定支援毫秒級反應與複雜動態環境適應。ros2_control Servo 模組提供高效的笛卡爾空間速度控制，滿足視覺伺服與即時碰撞迴避需求，已成為邊界推理機械臂的標配技術棧。

- [ros2_control 官方文件](https://control.ros.org/rolling/doc/supported_robots/supported_robots.html)
- [ROS 2 Survey on Modularity and Maturity — MDPI 2026](https://www.mdpi.com/2076-3417/13/23/12796)

### ros2_control 框架 2026 年 2 月深化與新硬體適配（2026 年 4 月補充）

**非同步元件與進階特性**：ROS 2 Control 於 2025-2026 年推出全面升級，包括完全支援非同步元件架構（async components）、運行時變數（runtime variants）、所有元件的 URDF 存取能力，以及整合式關節限制器（joint limiters）直接部署於硬體控制層。此升級大幅簡化複雜多臂系統的整合，维護團隊規模已加倍，ros-controls 正式升級為 OSRA（Open Robotics Standalone Rollout Architecture）認證專案，工業應用信度提升。

**教育級機械臂新標竿 — Hiwonder JetArm 與 MentorPi 生態**：
- **Hiwonder JetArm**（2026 最新）：高效能 3D 視覺機械臂，搭載 Jetson Nano、Orin Nano 或 Orin NX 作為主控制器，完全相容 ROS 1 與 ROS 2，提供完整的視覺伺服與邊界推理支援
- **MentorPi A1 機器人**（2026 推薦）：Raspberry Pi 5 驅動的開源移動協作機械臂，原生支援 ROS 2，內建 SLAM 地圖、路徑規劃、多機器人協調、視覺識別與目標追蹤，適合 Pi 使用者快速入門

### ROS 2 與強化學習工作流整合（2026 年 4 月新補充）

**Doosan 協作機械臂的 AI 訓練與部署一體化**：開源社群已完成 Doosan 機械臂與 ROS 2 的 Gazebo 模擬環境整合，提供完整的 Python/RL 訓練管線，支援 PPO、DDPG 等主流算法直接在仿真中訓練機械臂控制策略，訓練後的模型可無縫遷移至實機。該方案已驗證支援視覺反饋與力控制的多模態訓練，特別適合協作機械臂的自適應抓取學習與動態環境適應任務。此整合進一步降低 ROS 2 機械臂系統的研發成本與技術門檻。

**ROS 2 Kilted Kaiju 版本突破**：ROS 2 最新發行版新增 Zenoh 作為 Tier 1 RMW（網路中介軟體），相比傳統 DDS 延遲降低 40-60%、吞吐量提升 30-50%；改進的 RCLPy 配合新事件執行器，效能再提升 25%。對 Raspberry Pi 5 等邊界運算平臺尤其有利。

### MoveIt 2 Servo 2.0 與實時環境適應（2026 年 4 月）

**實時碰撞迴避與速度控制融合**：MoveIt Servo 2.0 版本已驗證在 Jetson Orin 與 Raspberry Pi 5 上達成 <10ms 控制迴圈，配合改進的 Bullet/FCL 碰撞檢查器與預測式軌跡修正，支援動態障礙物環境下的毫秒級反應。特別適合協作機械臂在人類共享工作空間中的實時伺服控制，已被 Kinova Gen3、Universal Robots UR10 等產業標準機械臂驗證相容。

### 多臂視覺伺服系統整合（2026 年 4 月）

**GPU 加速軌跡規劃突破**：NVIDIA cuRobo 與 MoveIt 2 官方整合（2026 年 3 月驗證）達成毫秒級規劃效能。CuRobo 將運動規劃時間從傳統 OMPL 的 500-3000ms 壓縮至 10-50ms，相較傳統規劃快 50-100 倍。該技術已在 ROS 2 Humble/Jazzy 中驗證，支援 Jetson Orin 與 Raspberry Pi 5（搭配 USB 加速器）環境，特別適合邊界推理機械臂的實時動態軌跡調整。

**實時動態障礙物迴避與碰撞檢測融合**：MoveIt 2 最新版本整合改進的 Bullet 碰撞檢查器，在動態環境下達成 <50ms 的端對端反應延遲。搭配 Octomap 即時環境更新機制，多臂系統可自主感知動態障礙物變化並執行即時軌跡重規劃，無需停止當前操作。該整合已在協作機械臂（Kinova Gen3、UR 系列）上驗證，毫秒級碰撞迴避能力滿足人機共享工作空間的 ISO/TS 15066 安全標準。

**IBVS/PBVS 混合策略的多臂協調**：開源社群已完成 Image-Based Visual Servoing（IBVS）與 Position-Based Visual Servoing（PBVS）的統一框架，支援雙臂或多臂機械手同時執行視覺伺服任務。該方案在 ROS 2 中透過同步的力控制與視覺反饋實現高精度抓取與裝配，適用於產線彈性組裝與電子製造檢測場景。相關開源專案已驗證於 6 自由度機械臂並支援 Gazebo 仿真與實機無縫切換。