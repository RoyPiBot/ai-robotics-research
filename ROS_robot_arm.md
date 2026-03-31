# ROS 與機械手臂入門指南

> 撰寫日期：2026-03-31（第四次更新）
> 對象：完全新手，從零開始
> 環境：Raspberry Pi 5 (16GB RAM)

---

## 目錄

1. [ROS 是什麼](#1-ros-是什麼)
2. [機械手臂基本原理](#2-機械手臂基本原理)
3. [如何在樹莓派上安裝 ROS 2](#3-如何在樹莓派上安裝-ros-2)
4. [簡單範例程式](#4-簡單範例程式)
5. [推薦學習路線](#5-推薦學習路線)
6. [**附錄 E：機械手臂進階篇**](#附錄-e機械手臂進階篇--從入門到專業2026-03-31-新增) ← 🆕 新增

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
- [Gazebo 模擬器](https://gazebosim.org/)
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
- [ROS-Z — Rust + Zenoh 原生 ROS 2 實作](https://fosdem.org/2026/schedule/event/BQ8DVM-ros-z/)（FOSDEM 2026 發表）
- [Meta-ROS — 下一代中介層，吞吐量比標準 ROS 2 高 30%](https://arxiv.org/html/2601.21011v1)
- [ROS 2 Kilted Kaiju Release Notes](https://docs.ros.org/en/rolling/Releases/Release-Kilted-Kaiju.html)
- [ros-realtime Pi 映像](https://github.com/ros-realtime/ros-realtime-rpi4-image) — Pi 3/4/5 預裝 ROS 2 + Linux RT

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
- Pi 5 + Ubuntu 22.04 + ROS 2 Humble 是 2026 年最常見的配置
- NEMA17 步進馬達可搭配 ros2_control 用於手臂關節

### 2026 年硬體選擇建議更新

| 方案 | 處理器 | 適合場景 | 參考價格 |
|------|--------|----------|----------|
| Pi 5 + ROS 2 Humble | ARM Cortex-A76 | 教育、輕量應用 | ~$80 USD |
| Jetson Nano + ROS 2 | Maxwell GPU | 需要 AI 推理加速 | ~$150 USD |
| Pi 5 + USB 加速器 | ARM + Coral TPU | 邊緣 AI + 機器人 | ~$130 USD |

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

### GitHub 上值得關注的新專案

| 專案 | Stars | 說明 |
|------|-------|------|
| [enactic/openarm](https://github.com/enactic/openarm) | ★2,017 | 7-DOF 仿人手臂，MuJoCo/Isaac Sim |
| [norma-core/elrobot](https://github.com/norma-core/norma-core/tree/main/hardware/elrobot) | 新 | 7+1 DOF 全 3D 列印，Physical AI 研究用 |
| [PCrnjak/PAROL6](https://github.com/PCrnjak/PAROL6-Desktop-robot-arm) | 活躍 | 工業風格 6-DOF，ROS 2 + MoveIt 2 整合 |
| [Jelatine/mockway_robotics](https://github.com/Jelatine/mockway_robotics) | ★100 | 開源六軸協作機械臂（含機構、電路、軟體） |
| [msf4-0/so101_ros2](https://github.com/msf4-0/so101_ros2) | 新 | SO101 手臂 ROS 2 + MoveIt 整合 |
| [ros2-jazzy-6-axis-arm-planning](https://github.com/ShaotianZhou/ros2-jazzy-6-axis-arm-planning) | 新 | ROS 2 Jazzy 6 軸手臂 RRT* 路徑規劃 |

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

---

## 附錄 E：機械手臂進階篇 — 從入門到專業（2026-03-31 新增）

> 本篇適合已掌握基礎 ROS 2 操作、理解正/逆向運動學概念、並跑通 MoveIt 2 簡單範例的讀者。
> 涵蓋十大進階主題，帶你從「會動」進化到「會思考、會感知、會協作」的機械手臂。

### 目錄（進階篇）

- [E.1 進階運動規劃演算法](#e1-進階運動規劃演算法)
- [E.2 力/力矩控制與柔順控制](#e2-力力矩控制與柔順控制)
- [E.3 抓取規劃](#e3-抓取規劃)
- [E.4 ros2_control 進階用法](#e4-ros2_control-進階用法)
- [E.5 強化學習應用於操控](#e5-強化學習應用於操控)
- [E.6 Sim-to-Real 模擬到現實](#e6-sim-to-real-模擬到現實)
- [E.7 雙臂與多臂協調](#e7-雙臂與多臂協調)
- [E.8 移動式操控（Mobile Manipulation）](#e8-移動式操控mobile-manipulation)
- [E.9 數位孿生（Digital Twin）](#e9-數位孿生digital-twin)
- [E.10 安全標準與協作機器人規範](#e10-安全標準與協作機器人規範)

---

### E.1 進階運動規劃演算法

入門篇提到 MoveIt 會幫你做路徑規劃，但背後到底用了什麼演算法？當你需要更快、更平滑、或在更複雜的環境中規劃路徑時，理解這些演算法就非常重要了。

#### 兩大流派

| 流派 | 核心思想 | 代表演算法 | 優點 | 缺點 |
|------|---------|-----------|------|------|
| **取樣式（Sampling-Based）** | 在組態空間中隨機取樣，建立連通圖 | RRT, RRT*, RRT-Connect, PRM | 能處理高維度空間、不需要梯度資訊 | 路徑可能不平滑、非最優 |
| **最佳化式（Optimization-Based）** | 將路徑視為函數，用數學最佳化求解 | STOMP, CHOMP, TrajOpt | 路徑平滑、可加入多種約束 | 需要初始猜測、可能陷入局部最優 |

**業界最佳實踐（2026 年）**：混合使用！先用取樣式（如 RRT-Connect）快速找到一條可行路徑，再用最佳化式（如 STOMP）打磨出平滑軌跡。

#### 主要演算法詳解

**RRT（Rapidly-exploring Random Tree）**
> 比喻：像一棵樹從起點往四面八方生長，直到其中一根枝條碰到終點。

- RRT 是最基礎的版本——找到路徑就停
- **RRT\*** 多了一步「重新佈線（rewire）」，隨著取樣越多，路徑越接近最優
- **RRT-Connect** 從起點和終點同時生長兩棵樹，相遇即完成，速度快很多
- **RRT\*-Connect**（2025 新論文）：結合雙向搜索的速度和 RRT\* 的最優性保證

**PRM（Probabilistic Roadmap）**
> 比喻：先在地圖上畫很多路標和道路（建構階段），之後不管要從哪到哪，都能快速查路線（查詢階段）。

- 適合同一個環境要反覆規劃不同起終點的情況
- 建構階段比較慢，但查詢階段極快

**STOMP（Stochastic Trajectory Optimization for Motion Planning）**
> 比喻：先畫一條直線從起點到終點，然後隨機「抖動」這條線很多次，留下最好的抖法。

- 不需要計算梯度（適合碰撞約束不可微分的情況）
- MoveIt 2 已內建支援

**CHOMP（Covariant Hamiltonian Optimization for Motion Planning）**
> 比喻：像 STOMP，但用梯度下降而非隨機抖動來優化路徑。

- 需要障礙物的距離場（SDF）來計算梯度
- 收斂速度比 STOMP 快，但對初始路徑更敏感

#### MoveIt 2 中使用 STOMP 的設定範例

```yaml
# stomp_planning.yaml
planning_plugin: stomp_moveit/StompPlanner
request_adapters: >-
  default_planner_request_adapters/AddTimeOptimalParameterization
  default_planner_request_adapters/FixWorkspaceBounds
stomp:
  num_timesteps: 60          # 軌跡離散點數
  num_iterations: 40         # 最佳化迭代次數
  num_iterations_after_valid: 0  # 找到可行解後再優化幾次
  num_rollouts: 30           # 每次迭代的隨機樣本數
  max_rollouts: 30
  initialization_method: 1   # 1 = 線性內插
```

#### 深度強化學習（DRL）規劃器的崛起

2025 年的基準測試顯示，使用 **SAC（Soft Actor-Critic）** 訓練的 DRL 規劃器在 UR3e 機器人上已經能達到：
- **更高的成功率**和**更低的規劃時間**（對比 OMPL 傳統規劃器）
- 產生更緊湊、更確定性的軌跡
- 訓練資料集：100,000+ 條無碰撞軌跡

**ML 增強的規劃器**預計到 2026 年底在動態環境中達到 90%+ 的成功率。

#### 進階運動規劃參考資源

- [OMPL 官方網站](https://ompl.kavrakilab.org/) — 23 種取樣式規劃器
- [MoveIt 2 規劃器文件](https://moveit.ai/documentation/planners/)
- [STOMP in MoveIt 2（PickNik 教學）](https://picknik.ai/moveit%202/ros/2023/05/19/optimization-based-planning-with-stomp.html)
- [RRT\*-Connect 論文（2025）](https://arxiv.org/html/2505.10542v1)
- [DRL vs 取樣式規劃器基準測試（2025）](https://pmc.ncbi.nlm.nih.gov/articles/PMC12431259/)

---

### E.2 力/力矩控制與柔順控制

入門篇中，機械手臂只是「移動到指定位置」——這叫**位置控制**。但真實世界中，你常常需要手臂能**感知力量**並做出反應。例如：打磨表面時要維持穩定力道、組裝零件時不能硬塞、與人協作時碰到人要立即停下。

#### 三種控制模式

| 控制模式 | 輸入 | 輸出 | 適用場景 | 硬體需求 |
|---------|------|------|---------|---------|
| **位置控制** | 目標位置 | 移動指令 | 搬運、取放 | 基礎編碼器 |
| **阻抗控制（Impedance）** | 目標位置 + 虛擬彈簧阻尼 | 力矩指令 | 精密接觸任務 | 力矩可控馬達 |
| **導納控制（Admittance）** | 力/力矩感測器讀數 | 位置修正 | 工業協作、組裝 | 力/力矩感測器 |

用比喻來理解：
- **位置控制**：像一把鎖定的尺——不管碰到什麼，都硬要移到指定位置
- **阻抗控制**：像一根彈簧——離目標越遠，施力越大，但碰到阻礙會自然退讓
- **導納控制**：像有人推你的手，你感覺到力量後主動讓開——感測力量，修正位置

#### 阻抗控制數學（簡化版）

```
F = M × a + B × v + K × x

F = 施加的力（牛頓）
M = 虛擬質量（越大，反應越慢、越穩定）
B = 虛擬阻尼（越大，運動越平緩）
K = 虛擬剛度（越大，越硬、越像位置控制）
x = 離目標的偏差
v = 速度
a = 加速度
```

調整 M、B、K 就能改變手臂的「手感」：
- **高 K、低 B**：手臂很硬，精確回到目標位置
- **低 K、高 B**：手臂很軟，被推開後慢慢回來
- **K = 0**：純力控制，維持特定力量而不在乎位置

#### ROS 2 實作：導納控制器

ros2_controllers 提供了生產等級的 `admittance_controller`，支援 ROS 2 Jazzy / Rolling：

```yaml
# admittance_controller 設定範例
admittance_controller:
  ros__parameters:
    joints:
      - joint_1
      - joint_2
      - joint_3
      - joint_4
      - joint_5
      - joint_6
    command_interfaces:
      - position            # 輸出位置修正指令
    state_interfaces:
      - position
      - velocity
    ft_sensor:
      name: ft_sensor       # 力/力矩感測器名稱
      frame:
        id: tool0           # 感測器的座標系
    admittance:
      # 每個自由度（x, y, z, rx, ry, rz）的虛擬質量
      mass: [10.0, 10.0, 10.0, 5.0, 5.0, 5.0]
      # 阻尼比（1.0 = 臨界阻尼，不會震盪）
      damping_ratio: [1.0, 1.0, 1.0, 1.0, 1.0, 1.0]
      # 虛擬剛度（N/m 或 Nm/rad）
      stiffness: [200.0, 200.0, 200.0, 50.0, 50.0, 50.0]
```

#### 2025 年新進展：自適應質量導納控制

2025 年 4 月的論文提出了**質量自適應導納控制（Mass-Adaptive Admittance Control）**：
- 即時估算手臂抓取的物體質量
- 動態調整導納控制參數，補償未知負載
- 例如：手臂不知道拿的杯子是空的還是裝滿水的，都能穩定操作

#### 力/力矩控制參考資源

- [ROS 2 Admittance Controller 官方文件](https://control.ros.org/iron/doc/ros2_controllers/admittance_controller/doc/userdoc.html)
- [Force Torque Sensor Broadcaster](https://control.ros.org/master/doc/ros2_controllers/force_torque_sensor_broadcaster/doc/userdoc.html)
- [笛卡爾阻抗控制器（GitHub）](https://github.com/matthias-mayr/Cartesian-Impedance-Controller)
- [質量自適應導納控制（2025 論文）](https://arxiv.org/html/2504.16224v1)
- [柔順力控制綜述（2025）](https://www.mdpi.com/2227-7390/13/13/2204)

---

### E.3 抓取規劃

「移動到物體旁邊」只是第一步，「穩穩地抓起來」才是真正的挑戰。抓取規劃（Grasp Planning）要決定：手該從哪個角度接近、哪些手指該碰哪裡、施多少力。

#### 從分析式到學習式的演進

| 世代 | 方法 | 代表工具 | 原理 |
|------|------|---------|------|
| **第一代** | 分析式 | GraspIt! | 用幾何和力學公式計算最佳抓取點 |
| **第二代** | 深度學習 | Dex-Net 2.0 | 從合成點雲資料訓練CNN，預測抓取品質 |
| **第三代** | 通用抓取 | AnyGrasp / AnyDexGrasp | 不限機械手形狀，泛化到任意物體 |

#### 主要工具與框架（2025-2026）

| 工具 | 特色 | 狀態 |
|------|------|------|
| **GraspIt!** | 經典開源抓取模擬器，用力學品質指標分析 | 基準線，逐漸被學習方法超越 |
| **Fast-Grasp'D** | 比 GraspIt! 快 10 倍，產生更穩定的接觸抓取 | 活躍開發（Grasp'D-1M 資料集） |
| **Dex-Net 2.0** | 深度學習 + 合成點雲，穩健的平行夾爪抓取 | 基礎工作，已衍生多版本 |
| **AnyGrasp** | 在空間和時間域都穩健的抓取感知 | 發表於 IEEE TRO |
| **AnyDexGrasp**（2025） | 適用於任何機械手的通用抓取模型，150+ 新物體達 75-95% 成功率 | 最先進 |
| **DexGraspNet 2.0** | 大規模資料集 + 可微分力封閉估算 | 研究前沿 |
| **DexGrasp-Anything**（2025） | 整合物理感知約束到訓練和推理 | 最新研究 |

#### AnyDexGrasp 的兩階段方法

AnyDexGrasp 是目前最先進的通用抓取系統，其核心是**兩階段解耦**：

1. **通用抓取模型**：輸入場景幾何（點雲），輸出以「接觸點」為中心的抓取表示——與具體的機械手無關
2. **手專屬決策模型**：透過真實世界的反覆試錯，學習如何將通用抓取表示轉換為特定機械手的動作

這種設計讓同一個通用模型可以搭配不同的機械手（平行夾爪、三指手、靈巧手），只需訓練手專屬的第二階段。

#### 抓取規劃參考資源

- [學習式靈巧抓取綜述（2025, Springer）](https://link.springer.com/article/10.1007/s10462-025-11262-2)
- [AnyDexGrasp 官網](https://graspnet.net/anydexgrasp/)
- [AnyGrasp（IEEE TRO）](https://ieeexplore.ieee.org/document/10167687/)
- [DexGraspNet](https://pku-epic.github.io/DexGraspNet/)
- [GraspIt! 文件](https://graspit-simulator.github.io/build/html/grasp_planning.html)

---

### E.4 ros2_control 進階用法

入門篇用了 `sensor_msgs/JointState` 來發布關節狀態，但真正的機器人硬體控制需要 **ros2_control** 框架。它是 ROS 2 中連接軟體和硬體的標準層。

#### 架構概覽

```
┌──────────────────────────────────────────────────┐
│                   使用者程式                       │
│            (MoveIt 2 / 你的 Python 節點)           │
└──────────────────┬───────────────────────────────┘
                   │ action/topic
┌──────────────────▼───────────────────────────────┐
│              Controller Manager                   │
│    (管理所有控制器的生命週期)                        │
├───────────────────────────────────────────────────┤
│  joint_trajectory_controller  │  admittance_ctrl  │
│  position_controller          │  velocity_ctrl    │
└──────────────────┬───────────────────────────────┘
                   │ command/state interfaces
┌──────────────────▼───────────────────────────────┐
│            Hardware Interface（硬體介面）           │
│    你自己寫的 C++ 類別，負責跟實體硬體溝通           │
│    read() ← 讀感測器    write() → 送指令到馬達      │
└──────────────────┬───────────────────────────────┘
                   │ 串口/CAN/EtherCAT/I2C...
┌──────────────────▼───────────────────────────────┐
│              實體硬體（馬達、感測器）                 │
└──────────────────────────────────────────────────┘
```

#### 自訂硬體介面（C++ 範例）

當你用的馬達沒有現成 ROS 2 驅動時，就需要自己寫硬體介面：

```cpp
// my_robot_hardware.hpp
#include "hardware_interface/system_interface.hpp"
#include "rclcpp_lifecycle/state.hpp"

namespace my_robot {

class MyRobotHardware : public hardware_interface::SystemInterface {
public:
  // ===== 生命週期管理 =====

  // 初始化：解析 URDF 中的參數（如串口路徑、波特率）
  CallbackReturn on_init(
    const hardware_interface::HardwareInfo & info) override;

  // 啟動：打開串口連線、使能馬達
  CallbackReturn on_activate(
    const rclcpp_lifecycle::State &) override;

  // 停止：關閉馬達、斷開連線
  CallbackReturn on_deactivate(
    const rclcpp_lifecycle::State &) override;

  // ===== 介面匯出 =====

  // 告訴 Controller Manager 這個硬體能提供哪些狀態
  // （例如：position, velocity, effort）
  std::vector<hardware_interface::StateInterface>
    export_state_interfaces() override;

  // 告訴 Controller Manager 這個硬體能接受哪些指令
  std::vector<hardware_interface::CommandInterface>
    export_command_interfaces() override;

  // ===== 即時控制迴圈（每個週期都被呼叫）=====

  // 從硬體讀取當前狀態（編碼器位置、電流等）
  return_type read(
    const rclcpp::Time & time,
    const rclcpp::Duration & period) override;

  // 將指令寫入硬體（目標位置、速度等）
  return_type write(
    const rclcpp::Time & time,
    const rclcpp::Duration & period) override;

private:
  // 每個關節的狀態和指令緩衝區
  std::vector<double> hw_positions_;
  std::vector<double> hw_velocities_;
  std::vector<double> hw_commands_;
  // 硬體通訊物件（串口、CAN 等）
  // SerialPort serial_;
};

}  // namespace my_robot
```

#### EtherCAT 整合

EtherCAT 是工業機器人最常用的即時通訊協議。**ethercat_driver_ros2**（ICube-Robotics 開發）是 ROS 2 的主要解決方案：

- 每個 EtherCAT Master 被定義為一個 ros2_control 硬體介面
- 支援**熱插拔（Hot Connect）**：動態新增/移除裝置不需重啟
- 支援**分散式時鐘（Distributed Clocks）**：多裝置高精度同步
- 設定透過參數檔，模組化組裝
- **acontis EC-Master** 整合預計 2026 Q1 正式支援

```bash
# 安裝 ethercat_driver_ros2
cd ~/ros2_ws/src
git clone https://github.com/ICube-Robotics/ethercat_driver_ros2.git
cd ~/ros2_ws
colcon build --packages-select ethercat_driver_ros2
```

#### ros2_control 進階參考資源

- [自訂硬體元件教學（Rolling, 2026/03）](https://control.ros.org/rolling/doc/ros2_control/hardware_interface/doc/writing_new_hardware_component.html)
- [ethercat_driver_ros2（GitHub）](https://github.com/ICube-Robotics/ethercat_driver_ros2)
- [EtherCAT Driver 文件](https://icube-robotics.github.io/ethercat_driver_ros2/)
- [acontis EtherCAT for ROS](https://www.acontis.com/en/ethercat-for-ros.html)
- [ADLINK Software EtherCAT + ROS 2](https://www.adlinktech.com/en/software-ethercat-ros2-amr-development)

---

### E.5 強化學習應用於操控

不用手動定義規則或示範，讓機器人透過**反覆試錯**自己學會怎麼操控物體——這就是強化學習（RL）在機械手臂上的應用。

#### 核心演算法比較（2025-2026 基準測試）

| 演算法 | 全名 | 抓取成功率 | 抓取後成功率 | 特點 |
|--------|------|-----------|-------------|------|
| **SAC** | Soft Actor-Critic | **87%** | **75%** | 最大化熵，探索能力強 |
| **PPO** | Proximal Policy Optimization | 82% | 68% | 穩定但較保守 |
| **M2ACD** | Multi-Actor-Critic DDPG | — | — | 軌跡更平滑，收斂更好 |

#### 從 RL 到基礎模型的典範轉移

2025-2026 年，機器人操控正從「針對單一任務訓練 RL 策略」轉向「用大規模預訓練基礎模型」：

| 模型 | 類型 | 資料量 | 特色 |
|------|------|--------|------|
| **OpenVLA**（2025/01） | Vision-Language-Action | 527K 軌跡 | 第一個開源 VLA，Apache 2.0 |
| **Octo** | Diffusion Policy | 800K episodes | Open X-Embodiment 預訓練，500-2000 次示範即可微調 |
| **π0** | Flow Matching | 7 平台 × 68 任務 | 最強通用機器人策略 |
| **ABot-M0**（2026/02） | VLA + Action Manifold | — | 動作流形學習 |
| **Cosmos Policy**（NVIDIA, 2026） | Video Foundation Model | — | 微調 Cosmos 影片模型做動作預測 |

> **對新手的意義**：你不再需要從零訓練一個 RL 模型來做抓取。下載預訓練好的 OpenVLA 或 Octo，用自己的 500-2000 次示範微調，就能得到一個堪用的操控策略。

#### 接觸密集型 RL（2026 新趨勢）

2026 年 SAGE 期刊的綜述指出，RL 特別適合**接觸密集型任務**（assembly、polishing、insertion）：

- **關鍵挑戰**：不同任務間的知識遷移、不同機器人間的策略遷移、持續真實世界 RL
- **觸覺回饋框架「Grasp Mechanics」**：用觸覺取代視覺，降低感測系統複雜度
- **安全感知 RL**：在人機協作環境中，RL 策略內建安全約束

#### 強化學習操控參考資源

- [接觸密集型 RL 操控（2026, SAGE）](https://journals.sagepub.com/doi/10.1177/09596518251350353)
- [深度 RL 自適應抓取（2025）](https://www.mdpi.com/1999-5903/17/10/437)
- [雙臂靈巧操控的漸進式策略學習](https://www.mdpi.com/2227-7390/13/22/3585)
- [機器人操控基礎模型綜述（2025）](https://journals.sagepub.com/doi/10.1177/02783649251390579)
- [開源機器人基礎模型指南](https://robocloud-dashboard.vercel.app/learn/blog/open-weight-robot-models)
- [Octo 模型](https://octo-models.github.io/)

---

### E.6 Sim-to-Real 模擬到現實

在模擬器裡訓練好的 AI 策略，放到真實機器人上卻不能用？這就是**現實差距（Reality Gap）**。Sim-to-Real 技術就是要解決這個問題。

#### 現實差距的來源

| 差距類型 | 範例 | 解法 |
|---------|------|------|
| **物理差距** | 模擬的摩擦力、質量跟真實不同 | 物理參數隨機化 |
| **視覺差距** | 模擬的光影、材質跟真實不同 | 視覺域隨機化 |
| **感測器差距** | 模擬的相機雜訊跟真實不同 | 加入感測器雜訊模型 |
| **致動器差距** | 模擬的馬達響應跟真實不同 | 系統識別 + 參數校準 |

#### 核心技術

**1. 域隨機化（Domain Randomization）**

> 比喻：如果你在各種光線、各種地形、各種天氣下都練習過開車，那到了新環境也不怕。

在訓練時隨機變化模擬環境的參數：
- 視覺：材質、光照、顏色、背景
- 物理：質量、摩擦係數、阻尼
- 感測器：雜訊、延遲、偏差

**2. 系統識別（System Identification）**

測量真實硬體的物理參數（關節摩擦、連桿質量等），然後在模擬器中精確匹配。越準確，差距越小。

**3. 自動域隨機化（ADR, Automatic Domain Randomization）**

NVIDIA Isaac Lab 2.3 引入——訓練過程中根據策略表現自動調整隨機化範圍：策略表現好就加大隨機化（增加難度），表現差就縮小範圍。

#### NVIDIA Isaac Lab：2026 年主流平台

Isaac Lab 是 Isaac Gym 的繼任者，目前是 Sim-to-Real 的領導平台：

```python
# Isaac Lab 域隨機化設定範例
from isaaclab.envs import ManagerBasedRLEnvCfg
from isaaclab.managers import EventTermCfg

@configclass
class EventCfg:
    # 隨機化關節摩擦（每次 episode 重置時）
    randomize_joint_friction = EventTermCfg(
        func=mdp.randomize_joint_parameters,
        params={
            "friction_range": (0.5, 2.0),  # 0.5x ~ 2x 原值
            "asset_cfg": SceneEntityCfg("robot")
        },
        mode="reset",
    )
    # 隨機化物體質量
    randomize_mass = EventTermCfg(
        func=mdp.randomize_rigid_body_mass,
        params={
            "mass_range": (-0.2, 0.5),  # 原值 -0.2 ~ +0.5 kg
            "asset_cfg": SceneEntityCfg("object")
        },
        mode="reset",
    )
```

#### 2025 年新技術：隱空間擴散模型

用**潛在擴散模型（Latent Diffusion Models）**將模擬影像轉換為逼真影像：
- 在潛在空間操作，效率高
- 支援少樣本適應（few-shot）
- 可即時運行

#### 零樣本部署已實現

2025-2026 年多個團隊已展示：在 Isaac Lab 中訓練的策略，**直接**部署到真實機器人上，無需微調。這是 Sim-to-Real 的里程碑。

#### Sim-to-Real 參考資源

- [NVIDIA Isaac Lab: Sim-to-Real 工業組裝](https://developer.nvidia.com/blog/bridging-the-sim-to-real-gap-for-industrial-robotic-assembly-applications-using-nvidia-isaac-lab/)
- [Isaac Lab 視覺域隨機化文件](https://docs.nvidia.com/learning/physical-ai/getting-started-with-isaac-lab/latest/transferring-robot-learning-policies-from-simulation-to-reality/03-bridging-the-gap-simulation-enhancement/01-visual-domain-randomization.html)
- [Isaac Lab 2.3: 全身控制與遙操作](https://developer.nvidia.com/blog/streamline-robot-learning-with-whole-body-control-and-enhanced-teleoperation-in-nvidia-isaac-lab-2-3/)
- [機器人現實差距：挑戰與解法（2025）](https://arxiv.org/html/2510.20808v1)
- [持續域隨機化（2025）](https://arxiv.org/html/2403.12193v2)

---

### E.7 雙臂與多臂協調

一隻手臂能做的事有限——想像用一隻手打開瓶蓋，或用一隻手折衣服，幾乎不可能。雙臂（或多臂）協調是讓機器人做更複雜任務的關鍵。

#### 核心挑戰

1. **運動協調**：兩隻手臂不能撞到彼此
2. **力協調**：抬重物時兩隻手臂要分攤力量
3. **任務分解**：哪隻手臂做什麼？何時要同步、何時要獨立？

#### 2025-2026 年主要框架

| 框架 | 特色 | 適用場景 |
|------|------|---------|
| **DualTHOR**（2025） | 雙臂人形模擬平台，含真實機器人資源和逆運動學 | 雙臂協作任務的 RL 訓練 |
| **RoboTwin** | 用 3D 生成模型 + LLM 自動產生雙臂任務的專家資料 | 資料集生成與評估 |
| **DAC-MPiH** | 雙臂協作多柱嵌入孔組裝，八階段組裝策略 | 工業組裝 |
| **SSMC-TDE**（2026） | 同步滑模控制 + 時延估計，精確關節協調 | 高精度雙臂同步 |
| **DA-VIL** | 結合 RL 和可變阻抗控制的自適應雙臂操控 | 安全的人機雙臂互動 |

#### 雙臂協調的常見策略

1. **主從式（Leader-Follower）**：一隻手臂是主導，另一隻跟隨——簡單但不靈活
2. **對等式（Peer-to-Peer）**：兩隻手臂各自規劃，透過約束保持協調——靈活但複雜
3. **任務優先級式（Task Priority）**：定義任務優先級堆疊，高優先級任務先滿足——工業常用

#### 雙臂協調參考資源

- [DualTHOR: 雙臂人形模擬（2025）](https://arxiv.org/html/2506.16012v1)
- [DA-VIL: 自適應雙臂 RL + 可變阻抗](https://arxiv.org/html/2410.19712v1)
- [雙臂協作柱嵌入孔組裝（2025）](https://www.sciencedirect.com/science/article/abs/pii/S0736584525000456)
- [SSMC-TDE 雙臂同步控制（2026）](https://www.mdpi.com/2076-3417/16/4/2042)
- [RoboTwin: 雙臂數位孿生基準](https://www.researchgate.net/publication/392068793)
- [多機器人協作操控（Frontiers, 2025-2026）](https://www.frontiersin.org/journals/robotics-and-ai/articles/10.3389/frobt.2025.1585544/full)

---

### E.8 移動式操控（Mobile Manipulation）

把機械手臂裝在移動底盤上——讓機器人既能走又能拿。這是 2026 年工業和服務機器人的主流形態。

#### 市場規模

協作移動操控機器人市場在 2025 年達到 **18 億美元**，2026 年預估 **23 億美元**（年增長率 22.8%）。核心價值：消除固定傳輸區，機器人自主導航到庫存架、抓取元件、送到組裝線。

#### 技術架構

```
┌─────────────────────────────────────┐
│           任務規劃層                  │
│  (LLM / 行為樹 / 狀態機)             │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│        全域路徑規劃（去哪）             │
│  Nav2 / A* / Dijkstra               │
├──────────────┬──────────────────────┤
│        局部避障（怎麼走）              │  ← 同時運行
│  DWB / MPPI / TEB                   │
├──────────────┬──────────────────────┤
│        手臂操控（怎麼拿）              │
│  MoveIt 2 / VLA 模型                │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│        硬體控制                       │
│  ros2_control (底盤 + 手臂)          │
└─────────────────────────────────────┘
```

#### 關鍵技術（2025-2026）

**1. 分層運動規劃 + CLF-CBF 安全保障**
- 融合全域和局部規劃器
- **CLF（Control Lyapunov Function）**：確保追蹤全域路徑
- **CBF（Control Barrier Function）**：保證不碰撞
- 兩者結合，同時滿足「到得了」和「不會撞」

**2. 混合 DRL + 圖搜索規劃**
- 遠距離目標用圖搜索（利用地圖資訊）
- 近距離動態避障用深度 RL（即時反應）

**3. NaVILA（2025-2026）**
- 專為有腿機器人設計的 Vision-Language-Action 模型
- 代表基礎模型和移動操控的融合

#### 移動式操控參考資源

- [協作移動操控機器人市場報告（2026）](https://www.futuremarketinsights.com/reports/collaborative-mobile-manipulator-robots-market)
- [分層運動規劃 + CLF-CBF（2025）](https://www.mdpi.com/2218-6581/14/7/96)
- [混合 DRL 運動規劃（2025）](https://arxiv.org/html/2512.24651v1)
- [零樣本操控框架（Nature, 2025）](https://www.nature.com/articles/s41598-025-17015-z)

---

### E.9 數位孿生（Digital Twin）

數位孿生不只是「模擬」——它是真實機器人的**即時數位鏡像**。模擬器跑一次就結束，但數位孿生持續同步：真實機器人動了，數位版也跟著動；在數位版上測試新策略，驗證後再部署到真實機器人。

#### NVIDIA 平台（2026 年市場領導者）

| 工具 | 用途 |
|------|------|
| **Isaac Sim** | 物理模擬，設計/測試/訓練 AI 機器人 |
| **Isaac Lab** | GPU 加速的 RL/模仿學習框架（Isaac Gym 繼任者） |
| **Omniverse** | 底層平台，被定位為「Physical AI 作業系統」 |
| **Mega**（2026 新） | 工廠規模數位孿生，測試多機器人機隊 |
| **Cosmos 3** | 物理世界 AI 基礎模型 |
| **GR00T N1.7** | 人形機器人基礎模型（雙系統：慢思考 + 快行動） |

**GTC 2026 重大公告**：
- **Mega Blueprint**：在工廠級數位孿生中測試多機器人機隊，已進入預覽
- **Physical AI Data Factory**：推進世界建模、人形技能、自動駕駛
- 多家企業已在使用：
  - **Schaeffler + Accenture**：用 Mega 模擬 Agility Digit 機器人做物料搬運
  - **Hyundai**：模擬 Boston Dynamics Atlas 在組裝線上的作業
  - **Mercedes-Benz**：模擬 Apptronik Apollo 人形機器人進行車輛組裝
  - **Infineon + NVIDIA**（2026/03）：擴大人形機器人數位孿生合作

#### 開源替代方案

| 工具 | 特色 | 適合場景 |
|------|------|---------|
| **Gazebo** | ROS 2 生態系統標配，免費 | 教育、中小型專案 |
| **MuJoCo** | Google DeepMind 開源，物理精度高 | RL 研究 |
| **RoboTwin** | 用 3D 生成模型 + LLM 自動建立雙臂任務環境 | 資料集生成 |

#### 數位孿生參考資源

- [NVIDIA 機器人平台](https://www.nvidia.com/en-us/industries/robotics/)
- [GTC 2026: 虛擬世界驅動物理 AI](https://blogs.nvidia.com/blog/gtc-2026-virtual-worlds-physical-ai/)
- [Isaac Sim](https://developer.nvidia.com/isaac/sim)
- [Isaac GR00T（GTC 2026）](https://blockchain.news/news/nvidia-isaac-gr00t-robotics-platform-gtc-2026)
- [Infineon + NVIDIA 數位孿生合作（2026/03）](https://roboticsandautomationnews.com/2026/03/17/infineon-and-nvidia-expand-collaboration-to-accelerate-humanoid-robots-using-digital-twins/99784/)
- [數位孿生學習路徑](https://www.nvidia.com/en-us/learn/learning-path/digital-twins/)

---

### E.10 安全標準與協作機器人規範

機械手臂可以傷人——尤其是工業等級的。了解安全標準不只是法規要求，更是保護自己和他人的基本功。

#### 重大更新：ISO 10218:2025（2025 年發布）

ISO 10218 是工業機器人安全的核心標準，2025 年發布了自 2011 年以來**最大幅度的修訂**，由 20+ 國專家歷時近 8 年完成。

##### 關鍵變化

| 變化 | 舊版（2011） | 新版（2025） |
|------|------------|------------|
| **協作機器人定義** | 有「協作機器人」概念 | 改為「**協作應用（Collaborative Application）**」——只有使用方式可以是協作的，不是機器人本身 |
| **ISO/TS 15066** | 獨立標準 | **整合進 ISO 10218-2:2025**，不再需要另外參照 |
| **資安要求** | 無 | **新增網路安全要求** |
| **範圍** | 「機器人系統」 | 改為「**機器人應用**」，更貼近實際使用 |

##### 四種協作模式（沿用但強化）

1. **安全等級監控停止（Safety-Rated Monitored Stop）**
   - 人靠近時暫停，離開後恢復
   - 最簡單的協作方式

2. **手動引導（Hand Guiding）**
   - 人直接用手帶著機器人移動
   - 適合教學和直覺式程式設計

3. **速度與分離監控（Speed and Separation Monitoring）**
   - 根據人的距離動態調整速度
   - 人越近速度越慢，太近則停止

4. **功率與力量限制（Power and Force Limiting）**
   - 限制機器人能施加的最大力量和功率
   - 即使碰撞也不會造成傷害
   - 常見於「協作機器人（cobot）」如 UR 系列

##### 對開發者的實際意義

- 任何新的協作機器人部署都應參照 **ISO 10218:2025**
- 風險評估現在必須包含**網路安全威脅**
- 標準整合後，合規變得更簡單——不需要同時查閱 ISO 10218 和 ISO/TS 15066

#### 安全標準參考資源

- [ISO 10218-1:2025（ISO 官網）](https://www.iso.org/standard/73933.html)
- [ISO 10218 重大改版（The Robot Report）](https://www.therobotreport.com/iso-10218-industrial-robot-safety-standard-receives-major-overhaul/)
- [ISO 10218-1:2025 概覽（ANSI Blog）](https://blog.ansi.org/ansi/iso-10218-1-2025-robots-and-robotic-devices-safety/)
- [合規影響分析（Intertek）](https://assuranceinaction.intertek.com/post/102ksxe/iso-10218-update-what-it-means-for-robotic-safety-and-compliance)
- [ISO 10218 FAQ（A3/Automate）](https://www.automate.org/robotics/blogs/updated-iso-10218-faq)
- [協作機器人安全標準概覽](https://standardbots.com/blog/collaborative-robot-safety-standards)

---

### 進階篇學習路線建議

根據你的目標選擇路線：

#### 路線 E：學術研究導向

| 順序 | 主題 | 時間 | 產出 |
|------|------|------|------|
| 1 | E.1 運動規劃 + E.3 抓取規劃 | 2-3 週 | 了解 OMPL/STOMP，能設定 MoveIt 2 規劃器 |
| 2 | E.5 強化學習 + E.6 Sim-to-Real | 3-4 週 | 在 Isaac Lab 中訓練抓取策略並部署 |
| 3 | E.9 數位孿生 | 1-2 週 | 建立自己的模擬-現實管線 |
| 4 | 附錄 D 的 VLA 模型 | 2-3 週 | 微調 SmolVLA/OpenVLA 在自己的機械手臂上 |

#### 路線 F：工業應用導向

| 順序 | 主題 | 時間 | 產出 |
|------|------|------|------|
| 1 | E.4 ros2_control + E.2 力/力矩控制 | 2-3 週 | 寫出自訂硬體介面，實現導納控制 |
| 2 | E.10 安全標準 | 1 週 | 理解 ISO 10218:2025 的四種協作模式 |
| 3 | E.1 運動規劃 + E.3 抓取規劃 | 2-3 週 | 設定 STOMP + AnyGrasp 管線 |
| 4 | E.8 移動式操控 | 2-3 週 | 整合 Nav2 + MoveIt 2 |

#### 路線 G：AI 機器人研究（2026 最前沿）

| 順序 | 主題 | 時間 | 產出 |
|------|------|------|------|
| 1 | 附錄 D 的 VLA 模型 + E.5 RL | 3-4 週 | 理解基礎模型和 RL 的關係 |
| 2 | E.6 Sim-to-Real | 2-3 週 | 掌握 Isaac Lab 域隨機化 |
| 3 | E.7 雙臂協調 | 2 週 | 用 DualTHOR 或 RoboTwin 做雙臂任務 |
| 4 | E.3 抓取規劃 | 2 週 | 用 AnyDexGrasp 實現通用抓取 |

---

### 進階篇資料來源總整理

#### 運動規劃
- [OMPL](https://ompl.kavrakilab.org/)
- [MoveIt 2 規劃器文件](https://moveit.ai/documentation/planners/)
- [RRT\*-Connect（2025）](https://arxiv.org/html/2505.10542v1)
- [DRL vs 取樣式規劃器（2025）](https://pmc.ncbi.nlm.nih.gov/articles/PMC12431259/)

#### 力控制
- [ROS 2 Admittance Controller](https://control.ros.org/iron/doc/ros2_controllers/admittance_controller/doc/userdoc.html)
- [質量自適應導納控制（2025）](https://arxiv.org/html/2504.16224v1)

#### 抓取
- [AnyDexGrasp](https://graspnet.net/anydexgrasp/)
- [靈巧抓取綜述（2025）](https://link.springer.com/article/10.1007/s10462-025-11262-2)

#### ros2_control
- [自訂硬體元件教學](https://control.ros.org/rolling/doc/ros2_control/hardware_interface/doc/writing_new_hardware_component.html)
- [EtherCAT Driver ROS2](https://github.com/ICube-Robotics/ethercat_driver_ros2)

#### 強化學習
- [接觸密集型 RL（2026）](https://journals.sagepub.com/doi/10.1177/09596518251350353)
- [基礎模型綜述（2025）](https://journals.sagepub.com/doi/10.1177/02783649251390579)
- [Octo 模型](https://octo-models.github.io/)

#### Sim-to-Real
- [NVIDIA Isaac Lab](https://developer.nvidia.com/isaac/sim)
- [現實差距（2025）](https://arxiv.org/html/2510.20808v1)

#### 雙臂
- [DualTHOR（2025）](https://arxiv.org/html/2506.16012v1)
- [SSMC-TDE（2026）](https://www.mdpi.com/2076-3417/16/4/2042)

#### 移動操控
- [市場報告](https://www.futuremarketinsights.com/reports/collaborative-mobile-manipulator-robots-market)
- [CLF-CBF 分層規劃（2025）](https://www.mdpi.com/2218-6581/14/7/96)

#### 數位孿生
- [NVIDIA Omniverse](https://www.nvidia.com/en-us/industries/robotics/)
- [GTC 2026 公告](https://blogs.nvidia.com/blog/gtc-2026-virtual-worlds-physical-ai/)

#### 安全標準
- [ISO 10218:2025](https://www.iso.org/standard/73933.html)
- [ISO 10218 FAQ（Automate）](https://www.automate.org/robotics/blogs/updated-iso-10218-faq)
- [Gazebo Kura 發布日期](https://discourse.openrobotics.org/t/new-release-date-for-gazebo-kura/51429)
