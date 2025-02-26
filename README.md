## Set up

First [install ROS 2 Humble on Ubuntu 22.04](http://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debians.html). Then follow the instruction below.

```shell
$ source /opt/ros/humble/setup.bash
$ mkdir -p ~/ros2_ws/ && cd ~/ros2_ws/
$ git clone https://github.com/gabrieldse/manipulator_h.git src && cd src
$ git submodule update --init --recursive
$ vcs import . < dynamixel_hardware/dynamixel_control.repos && cd ..
$ rosdep install --from-paths src --ignore-src -r -y
$ colcon build --symlink-install --cmake-args -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
$ source install/setup.bash
```

## Demo with real ROBOTIS Manipulator H

### To check if everything was done correctly and check the urdf file

```shell
ros2 launch manipulator_h_description manipulator_h_rviz_gui.launch.py
```

or to see it with a gripper:

```shell
ros2 launch manipulator_h_description manipulator_h_rviz_gui_with_gripper.launch.py
```

### Configure Dynamixel motor parameters

Update the `usb_port`, `baud_rate`, and `joint_ids` parameters on `manipulator_h/control/manipulator_h.ros2_control.xacro` to correctly communicate with Dynamixel motors.
The `use_dummy` parameter is required if you don't have a real Manipulator H.

Note that `joint_ids` parameters must be splited by `,`.

```xml
<hardware>
  <plugin>dynamixel_hardware/DynamixelHardware</plugin>
  <param name="usb_port">/dev/ttyUSB0</param>
  <param name="baud_rate">1000000</param>
  <!-- <param name="use_dummy">true</param> -->
</hardware>
```

- Terminal 1

Launch the `ros2_control` manager for the Manipulator H.

```shell
$ ros2 launch manipulator_h_description manipulator_h_ros2_control.launch.py
```

- Terminal 2

Start the `joint_trajectory_controller` and send a `/joint_trajectory_controller/follow_joint_trajectory` goal to move the OpenManipulator-X.

```shell
$ ros2 control switch_controllers --activate joint_state_broadcaster --activate joint_trajectory_controller --deactivate velocity_controller
$ ros2 action send_goal /joint_trajectory_controller/follow_joint_trajectory control_msgs/action/FollowJointTrajectory -f "{
  trajectory: {
    joint_names: [joint1, joint2, joint3, joint4, joint5, joint6],
    points: [
      { positions: [0.1, 0.1, 0.1, 0.1, 0.1, 0.1], time_from_start: { sec: 2 } },
      { positions: [-0.1, -0.1, -0.1, -0.1, -0.1, -0.1], time_from_start: { sec: 4 } },
      { positions: [0, 0, 0, 0, 0, 0], time_from_start: { sec: 6 } }
    ]
  }
}"
```

If you would like to use the velocity control instead, switch to the `velocity_controller` and publish a `/velocity_controller/commands` message to move the OpenManipulator-X.

```shell
$ ros2 control switch_controllers --activate joint_state_broadcaster --deactivate joint_trajectory_controller --activate velocity_controller
$ ros2 topic pub /velocity_controller/commands std_msgs/msg/Float64MultiArray "data: [0.1, 0.1, 0.1, 0.1, 0]"
```

## TODO
- Add the launch file "manipulator_h_ros2_control_with_gripper.launch.py"
- Add gripper as a dependency.