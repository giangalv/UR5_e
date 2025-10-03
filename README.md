# UR5 + ROS 2 Remote Control Setup Tutorial
This guide shows how to set up a UR5 robot to work with ROS 2 using **Remote Control** via the **External Control URCap**.

## PART 1 - UR5 Control Box (Robot) Setup
### Step 1 - Access Network Setting
1. On the **teach pendant, go to:

```css
Menu (≡) → Setup Robot → Network
```
2. Select the Ethernet interface (eth0) you will use.

### Step 2 - Configure a Static IP
* Change IP assignment from DHCP to **Static**.
* Enter the following settings (based on your current network):

```nginx
IP address:        192.168.12.21   (UR5’s IP)
Subnet mask:       255.255.255.0
Default gateway:   192.168.12.1
Preferred DNS:     192.168.12.1
Alternative DNS:   0.0.0.0
```
| ✅ Note: The IP must be unique on the network and in the same subnet as your PC.

**Save** the settings and **reboot** the robot.

### Step 3 - Install External Control URCap
1. Obtain the `.urcap` file (e.g. `externalcontrol-x.x.x.urcap`) from the **Universal Robots ROS 2 driver resources**.
2. Transfer the file to the robot:
- **USB stick**: plug into the teach pendant
3. On the pendant:

```css
Menu (≡) → Setup Robot → URCaps → + Add URCap
```
4. Select the `.urcap` file, install, and **restart the robot**.
5. After reboot, configure the URcap:
- Set **Host IP = 192.168.12.xxx (your PC's IP)
- Save the configuration.

### Step 4 - Create External Control Program
1. Open **PolyScope** -> **Program Robot** -> **New Program**.
2. Insert the External Control program node.
3. Save the program - this will be used to connect the robot to ROS 2.

### Step 5 - Verify Robot IP
* On the pendant: **Menu** -> **Setup Robot** -> **Network**
* Confirm the robot IP matches the planned adress (`192.168.12.21`) and is reachable from the PC.

## PART 2 - Remote PC (Laptop) Setup
### Step 1 - COnfigure Network Interface
1. Disable other connections temporaily (Wi-fi,VPN).
2. Create a **new wired connection**:
* Name: `UR5-e`
* IPv4 -> Manual
3. Set the static IP
```makefile
Address:    192.168.12.xxx  (your PC)
Netmask:    255.255.255.0
Gateway:    192.168.12.1
DNS:        192.168.12.1
```

### Step 2 - Verify Network Connection
* Open terminal and ping the robot:
```bash
ping 192.168.12.21
```
* You should see replies like:
```bash
64 bytes from 192.168.12.21: icmp_seq=1 ttl=64 time=0.037 ms
```
| ✅ If ping works, PC and UR5 are in the same subnet.

### Step 3 - Install ROS 2 UR Driver
1. 
```bash
sudo apt update
sudo apt install -y \
    ros-humble-ros2-control \
    ros-humble-ros2-controllers \
    ros-humble-moveit \
    ros-humble-moveit-servo \
    ros-humble-joint-state-publisher-gui \
    python3-colcon-common-extensions \
    build-essential

cd ~/ur_ws/src

# UR ROS2 Driver
git clone -b humble https://github.com/UniversalRobots/Universal_Robots_ROS2_Driver.git

# Description and common packages
git clone -b humble https://github.com/UniversalRobots/Universal_Robots_Description.git
git clone -b humble https://github.com/UniversalRobots/Universal_Robots_ROS2_Description.git

# MoveIt configuration (optional if you want planning)
git clone -b humble https://github.com/ros-planning/moveit_resources.git

cd ..

rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install
```

## PART 3 - Run Calibration Script
The ROS 2 driver provides a helper launch file to extract factory calibration.
```bash
ros2 launch ur_calibration calibration_correction.launch.py \
robot_ip:=192.168.12.21 target_filename:="${HOME}/my_robot_calibration.yaml"
```

The script connects to the robot and downloads its **factory calibration data**.
It will save it in a **YAML file**, which can later be loaded by the UR ROS driver.

Typical output:
```
[calibration_correction-1] [INFO] [1759493793.055217028] [ur_calibration]: Writing calibration data to "${HOME}/my_robot_calibration.yaml"
[calibration_correction-1] [INFO] [1759493793.055813248] [ur_calibration]: Wrote output.
[calibration_correction-1] [INFO] [1759493793.055883057] [ur_calibration]: Calibration correction done
```

```bash
ros2 launch ur_robot_driver ur5.launch.py robot_ip:=192.168.12.21 calibration_file:="${HOME}/my_robot_calibration.yaml"
```
