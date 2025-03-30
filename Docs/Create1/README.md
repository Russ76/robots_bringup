# Turtlebot Create 1 Notes (Jazzy version)

This is an updated and streamlined version of the prior document ( [RPi_Setup](https://github.com/slgrobotics/turtlebot_create/tree/main/RPi_Setup) ) - adjusted for ROS Jazzy.

Familiarity with (now outdated) [Old README](https://github.com/slgrobotics/turtlebot_create/blob/main/README.md) is highly recommended.

My Create 1 has a Raspberry Pi 3B ("turtle"). The RPi runs sensors drivers (XV11 LIDAR and BNO055 IMU), and [Autonomy Lab _base_ code](https://github.com/slgrobotics/create_robot). 

The rest of the robot nodes run on the Desktop, using my *articubot_one* codebase, inspired by Articulated Robotics.

**Note:** You can just run Turtle robot in Gazebo sim on a Desktop machine (no robot hardware required) - 
refer to [this section](https://github.com/slgrobotics/robots_bringup/blob/main/Docs/ROS-Jazzy/README.md#build-articubot_one-robot-codebase)

## Turtle Raspberry Pi 3B Build and Run Instructions:

Turtle has _Ubuntu 24.04 Server (64 bit)_ and _ROS2 Jazzy Base_ installed - see https://github.com/slgrobotics/robots_bringup/tree/main/Docs/Ubuntu-RPi for instructions.

### XV11 LIDAR setup

Refer to https://github.com/slgrobotics/robots_bringup/blob/main/Docs/Sensors/XV11_LIDAR.md

Alternatively, use LD14 LIDAR: https://github.com/slgrobotics/robots_bringup/blob/main/Docs/Sensors/LD14.md

### ROS2 driver for BNO055 9DOF IMU

Refer to https://github.com/slgrobotics/robots_bringup/blob/main/Docs/Sensors/BNO055%20IMU.md

Alternatively, use MPU9250 sensor: https://github.com/slgrobotics/robots_bringup/blob/main/Docs/Sensors/MPU9250.md

### Compile forked Autonomy Lab ROS2 driver for Create 1 base

For iRobot Create 1 (released in 2004, based on Roomba 500 series) and other roombas of 400, 500 and 600 series (https://en.wikipedia.org/wiki/IRobot_Create).

It can be connected via FTDI USB Serial (_/dev/ttyUSB0_ on turtle) or via TTL Serial (pins 1-RXD and 2-TXD on the DB25 connector). A desktop machine can connect via RS232 serial, using special iRobot Create serial cable (/dev/ttyS0 on desktop).

Generally, we follow this guide:

https://github.com/girvenavery2022/create_robot/tree/galactic
    
Review the following. Create 1 requires analog gyro, connected to pin 4 of its Cargo Bay DB25:

https://github.com/AutonomyLab/create_robot/issues/28

https://github.com/slgrobotics/create_robot   (branch _"jazzy"_ is default there, forked from upstream _"iron"_)

https://github.com/slgrobotics/libcreate

https://github.com/slgrobotics/Misc/tree/master/Arduino/Sketchbook/MPU9250GyroTurtlebot

Here are all commands on the Raspberry Pi:
```
# mkdir -p ~/robot_ws/src
cd ~/robot_ws/src
git clone https://github.com/slgrobotics/create_robot.git
git clone https://github.com/slgrobotics/libcreate.git

# you may edit Create Base port here - /dev/ttyS0 if on desktop, /dev/ttyUSB0 on turtle:
vi ~/robot_ws/src/create_robot/create_bringup/config/default.yaml

cd ~/robot_ws
### Note: See https://docs.ros.org/en/humble/Tutorials/Intermediate/Rosdep.html
# sudo rosdep init     -- do it once
rosdep update
# this will take a while, many additional packages installed:
rosdep install --from-paths src --ignore-src -r -y

# On an RPi 3B build will take VERY long time (over 14 hours in my case) and needs at least 2GB swap space.
# For RPi 3B and 4 you must limit number of parallel threads, no need to do it for RPi 5 8GB:
export MAKEFLAGS="-j 1"
colcon build --parallel-workers=1 --executor sequential
(or, just "colcon build" on a Raspberry Pi 4 or 5 - which takes minutes)
```
This is how it looked on my Raspberry Pi 3B with 1 GB RAM:

**Note:** use HDMI monitor and USB keyboard, as SSH will be interrupted on RPi 3 for the lack of resources.
```
ros@turtle:~/robot_ws$ export MAKEFLAGS="-j 1"
ros@turtle:~/robot_ws$ colcon build --parallel-workers=1 --executor sequential
[5.063s] WARNING:colcon.colcon_ros.prefix_path.ament:The path '/home/ros/robot_ws/install/create_robot'
                  in the environment variable AMENT_PREFIX_PATH doesn't exist
...
Starting >>> bno055
Finished <<< bno055 [14.3s]
Starting >>> create_description
Finished <<< create_description [2.29s]
Starting >>> create_msgs
Finished <<< create_msgs [6min 3s]
Starting >>> libcreate
Finished <<< libcreate [7min 48s]
Starting >>> xv_11_driver
Finished <<< xv_11_driver [4min 24s]
Starting >>> create_driver
Finished <<< create_driver [14h 14min 39s]
Starting >>> create_bringup
Finished <<< create_bringup [16.5s]
Starting >>> create_robot
Finished <<< create_robot [10.9s]

Summary: 8 packages finished [14h 33min 43s]
ros@turtle:~/robot_ws$
```
Test it on _turtle_:
```
source ~/robot_ws/install/setup.bash
ros2 launch create_bringup create_1.launch
    or, for Roomba 500/600 series:
ros2 launch create_bringup create_2.launch
```
This is how the robot comes up on my screen:
```
ros@turtle:~/robot_ws$ ros2 launch create_bringup create_1.launch
[INFO] [launch]: All log files can be found below /home/ros/.ros/log/2024-07-23-09-11-31-032285-turtle-26928
[INFO] [launch]: Default logging verbosity is set to INFO
[INFO] [create_driver-1]: process started with pid [26934]
[INFO] [robot_state_publisher-2]: process started with pid [26935]
[robot_state_publisher-2] [INFO] [1721743897.733288931] [robot_state_publisher]: Robot initialized
[create_driver-1] [INFO] [1721743897.817175928] [create_driver]: [CREATE] gyro_offset: 0    gyro_scale: 1
[create_driver-1] [INFO] [1721743897.818827061] [create_driver]: [CREATE] "CREATE_1" selected
[create_driver-1] [INFO] [1721743898.881177255] [create_driver]: [CREATE] Connection established.
[create_driver-1] [INFO] [1721743898.881650532] [create_driver]: [CREATE] Battery level 100.00 %
[create_driver-1] [INFO] [1721743899.054224697] [create_driver]: [CREATE] Ready.
```
### Populate _launch_ folder: _/home/ros/launch_
```
mkdir ~/launch
cd ~/launch
# place myturtle.py and bootup_launch.sh here:
cp ~/robot_ws/src/create_robot/create_bringup/launch/bootup_launch.sh .
cp ~/robot_ws/src/create_robot/create_bringup/launch/myturtle.py .
# You might need a helper program to adjust "gyro_offset":
cp ~/robot_ws/src/create_robot/create_bringup/launch/roomba.py .
chmod +x ~/launch/bootup_launch.sh    
```
Now, when you need to run the on-board nodes on the robot using SSH, just type:
```
cd ~/launch
./bootup_launch.sh
```
## Test-driving Create 1 Turtlebot with _teleop_

Check out https://github.com/slgrobotics/robots_bringup/blob/main/Docs/Sensors/Joystick.md

Keep in mind, that Create 1 base expects */diff_cont/cmd_vel* - while joystick without tweaking produces */cmd_vel_joy*

You may want to temporarily modify *joystick.launch.py* for this test.

## _Optional:_ Create a Linux service for on-boot autostart

With _Create base_, _XV11 Laser Scanner_ and _BNO055 IMU_ ROS2 nodes tested, it is time to set up autostart on boot for hands-free operation.

See https://github.com/slgrobotics/robots_bringup/blob/main/Docs/Ubuntu-RPi/LinuxService.md

## On the Desktop:

Once you have Turtlebot on-board nodes (hardware "drivers") running, it is time to run the rest of the nodes and RViz on the Desktop.

Consult [running-a-physical-robot](https://github.com/slgrobotics/robots_bringup/tree/main/Docs/ROS-Jazzy#running-a-physical-robot)

## Tuning your Gyro (only for Create 1)

If you have Create 1 base - it needs a gyro to compensate for a firmware bug (see https://github.com/AutonomyLab/create_robot/issues/28).

Any analog gyro will do. The original accessory interface board which plugs into the Cargo Bay DB25 connector has an analog gyro, ADXR613. 

I had to create an _"analog gyro emulator"_ by connecting arduino mini to MPU9250 - and producing the same analog signal via PWM (https://github.com/slgrobotics/Misc/tree/master/Arduino/Sketchbook/MPU9250GyroTurtlebot).

*Create base* does not read the gyro (or gyro emulator) directly - it just passes it through from the analog input (DB25 pin 4 of Cargo Bay) to the serial stream (which sends all sensor data every 15ms).

When stationary, the gyro output will produce an ADC value between 0 and 1024, hopefully around 512, and that value will vary with rotation (reflecting, naturally, robot's turn rate).

Autonomy Lab *Create driver* [with my modifications](https://github.com/slgrobotics/create_robot) reads this value, adds gyro_offset and multiplies it by gyro_scale - and then integrates it (by dt) to produce turn angle.

https://github.com/slgrobotics/libcreate/blob/master/src/create.cpp : 148
```
// This is a fix involving analog gyro connected to pin 4 of Cargo Bay:
uint16_t angleRaw = GET_DATA(ID_CARGO_BAY_ANALOG_SIGNAL);
float angleF = -((float)angleRaw - 512.0 + getGyroOffset()) * dt;
angleF = angleF * 0.25 * getGyroScale(); // gyro calibration factor
//std::cout<< "dt: " << dt << " distanceRaw: " << distanceRaw << " angleRaw: " << angleRaw << " angleF: " << angleF << std::endl;
deltaYaw = angleF * (util::PI / 180.0); // D2R
```

Create driver needs *angle* to correctly publish *diff_cont/odom* topic, which is important for robot localization as it moves. Correct wheel joints rotation is the best indication of normal operation of odometry calculations.

You will need to calibrate your gyro, by tweaking parameters (see launch file at https://github.com/slgrobotics/turtlebot_create/tree/main/RPi_Setup/launch ).

**Tuning gyro_offset, gyro_scale and distance_scale**

There are three parameters in *~/launch/myturtle.py* (which you copied above). By adjusting them you make odometry (reported by *Create base driver*) work properly.
```
'gyro_offset': 0.0,
'gyro_scale': 1.19,
'distance_scale': 1.02
```
"Cargo Bay Analog Signal", as read by *Create 1 base* on DB25 pin 4, connected in our case to gyro, is expected to be 512 when the robot is stationary. If it differs (say, 202 when robot doesn't move) - gyro_offset compensates for that (say, 512-202=310). Adjust it accordingly using a helper program: ```cd ~/launch; python3 roomba.py```.

Now you need to bring up Rviz2 to see odometry vector. The best way is to follow "On the Desktop" section above.

The turn rate scale, as reported by gyro, usually needs adjustment. Drive the robot using joystick - turn it 360 degrees and see if the Odometry vector (showing *diff_cont/odom*) is lagging behind or gaining over the robot's orientation. Adjust 'gyro_scale' to have them match.

The *distance_scale* can be adjusted so that *diff_cont/odom* **pose** reports proper distance when robot is driven forward or backward.

As a final test, you need to drive the robot forward a couple meters and watch the odom point in Rviz to stay at the launch point. Then turn the robot and watch the *odom* point move. You should strive for minimal odom displacement during straight runs and rotations.

Once the parameters are adjusted, your robot will be able to map the area, and the _odom_ point will not move dramatically when the robot drives and turns in any direction.

**Tip:** Any time you need to produce a robot URDF from ```.xacro``` files, use "_xacro_" command, for example:
```
source ~/robot_ws/install/setup.bash
xacro ~/robot_ws/install/articubot_one/share/articubot_one/robots/turtle/description/robot.urdf.xacro sim_mode:=true > /tmp/robot.urdf
```
----------------

**Back to** [Docs Folder](https://github.com/slgrobotics/robots_bringup/tree/main/Docs)
