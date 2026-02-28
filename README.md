---

# Livox Mid-360 + FAST_LIO ROS 2 Humble 配置指南

本指南旨在帮助开发者在 **Ubuntu 22.04 + ROS 2 Humble** 环境下，快速完成 Mid-360 雷达驱动安装及 FAST_LIO 算法部署。

---

## 1. 安装底层通信库：Livox-SDK2

`Livox-SDK2` 是驱动雷达的核心底层库，必须首先安装。

```bash
# 安装必要编译工具
sudo apt update

# 克隆并编译 Livox-SDK2
git clone https://github.com/Livox-SDK/Livox-SDK2.git
cd Livox-SDK2/
mkdir build && cd build
cmake .. && make -j$(nproc)
sudo make install
```

---

## 2. 网络环境配置

1. **物理连接**：将雷达上电并接入电脑网口。
2. **设置静态 IP**：#就是连雷达的那个口的IP设置设置 windows和ubuntu设置办法我挂两个链接
   - **IP 地址**：`192.168.1.50` 
   - **子网掩码**：`255.255.255.0`
   - **网关**：`192.168.1.1`

---

## 3. 安装 Livox ROS Driver 2

### 3.1 源码克隆
建议创建独立的工作空间（如 `ws_livox`）：
```bash
mkdir -p ~/ws_livox/src
cd ~/ws_livox/src
git clone https://github.com/tangan-harsh/Fast_lio_Tangan.git
```

### 3.2 修改 IP 配置文件
打开 `src/livox_ros_driver2/config/MID360_config.json`，根据实际情况修改：
- `host_ip`: 改为你的电脑静态 IP `192.168.1.50`。
- `ip`: 改为雷达 IP `192.168.1.1XX`（`XX` 为雷达 S/N 码最后两位）。

### 3.3 编译与测试
官方提供了编译脚本，会自动处理复杂的依赖路径：
```bash
cd ~/ws_livox/src/livox_ros_driver2
chmod +x build.sh
./build.sh humble  # 脚本会自动在工作空间下生成 build/install 目录

# 运行测试
cd ~/ws_livox
source install/setup.bash
ros2 launch livox_ros_driver2 rviz_MID360_launch.py
```
一般会到这个你可以看到点云了
如果没有点云：
我遇到的问题是这个雷达的ip是被改了的所以雷达ip不对
解决方法:下一个LivoxViewer2 在关闭wifi的情况下在192.168.1.50这个ip下查看那雷达ip
然后改回`192.168.1.1XX`（`XX` 为雷达 S/N 码最后两位）的格式就可以了

---

## 4. 运行与调试

### 4.1 启动流程
1. **终端 1：启动驱动**
   ```bash
   cd ~/ws_livox
   source install/setup.bash
   # 若跑算法，建议使用 msg 模式发布自定义点云格式
   ros2 launch livox_ros_driver2 msg_MID360_launch.py
   ```

2. **终端 2：启动 FAST_LIO**
   ```bash
   cd ~/ws_livox
   source install/setup.bash
   ros2 launch fast_lio mapping.launch.py
   ```

### 4.2 常见问题：Rviz 黑屏或地图极小
- **视角问题**：FAST_LIO 初始地图可能很小，请尝试在 Rviz 中滚动鼠标滚轮放大，或将 `Fixed Frame` 选为 `camera_init`。
- **坐标系**：确保 Rviz 的左侧面板已添加 `PointCloud2` 话题，且 `Topic` 路径为 `/CloudRegistered`（FAST_LIO 输出的配准后点云）。
