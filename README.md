# Autonomous Vehicle Plannin and Control

深蓝学院的规控课程，简单记点笔记。

- Project 1 PID 纵向控制
- Project 2 Stanley 横向控制
- Project 3 LQR 横向控制
- Project 4 MPC 横向控制
- Project 5 Frenet 坐标系下的局部路径规划（Lattice Planner）
- Project 6 Lanelet 2 高精度地图

## Carla-ROS 联合仿真环境配置

### 1.1 安装 Carla

从 Github 下载 CARLA 0.9.13 并解压即可：https://github.com/carla-simulator/carla/releases/tag/0.9.13/。

如果显卡驱动没问题的话，就可以运行 Carla 了

如果是 Ubuntu 用户，直接执行以下命令：

```bash
# 尖括号需要修改为⾃⼰电脑上的路径，例如 cd /home/qrq/Carla 
cd <path-to-carla>

# ⾸次运⾏ Carla 需要先导入配置文件
./ImportAssets.sh

# 若要以低分辨率运⾏，可以使⽤ ./CarlaUE4.sh -quality-level=Low
./CarlaUE4.sh
```

此时没准会报错，需要安装一下 `libomp5`。安装了还解决不了的话，再确定下是不是显卡驱动的问题，如果都没问题，那我也无能为力了。

```bash
sudo apt install libomp5
```

如果是 Windows 用户，直接双击 Carla 的图标即可运行。

### 1.2 Python 环境配置 

由于 Ubuntu 18 对应的 ROS-Melodic 使用的是 Python 2，所以后续的工作主要是针对 Python 2 的环境进行一些配置。

```bash
# 安装 python 2 的 pip
sudo apt install python-pip

# 确定 pip 版本
pip -V

# 升级 pip
pip install --upgrade pip

# 为 Python 2 安装 pygame、numpy、openCV
pip install --user pygame numpy 
pip install --user opencv-python
```

Ubuntu 20 用户则是对 Python 3 的环境进行设置：

```bash
# 安装 pip
sudo apt install python3-pip

# 确定 pip 的版本
pip3 -V

# \升级 pip
pip3 install --upgrade pip

pip3 install --user pygame numpy 
pip3 install opencv-python
```

如果在 `pip install` 的过程中报错 `WARNING: XXX is not on PATH`，则需要在 `~/.bashrc` 的最后一行添加 `export PATH=/home/<username>/.local/bin/:$PATH`

```bash
gedit ~/.bashrc

# <username> 需要更换为自己的用户名
export PATH=/home/<username>/.local/bin/:$PATH
```

保存并关闭 `gedit` 页面后，需要重新刷新环境变量使其生效：

```bash
source ~/.bashrc
```

此外，为了正常使用 Carla-ros-bridge，需要把 Carla 附带的 `egg` 文件的位置高速 Python。这个文件一般位于 `<path-to-carla>/PythonAPI/carla/dist`。

Ubuntu 18 用户，需要添加 Python 2 对应的 egg 文件，Ubuntu 20 用户则需要添加 Python 3 对应的 egg 文件。

首先打开 。bashrc：

```bash
gedit ~/.bashrc
```

再将以下内容填写到 `.bashrc` 的末尾：

```bash
# 尖括号需要修改为⾃⼰电脑上的路径 
export CARLA_ROOT=<path-to-carla>
export PYTHONPATH=$PYTHONPATH:$CARLA_ROOT/PythonAPI/carla/dist/carla- 
<carla_version_and_arch>.egg:$CARLA_ROOT/PythonAPI/carla
```

如果是使用虚拟机的用户，需要再下载个相同版本的 Linux 版 Carla，并将其拷贝到虚拟机中，此时 `.bashrc` 中的 `<path-to-carla>` 就修改为 Linux 端 Carla 的根目录即可。

### 1.3 安装 carla-ros-bridge

安装 ROS 可以借助鱼香 ROS 的牛逼脚本一键安装，ROS 1 和 ROS 2 都是可以的：
https://fishros.org.cn/forum/topic/20/%E5%B0%8F%E9%B1%BC%E7%9A%84%E4%B8%80%E9%94%AE%E5%AE%89%E8%A3%85%E7%B3%BB%E5%88%97?lang=zh-CN

安装完 ROS 后，直接找个地方创建工作空间，将 Github 上的 carla-ros-bridge 整个下载下来进行编译即可。

```bash
mkdir -p ~/carla-ros-bridge && cd ~/carla-ros-bridge
git clone --recurse-submodules https://github.com/carla-simulator/ros-bridge.git src/ros-bridge
```

功能包涉及到的依赖需要用 `rosdep` 进行安装，但可能因为网络的原因报错，这时候就需要再次请来鱼香 ROS 的牛逼脚本，用 `rosdepc` 来处理一下，随后就可正常编译了。

```bash
wget http://fishros.com/install -O fishros && . fishros 
rosdepc update
rosdepc install --from-paths src --ignore-src -r 
```

如果是 ROS 1 用户，请使用 `catkin build` 进行编译；
如果是 ROS 2 用户，请使用 `colcon build` 进行编译；

编译失败多半是依赖的问题，记得用 `rosdepc install --from-paths src --ignore-src -r ` 进行安装，或者直接根据报错信息自行安装。

如果是使用虚拟机的用户，此时还需要多进行一步。并将 `src/carla_ros_bridge/launch` 中 `.launch` 文件和 `.launch.py` 中的 `host` 参数改为主机的 IP 地址。如果是 VMWare 用户，还需要将网络状态改为桥接。修改完后重新编译一下以使设置生效。

此时，一切准备就绪，可以尝试联合仿真了。

Linux 用户需要开启一个终端运行 Carla，Windows 用户直接双击运行即可：

```bash
cd <path-to-carla>
./CarkaUE4.sh
```

Linux 用户需要再打开一个新终端，Windows 用户则需要在虚拟机中打开终端：

```bash
cd <path-to-ros_bridge>

# ROS 2 用户执行以下操作
source ./install/setup.bash
ros2 launch carla_ros_bridge carla_ros_bridge_with_example_ego_vehicle.launch.py

# ROS 1 用户执行以下操作
source ./devel/setup.bash
roslaunch carla_ros_bridge carla_ros_bridge_with_example_ego_vehicle.launch
```

如果一切顺利的话，此时会蹦出一个开车的 pygame 小窗口，按 `B` 可以切换到手动驾驶，`WASD` 操作车辆，`Q` 切换前进/倒车挡。

如果遇到如下报错：

```
Error: time-out of 10000ms while waiting for the simulator, 
make sure the simulator is ready and connected to localhost:2000
```

不必惊慌，只需要在上文提到的 `.launch` 和 `.launch.py` 文件中修改 `timeout` 参数，改成一个比较大的数，比如 10000，并重新编译以及刷新环境变量，此时就可以给 ros-bridge 充足的反应时间去连接 Carla 了，再运行就不会报错了。

如果是其他报错，还请自行 google。
