# LW_bachelor_NTNU2024

# Setup and Further Work

This document provides a quick introduction on how the MPC is set up, and what needs/should be done before it is implemented on Lone Wolf ATV. Additionally, it includes a guide for setting up and running the Simulator.



# MPC

The MPC has been developed with the correct ROS2 interface to ensure it can be implemented with the real system. There are a few things that need to be done before the MPC is ready to run on Lone Wolf, which will be summarized here.

The files in the MPC are divided into two files. The Python file for the MPC, `MPC.py`, and the MPC ROS node, `MPC_node_main.py`. `MPC.py` contains the MPC code and is set up as a class. This is included in the Node, where the MPC object is created. This is shown in the code snippet below.

```python
from MPC import MPC_Controller

class mpc_ros_controller(Node):
    def __init__(self):
        ...  # Code omitted for brevity
        self.mpc = MPC_Controller()
```


To calculate the next control input, this command can be used. 


```python
u_t = self.mpc.mpc_generate_step(physical_states, xref_n, yref_n)
```


`physical_states` are the 4 physical states `X`, `Y`, `\psi`, and `v`, and `xref_n` and `yref_n` are lists with X and Y coordinates.



### GPS Coordinates

The GPS coordinates must be converted to a local reference frame. This means converting from minutes, seconds, etc. to a Cartesian coordinate system, [X, Y], where Lone Wolf or another desired point is [0,0].

### Extended Kalman Filter

Depending on the quality of data obtained from Sensors and GPS, it may be necessary to implement an Extended Kalman Filter. Vectornav is a high-quality IMU, so the sensor data may be good enough. Additionally, if RTK is used, the GPS data will also be very good. If only regular GPS is used, EKF should be implemented.

### Path Planning

For testing, a manually specified reference can be used. For full autonomous operation, path planning must be implemented. This should consider obstacles, such as trees, cliffs, etc. It should also attempt to implement speed in the reference, so a trajectory is the reference instead of just points. If the trajectory is not implemented in the path planning algorithm, it is important that speed is embedded as part of the reference.

### Model Adaptation

Check the PDF for a guide on how to adapt the model to measured data! This should be done before testing the MPC to make the behavior more predictable.

### Brakes

The current model has not implemented brakes. This is because it has an aggressive on/off characteristic. This should be measured before implementation. This SHOULD be fixed before Lone Wolf is released autonomously.

### Electronic Emergency Stop

Fix the electronic emergency stop so it applies the brakes. Currently, it only cuts the throttle, and it rolls to a stop. This can lead to dangerous situations.

### Building Workspace on "Skapet"

ChatGPT is a bit unsure about different ROS versions, which often causes confusion. It is recommended to use the official guide as much as possible here.

ROS packages need to be built. The following link is a tutorial on how to create ROS packages.

[Creating Your First ROS2 Package](https://docs.ros.org/en/foxy/Tutorials/Beginner-Client-Libraries/Creating-Your-First-ROS2-Package.html)

This must be done if the code is to be run by ROS by writing `ROS2 run package_name code`.

It is done as follows:

On the Predator PC, ROS can be sourced by typing "2" in the command window. On Skapet, the following should be done:

```bash
source /opt/ros/foxy/setup.bash
```


Replace "ros2_ws" with an appropriate name.


```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
```

Build the package. Replace `--node-name my_node my_package` with relevant names.



```bash
ros2 pkg create --build-type ament_python --node-name my_node my_package
```

Build the package!



```bash
colcon build
```

Source ROS2. This can vary, and it might be wise to check startup.sh for how this is done.



```bash
source install/local_setup.bash
```

Enter the package name 

```bash
cd ros2_ws package_name package_name
```


Make the files executable
```bash
sudo chmod +x file_name
```



The file structure should look like this


```bash
my_ros2_package/
├── my_ros2_package/
│ ├── init.py
│ ├── MPC_node_main.py
│ └── MPC.py
├── setup.py
├── package.xml
├── resource/
│ └── my_ros2_package
└── setup.cfg
```

The `package.xml` file must be modified. Dependencies must be added, as shown here.



```xml
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
  <name>lw_data_collector</name>
  <version>0.0.0</version>
  <description>TODO: Package description</description>
  <maintainer email="jorgen@todo.todo">jorgen</maintainer>
  <license>TODO: License declaration</license>

  <depend>rclpy</depend>
  <depend>std_msgs</depend>
  <depend>geometry_msgs</depend>
  <depend>sensor_msgs</depend>
  <depend>numpy</depend>
  <depend>casadi</depend>


  <test_depend>ament_copyright</test_depend>
  <test_depend>ament_flake8</test_depend>
  <test_depend>ament_pep257</test_depend>
  <test_depend>python3-pytest</test_depend>

  <export>
    <build_type>ament_python</build_type>
  </export>
</package>
```
When all this is fixed, the MPC can be added to the startup script. Before testing, it might be wise to run it manually from Skapet. SSH to control Skapet from an external PC. A modem connected to Skapet is necessary.




# Simulator

1. **Open Terminal**: Start the terminal application on your Linux system, preferably using the LoneWolf KDA computer (Predator Orion 3000) as the files may be found locally on this. Alternatively, the files are uploaded on Azure under LoneWolf2024.

2. **Start MATLAB**: Type the following command in the terminal and press Enter:

    ```bash
    matlab
    ```

3. **Find the parameter file**: In MATLAB, navigate to the following directory:

    ```python
    /home/lonewolf/Documents/LoneWolf2024/Simulator/
    ```

    Find the `LoneWolfparams.m` file in this directory.

4. **Open and run the parameter file**: Open the `LoneWolfparams.m` file in MATLAB and run it.

5. **Open Simulink model**: In the same directory, find the `LoneWolfsimulator.slk` file. Open this file in Simulink.

6. **Run the Simulink model**: In Simulink, click the `Run` button to start the simulation.

7. **Run Python file**: Navigate to the folder containing the following:

    ```bash
    /home/lonewolf/Documents/LoneWolf2024/Simulator/MPC_node_main.py
    ```

8. **Real-time kernel**: If the program does not run in real-time, this can be observed in a scope at the bottom left corner of the simulator in Simulink, comparing time steps with simulator time. This is usually because the real-time kernel is not installed. This is done by typing the following command in the MATLAB command window:

    ```bash
    sldrtkernel -install
    ```


