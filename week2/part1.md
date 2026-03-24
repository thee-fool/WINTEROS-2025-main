# Week 2 — Simulating StarkOS Environments

Last week, you built the foundations of StarkOS — the software systems that allow different parts of a robot to communicate, coordinate, and respond.  
But software alone can’t move, see, or interact with the world.

Before Tony Stark ever trusted a real suit to fly, fight, or even stand upright, he tested everything inside controlled simulation chambers.  
Thrusters were stress-tested in virtual environments.  
Sensors were validated against simulated obstacles.  
Failures happened safely — without destroying hardware.

<p align="center">
  <img src="../Images/TonyStarkTestingSimulation.gif" width="700">
</p>

`"Tony Stark's Hand Repulsor Simulation"`

This week follows that exact philosophy.

Instead of running code directly on real robots, we will use a **virtual testing lab** to **run, test, and observe robotic behavior** in a realistic simulated world.  
Nothing is being physically built yet — we are learning how to *use* simulation as a tool for development, debugging, and validation.

Simulation allows us to:
- test robot behavior safely,
- iterate quickly without hardware damage,
- visualize physics, motion, and sensor data,
- and understand how robots interact with environments before deployment.

---

## Part 1 — Understanding the Simulation Lab (Gazebo Basics)

Gazebo is the simulation environment we will use as our virtual testing lab.

It provides a 3D world where robots, objects, and sensors exist under realistic physics.  
You can place robots into environments, observe how they move, collide, and respond, and control them using ROS — all without touching real hardware.

Think of Gazebo as:
- a flight simulator for robots,
- a sandbox where crashes are free,
- and a bridge between theory and real-world deployment.

In this part, we focus on:
- launching Gazebo,
- understanding what it simulates,
- and how it integrates with ROS 2.

---

## What is Gazebo

Gazebo is a powerful robotics simulation tool that provides a 3D environment for simulating robots, sensors, and objects. It is widely used in the ROS ecosystem for testing and developing robotics algorithms in a realistic virtual environment before deploying them to real hardware.

Gazebo integrates tightly with ROS, enabling simulation and control of robots using ROS topics, services, and actions. In ROS 2, with the latest Gazebo releases, this integration is facilitated by `ros_gz`.

### Key Features of Gazebo

- **3D Physics Engine**  
  Simulates rigid body dynamics, collision detection, gravity, and contact forces using engines like ODE, Bullet, and DART.

- **Realistic Sensors**  
  Simulates cameras, LiDAR, GPS, and other commonly used robotic sensors with configurable parameters.

- **Plugins**  
  Extensible through plugins that allow control of robots, customization of physics behavior, and integration with ROS.

- **Worlds and Models**  
  Enables users to create and load complex environments using pre-built or custom objects and robot models.

---

> We are using the latest LTS ROS 2 distribution.  
> **Gazebo Harmonic** is the recommended version for this setup.

To install Gazebo Harmonic binaries on Ubuntu 24.04, follow the official guide:  
https://gazebosim.org/docs/harmonic/install_ubuntu/

Once installed, verify the setup by running:
```bash
gz sim shapes.sdf
```
If everything works well you should see the following screen:
![image1](https://github.com/user-attachments/assets/251b28ee-61f5-4a17-9a48-b11d3a4c906a)


After Gazebo successfully starts we can install the Gazebo ROS integration with the following command:
```bash
sudo apt install ros-jazzy-ros-gz
```

You can find the official install guide [here](https://gazebosim.org/docs/harmonic/ros_installation/).

## Run Gazebo examples

Let's start again the `gz sim shapes.sdf` example again and let's see what is important on the Gazebo GUI:
![image2](https://github.com/user-attachments/assets/3bc6cd6b-c142-4005-8539-560713510174)


1. Blue - Start and pause the simulation. By default Gazebo starts the simulation paused but if you add the `-r` when you start Gazebo it automatically starts the simulation.
2. Red - You can add basic shapes or lights here and you can move and rotate them.
3. Pink - The model hierarchy, every item in the simulation is shown here, you can check the links (children) of the model, their collision, inertia, etc.
4. yellow - Detailed information of the Gazebo world.
5. Green - Plug-in browser, we'll open useful tools like `Resource Spawner`, `Visualize Lidar`, `Image Display`, etc.

Gazebo has an online model database available [here](https://app.gazebosim.org/), you can browse and download models from here. Normally this online model library is accessible within Gazebo although there might be issues in WSL2 or in virtual machines, so I prepared an offline model library with some basic models.

[download this offline model library](https://drive.google.com/file/d/1tcfoLFReEW1XNHPUAeLpIz2iZXqQBvo_/view?usp=share_link) from Google Drive.

After download unzip it and place it in the home folder of your user.

![alt text][image3]

To let Gazebo know about the offline model library we have to set the `GZ_SIM_RESOURCE_PATH` environmental variable, the best is to add it to the `.bashrc`:
1. Open `.bashrc` with this command `codium ~/.bashrc` . 
2. Paste the command below, save it and source it by `source ~/.bashrc` .
![image3](https://github.com/user-attachments/assets/3f19a57e-c866-4fca-bcaf-7a0abb3b8937)
```bash
export GZ_SIM_RESOURCE_PATH=~/gazebo_models
```

After setting up the offline model library let's open the `empty.sdf` in Gazebo and add a few models through the `Resource Spawner` within the `plug-in browser`(mentioned above):
You won't be able to see the Resource Spawner in Docker, close the World and entity tree by right clicking on it.
![image4](https://github.com/user-attachments/assets/1da77157-f99e-4867-8f73-772014cef59f)



You can also save the world go to the burger menu on the top left `save world`, name it as new_world.sdf.

# Download ROS package

From now, every lesson has a starter package that you can donwload from GitHub. To download the starter package clone the following git repo  to your colcon workspace:
```bash
git clone https://github.com/thee-fool/winteros_week2.git
```

Let's see what's inside the `erc_gazebo_basics` package with the `tree` command!

```bash
.
├── CMakeLists.txt
├── package.xml
├── meshes
│   ├── mecanum_wheel_left.STL
│   ├── mecanum_wheel_right.STL
│   ├── wheel.blend
│   ├── wheel.dae
│   ├── wheel.SLDPRT
│   ├── wheel.STEP
│   └── wheel.STL
├── rviz
│   ├── rviz.rviz
│   └── urdf.rviz
├── urdf
│   └──materials.xacro
└── worlds
    ├── empty.sdf
    └── world.sdf
```

There are a few folders that we didn't met before:
- meshes: this folder contains the 3D models in `dae` format (collada mesh) that we'll use later for our robot's body and wheels. In this lesson it also includes the SolidWorks and Blender models as reference, but it's not needed for the simulation.
- rviz: pre-configured RViz2 layouts that we can use to display the robot's model and the environment
- urdf: URDF = Universal Robot Description Format. We'll create the model of our robots in this folder. It already has a file with color codes and names.
- worlds: we'll store the Gazebo worlds that we use in the simulation. In the next chapter we learn how to create a Gazebo world.

# Creating a Gazebo world

Before we create a new world, let's see how can we open the example `world.sdf`. Let's navigate to the `worlds` folder and run the following command:

```bash
gz sim world.sdf
```

And it should open the example world I created for this package.
![image5](https://github.com/user-attachments/assets/5ce83586-269d-44f7-a5fb-e6bf54b83505)




Now let's switch to the empty template:
```bash
gz sim empty.sdf
```

Build a world you like using the resource spawner and in the end save it into the worlds folder with `sdf` extension.
![image6](https://github.com/user-attachments/assets/c317f382-f6fc-4cd8-be6a-7323df60b840)



## Launch Gazebo world from ROS

After we created the new world file, let's see how can we launch Gazebo and load the world using ROS. First, let's navigate to `launch` folder within the package. Inside this folder let's create a new launch file `world.launch.py`.

```python
import os
from ament_index_python.packages import get_package_share_directory
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument, IncludeLaunchDescription
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch.substitutions import LaunchConfiguration, PathJoinSubstitution, TextSubstitution

def generate_launch_description():
    world_arg = DeclareLaunchArgument(
        'world', default_value='world.sdf',
        description='Name of the Gazebo world file to load'
    )
    
    
    pkg_erc_gazebo_basics = get_package_share_directory('erc_gazebo_basics')
    pkg_ros_gz_sim = get_package_share_directory('ros_gz_sim')
    
    # Update this path to your actual gazebo models directory
    gazebo_models_path = "/home/ubuntu/gazebo_models"
    if os.path.exists(gazebo_models_path):
        os.environ["GZ_SIM_RESOURCE_PATH"] = os.environ.get("GZ_SIM_RESOURCE_PATH", "") + os.pathsep + gazebo_models_path
    
    gazebo_launch = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(
            os.path.join(pkg_ros_gz_sim, 'launch', 'gz_sim.launch.py'),
        ),
        launch_arguments={'gz_args': [PathJoinSubstitution([
            pkg_erc_gazebo_basics,
            'worlds',
            LaunchConfiguration('world')
        ]),
        TextSubstitution(text=' -r -v -v1')],
        'on_exit_shutdown': 'true'}.items()
    )
    
    launchDescriptionObject = LaunchDescription()
    launchDescriptionObject.add_action(world_arg)
    launchDescriptionObject.add_action(gazebo_launch)
    
    return launchDescriptionObject
```

This launch file has one argument, the world file's name - you can change the default value to your new world. It also ensures that the offline Gazebo model folder is added to the environmental variable, and finally it launches Gazebo through the `ros_gz_sim` with the right arguments (note that the simulation will start automatically because of the `-r` flag).



Let's build the workspace - if this is the first time that you build this package source the workspace - and we can launch our file:

```bash
ros2 launch erc_gazebo_basics world.launch.py
```

At this point, the simulation stops being a toy and starts behaving like an actual system.  
One command. One configuration. Everything boots together.

<p align="center">
  <img src="../Images/iron-man2-suit-up.gif" width="700">
</p>

`"Iron man's launchfile" -> his suitcase `

# URDF

URDF is Universal Robot Description Format, it's an XML format for representing a robot model commonly used in ROS, RViz and Gazebo.

However, we still call it URDF, in practice it also includes the functionalaties of `xacro`, in its full name XML macros. With `xacro` we can define re-usable constants, do basic mathematical calculations and substitute complete blocks of our robots with parametrized macros. It can be useful for example in case of a 6 DoF robot arm where all links and joints are identical with different length, diameter, weight, etc.

A robot description (3D model) in URDF is built up as the tree of links and joints that connects links together. A parent link can have multiple children, but a link can only have a single parent.

In the links we can define the mechanical parameters of the link (weight, inertia), the collision shape for the physical simulation and it's visual properties (e.g. detailed 3D models).

In the joints we define the parent and child links and we tell to the simulation what kind of joint do we have (fixed, rotation or linear).

For more detailed tutorials you can check out the official documentation [here](https://docs.ros.org/en/jazzy/Tutorials/Intermediate/URDF/URDF-Main.html).

> _“If it moves, it has a frame. If it has a frame, it belongs in the tree.”_  
> — StarkOS design notes


## Building our robot 1


>This part is optional, if you are interested you can do it. Add this file to urdf folder [dume_bot.urdf](dume_bot.urdf) and jump to [TF Tree](#tf-tree) part.


First of all let's create our robot's URDF in the `urdf` folder with `dume_bot.urdf` name. To start from the bare minimum let's add the following xml code to the file.

```xml
<?xml version='1.0'?>

<robot name="dume_bot" xmlns:xacro="http://www.ros.org/wiki/xacro">

  <!-- STEP 1 - Robot footprint -->
  <link name="base_footprint"></link>

</robot>
```

Now we only have the very first link of our robot without any mechanical, collision or visual properties, there is nothing to see yet.

Let's add a fix joint and the next link - `base_link` - that will be the body of our robot. It's a 40x20x10cm brick with 15kg, the inertia matrix is calculated from these parameters.
![image7](https://github.com/user-attachments/assets/79f039c5-71a1-41ab-b812-97c438af9021)


It's always very important to set realistic values into the inertia matrix, at least the order of magnitude should be in the right range. If the inertia matrix is set to a very unrealistic value it will cause unwanted effects during the physical simulation.

To quickly calculate the inertia matrix from the mechanical parameters you can use various tools, for example [this online calculator](https://www.omnicalculator.com/physics/mass-moment-of-inertia).



We always have to think and verify the values if we use `xacro`, during these lessons I'll always use the numeric values for the better understanding. Let's add the robot body with the above parameters:

```xml
  <!-- STEP 2 - Robot chassis = base_link -->
  <joint name="base_footprint_joint" type="fixed">
    <origin xyz="0 0 0" rpy="0 0 0" />
    <parent link="base_footprint"/>
    <child link="base_link" />
  </joint>

  <link name='base_link'>
    <pose>0 0 0.1 0 0 0</pose>

    <inertial>
      <mass value="15.0"/>
      <origin xyz="0.0 0 0" rpy=" 0 0 0"/>
      <inertia
          ixx="0.0625" ixy="0" ixz="0"
          iyy="0.2125" iyz="0"
          izz="0.25"
      />
    </inertial>

    <collision name='collision'>
      <origin xyz="0 0 0" rpy=" 0 0 0"/> 
      <geometry>
        <box size=".4 .2 .1"/>
      </geometry>
    </collision>

    <visual name='base_link_visual'>
      <origin xyz="0 0 0" rpy=" 0 0 0"/>
      <geometry>
        <box size=".4 .2 .1"/>
      </geometry>
    </visual>
  </link>
```

> Links and joints always have to be inside the <robot> tag of the URDF!

## View the robot in RViz

Now we have a robot body that we can visualize. ROS provides several powerful tools to visualize various data, we already met `rqt` now we meet `RViz`.

We can start it with the `rviz2` command. Then we can add (1) the `Robot Model` view (2), we browse for our URDF file (3) and we type `base_link` as Fixed Frame (4).
![image8](https://github.com/user-attachments/assets/82c86bf0-27c3-48a6-9195-87c91fd647eb)


Although is possible to use RViz like this, it's not the most convinient way. Later, when we add more links and joints we'll have the problem that the we don't have any program running that informs RViz about the right transformation among these links (if the joint isn't fixed).

Instead of this manual usage of RViz, let's move all these tasks into a ROS launch file.

We have to install a few packages first:
```bash
sudo apt install ros-jazzy-urdf
sudo apt install ros-jazzy-urdf-tutorial 
sudo apt install ros-jazzy-urdf-launch
```

Let's try what can we do with the `urdf-tutorial` package:
```bash
ros2 launch urdf_tutorial display.launch.py model:=urdf/01-myfirst.urdf
```

The source of this package is available [here](https://github.com/ros/urdf_tutorial/tree/ros2). This package does exactly what we need, but we cannot build up our tools onto `urdf-tutorial`, it's not suitable, but we can create our own launch file based on this package! Let's create the `check_urdf.launch.py` file in the launch folder with the following content:

```python
import os
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument, IncludeLaunchDescription
from launch.substitutions import LaunchConfiguration, PathJoinSubstitution
from launch_ros.substitutions import FindPackageShare


def generate_launch_description():
    
    pkg_erc_gazebo_basics = FindPackageShare('erc_gazebo_basics')
    default_rviz_config_path = PathJoinSubstitution([pkg_erc_gazebo_basics, 'rviz', 'urdf.rviz'])

    # Show joint state publisher GUI for joints
    gui_arg = DeclareLaunchArgument(name='gui', default_value='true', choices=['true', 'false'],
                                    description='Flag to enable joint_state_publisher_gui')
    
    # RViz config file path
    rviz_arg = DeclareLaunchArgument(name='rvizconfig', default_value=default_rviz_config_path,
                                    description='Absolute path to rviz config file')
    

    # URDF model path within the erc_gazebo_basics package
    model_arg = DeclareLaunchArgument(
        'model', default_value='dume_bot.urdf',
        description='Name of the URDF description to load'
    )

    # Use built-in ROS2 URDF launch package with our own arguments
    urdf = IncludeLaunchDescription(
        PathJoinSubstitution([FindPackageShare('urdf_launch'), 'launch', 'display.launch.py']),
        launch_arguments={
            'urdf_package': 'erc_gazebo_basics',
            'urdf_package_path': PathJoinSubstitution(['urdf', LaunchConfiguration('model')]),
            'rviz_config': LaunchConfiguration('rvizconfig'),
            'jsp_gui': LaunchConfiguration('gui')}.items()
    )

    launchDescriptionObject = LaunchDescription()

    launchDescriptionObject.add_action(gui_arg)
    launchDescriptionObject.add_action(rviz_arg)
    launchDescriptionObject.add_action(model_arg)
    launchDescriptionObject.add_action(urdf)

    return launchDescriptionObject
```

From now, in every lesson we'll create this launch file, because it's extremely useful during development to visualize our URDF. Build the workspace and let's try it:
```bash
ros2 launch erc_gazebo_basics check_urdf.launch.py 
```


![image9](https://github.com/user-attachments/assets/0dc69962-9a84-4f46-b975-56320020498a)




## Building our robot 2

### How to add color

By default everything is rendered in red color in RViz because we were not defining the colors of the links. Let's color the body of our robot orange and the wheels green. To color a link we have to put a `<material>` tag into the `<visual>` tag of each links.

We can use the following tags:
```xml
<material name="orange"/>
<material name="green"/>
```

By default RViz won't be able to understand these color names though, to define these colors we can include the `materials.xacro` that is already in the package of this lesson. Add the following import to the very beginning of the URDF within the `<robot>` tag:
```xml
  <!-- STEP 3 - RViz colors -->
  <xacro:include filename="$(find erc_gazebo_basics)/urdf/materials.xacro" />
```

### Wheel
Let's keep building the differential drive robot by adding the 4 wheels:(color is already added in the wheel)

```xml
<!-- STEP 4 - Wheels -->
  <joint type="continuous" name="front_left_wheel_joint">
    <origin xyz="0.15 0.15 0" rpy="0 0 0"/>
    <child link="front_left_wheel"/>
    <parent link="base_link"/>
    <axis xyz="0 1 0" rpy="0 0 0"/>
    <limit effort="100" velocity="10"/>
    <dynamics damping="1.0" friction="1.0"/>
  </joint>

  <link name='front_left_wheel'>
    <inertial>
      <mass value="5.0"/>
      <origin xyz="0 0 0" rpy="0 1.5707 1.5707"/>
      <inertia
          ixx="0.014" ixy="0" ixz="0"
          iyy="0.014" iyz="0"
          izz="0.025"
      />
    </inertial>

    <collision>
      <origin xyz="0 0 0" rpy="0 1.5707 1.5707"/> 
      <geometry>
        <cylinder radius=".1" length=".05"/>
      </geometry>
    </collision>

    <visual name='front_left_wheel_visual'>
      <origin xyz="0 0 0" rpy="0 1.5707 1.5707"/>
      <geometry>
        <cylinder radius=".1" length=".05"/>
      </geometry>
      <material name="green"/>
    </visual>
  </link>

  <joint type="continuous" name="rear_left_wheel_joint">
    <origin xyz="-0.15 0.15 0" rpy="0 0 0"/>
    <child link="rear_left_wheel"/>
    <parent link="base_link"/>
    <axis xyz="0 1 0" rpy="0 0 0"/>
    <limit effort="100" velocity="10"/>
    <dynamics damping="1.0" friction="1.0"/>
  </joint>

  <link name='rear_left_wheel'>
    <inertial>
      <mass value="5.0"/>
      <origin xyz="0 0 0" rpy="0 1.5707 1.5707"/>
      <inertia
          ixx="0.014" ixy="0" ixz="0"
          iyy="0.014" iyz="0"
          izz="0.025"
      />
    </inertial>

    <collision>
      <origin xyz="0 0 0" rpy="0 1.5707 1.5707"/> 
      <geometry>
        <cylinder radius=".1" length=".05"/>
      </geometry>
    </collision>

    <visual name='rear_left_wheel_visual'>
      <origin xyz="0 0 0" rpy="0 1.5707 1.5707"/>
      <geometry>
        <cylinder radius=".1" length=".05"/>
      </geometry>
      <material name="green"/>
    </visual>
  </link>

  <joint type="continuous" name="front_right_wheel_joint">
    <origin xyz="0.15 -0.15 0" rpy="0 0 0"/>
    <child link="front_right_wheel"/>
    <parent link="base_link"/>
    <axis xyz="0 1 0" rpy="0 0 0"/>
    <limit effort="100" velocity="10"/>
    <dynamics damping="1.0" friction="1.0"/>
  </joint>

  <link name='front_right_wheel'>
    <inertial>
      <mass value="5.0"/>
      <origin xyz="0 0 0" rpy="0 1.5707 1.5707"/>
      <inertia
          ixx="0.014" ixy="0" ixz="0"
          iyy="0.014" iyz="0"
          izz="0.025"
      />
    </inertial>

    <collision>
      <origin xyz="0 0 0" rpy="0 1.5707 1.5707"/> 
      <geometry>
        <cylinder radius=".1" length=".05"/>
      </geometry>
    </collision>

    <visual name='front_right_wheel_visual'>
      <origin xyz="0 0 0" rpy="0 1.5707 1.5707"/>
      <geometry>
        <cylinder radius=".1" length=".05"/>
      </geometry>
      <material name="green"/>
    </visual>
  </link>

  <joint type="continuous" name="rear_right_wheel_joint">
    <origin xyz="-0.15 -0.15 0" rpy="0 0 0"/>
    <child link="rear_right_wheel"/>
    <parent link="base_link"/>
    <axis xyz="0 1 0" rpy="0 0 0"/>
    <limit effort="100" velocity="10"/>
    <dynamics damping="1.0" friction="1.0"/>
  </joint>

  <link name='rear_right_wheel'>
    <inertial>
      <mass value="5.0"/>
      <origin xyz="0 0 0" rpy="0 1.5707 1.5707"/>
      <inertia
          ixx="0.014" ixy="0" ixz="0"
          iyy="0.014" iyz="0"
          izz="0.025"
      />
    </inertial>

    <collision>
      <origin xyz="0 0 0" rpy="0 1.5707 1.5707"/> 
      <geometry>
        <cylinder radius=".1" length=".05"/>
      </geometry>
    </collision>

    <visual name='rear_right_wheel_visual'>
      <origin xyz="0 0 0" rpy="0 1.5707 1.5707"/>
      <geometry>
        <cylinder radius=".1" length=".05"/>
      </geometry>
      <material name="green"/>
    </visual>
  </link>
```

Rebuild the workspace and let's see it using rviz:

```bash
ros2 launch erc_gazebo_basics check_urdf.launch.py 
```

![image10](https://github.com/user-attachments/assets/36dde29b-636f-4516-adb4-c1e22677c518)


## TF Tree

> **Note:** If you skipped building the robot URDF earlier, this section may feel a bit abstract — that’s okay.

The key idea is simple:

Every part of a robot has its own **reference frame** — a coordinate system attached to that part.  
You can think of it as each component describing the world from *its own point of view*.

When data needs to move between parts (position, velocity, orientation), ROS must **transform** that data from one frame to another.  
That’s exactly what the **TF Tree** represents — how all these frames are connected and how information flows between them.

You don’t need to master this now.  
Just remember: robots don’t agree on “where things are” unless TF tells them how to translate between viewpoints.

It's time to get to know another useful tool of ROS, the `TF Tree`. This tool helps visualizing the transformations between the reference frames of the robot. First we need to install the tool:
```bash
sudo apt install ros-jazzy-rqt-tf-tree 
```

After that, let's view our robot with the previous command in RViz:
```bash
ros2 launch erc_gazebo_basics check_urdf.launch.py
```

and in another terminal let's run TF Tree:
```bash
ros2 run rqt_tf_tree rqt_tf_tree
```

> You might experience an issue during the first start of TF Tree, in this case make sure that this rqt plugin is discovered:
> ```bash
> ros2 run rqt_tf_tree rqt_tf_tree --force-discover
> ```


![image12](https://github.com/user-attachments/assets/f7b5b7b0-78d1-4a18-bfac-afb23e940357)

## Load the URDF in Gazebo

Now we have a robot model that we see in RViz, but RViz is just a visualization tool, even if we can rotate some joints with the `joint_state_publisher_gui` it has nothing to do with the physical simulation. To insert our robot model into the Gazebo simulation environment we have to write a new launch file that spawns the robot through the right services of Gazebo.

At this stage, think of RViz as Stark’s design table — blueprints, rotations, and mockups.
Gazebo is the test chamber.

Nothing moves in the real world until the design is *inserted into physics*.
This step is where our robot stops being a diagram and starts existing inside gravity, collisions, and time.


Let's create `spawn_robot.launch.py` in our launch folder:
```python
import os
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument, IncludeLaunchDescription
from launch.conditions import IfCondition
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch.substitutions import LaunchConfiguration, PathJoinSubstitution, Command
from launch_ros.actions import Node
from ament_index_python.packages import get_package_share_directory

def generate_launch_description():

    pkg_erc_gazebo_basics = get_package_share_directory('erc_gazebo_basics')

    gazebo_models_path, ignore_last_dir = os.path.split(pkg_erc_gazebo_basics)
    os.environ["GZ_SIM_RESOURCE_PATH"] += os.pathsep + gazebo_models_path

    rviz_launch_arg = DeclareLaunchArgument(
        'rviz', default_value='true',
        description='Open RViz.'
    )

    world_arg = DeclareLaunchArgument(
        'world', default_value='world.sdf',
        description='Name of the Gazebo world file to load'
    )

    model_arg = DeclareLaunchArgument(
        'model', default_value='dume_bot.urdf',
        description='Name of the URDF description to load'
    )

    # Define the path to your URDF or Xacro file
    urdf_file_path = PathJoinSubstitution([
        pkg_erc_gazebo_basics,  # Replace with your package name
        "urdf",
        LaunchConfiguration('model')  # Replace with your URDF or Xacro file
    ])

    world_launch = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(
            os.path.join(pkg_erc_gazebo_basics, 'launch', 'world.launch.py'),
        ),
        launch_arguments={
        'world': LaunchConfiguration('world'),
        }.items()
    )

    # Launch rviz
    rviz_node = Node(
        package='rviz2',
        executable='rviz2',
        arguments=['-d', os.path.join(pkg_erc_gazebo_basics, 'rviz', 'rviz.rviz')],
        condition=IfCondition(LaunchConfiguration('rviz')),
        parameters=[
            {'use_sim_time': True},
        ]
    )

    # Spawn the URDF model using the `/world/<world_name>/create` service
    spawn_urdf_node = Node(
        package="ros_gz_sim",
        executable="create",
        arguments=[
            "-name", "my_robot",
            "-topic", "robot_description",
            "-x", "0.0", "-y", "0.0", "-z", "0.5", "-Y", "0.0"  # Initial spawn position
        ],
        output="screen",
        parameters=[
            {'use_sim_time': True},
        ]
    )

    robot_state_publisher_node = Node(
        package='robot_state_publisher',
        executable='robot_state_publisher',
        name='robot_state_publisher',
        output='screen',
        parameters=[
            {'robot_description': Command(['xacro', ' ', urdf_file_path]),
             'use_sim_time': True},
        ],
        remappings=[
            ('/tf', 'tf'),
            ('/tf_static', 'tf_static')
        ]
    )

    joint_state_publisher_gui_node = Node(
        package='joint_state_publisher_gui',
        executable='joint_state_publisher_gui',
    )

    launchDescriptionObject = LaunchDescription()

    launchDescriptionObject.add_action(rviz_launch_arg)
    launchDescriptionObject.add_action(world_arg)
    launchDescriptionObject.add_action(model_arg)
    launchDescriptionObject.add_action(world_launch)
    launchDescriptionObject.add_action(rviz_node)
    launchDescriptionObject.add_action(spawn_urdf_node)
    launchDescriptionObject.add_action(robot_state_publisher_node)
    launchDescriptionObject.add_action(joint_state_publisher_gui_node)

    return launchDescriptionObject
```

This launch file will include the `world.launch.py` that we created earlier, so we don't have to start and load the world into Gazebo separately. It will also open RViz with a pre-configured view. We see 3 new nodes that we didn't use (or didn't know that we used) before:
- `create` node of the `ros_gz_sim` package to spawn the robot from the `robot_description` topic into a specific location within the simulation.
- `robot_state_publisher` will convert and load the URDF/xacro into the `robot_description` topic it's providing the transformations between the links using static transforms from the URDF and dynamic transforms from real time `joint_states` topic
- `joint_state_publisher_gui` is a `joint_state_publisher` with the small graphical utility to change joint angles. This node is responsible to update dynamic changes between links through the `joint_states` topic

Although we didn't know before, we've already used the `robot_state_publisher` and `joint_state_publisher_gui` through the `urdf_launch` package that is the basis of our `check_urdf.launch.py`. We can see how these nodes are used in the `description.launch.py` and in the `display.launch.py` files [here](https://github.com/ros/urdf_launch/tree/main/launch).

We have to rebuild the workspace and we can try this new launch file:
```bash
ros2 launch erc_gazebo_basics spawn_robot.launch.py 
```

Right now, nothing publishes odometry for our robot so let's change the fixed frame to the `robot_footprint` in RViz.


![image14](https://github.com/user-attachments/assets/fc98318f-fa65-461f-ad86-53ad197fe3c4)


We see that doesn't matter how we change the wheel joint angles it has no impact on the physical simulation. We did the first step, the robot is spawned into a Gazebo simulation, but the integration just starts from here.

The robot now *exists* inside the simulator — it has mass, collisions, and a place in the world.
But it is still lifeless.

No motors.
No control logic.
No understanding of motion.

Just like a suit frame on the workshop floor — complete in shape, useless without actuators and control.


# Gazebo integration

To finally drive our robot in the physical simulation we have to do 2 things, adding a Gazebo plugin that can move the differential drive robot and bridging messages between ROS and Gazebo. Let's start with the first one:

## Diff drive plugin

Typical differential drive robots are hoverboards, robot vacuum cleaners and robot lawnmovers.

We'll use [the following Gazebo plugin](https://gazebosim.org/api/sim/8/classgz_1_1sim_1_1systems_1_1DiffDrive.html) to drive our robots. The kinematics of the differential robot is very simple, the radius from the center of curvature is `R`, the rate of rotation is `ω` and the wheel speeds are `Vl` and `Vr`. The distance between the wheels is `l`. 

![image15 (2)](https://github.com/user-attachments/assets/063149ba-b29b-451e-bebc-33bf7ed3c39c)


We can describe the kinematics of the differential drive robot [with the following equations](https://www.cs.columbia.edu/~allen/F17/NOTES/icckinematics.pdf):

![image16 (2)](https://github.com/user-attachments/assets/14bb6f0a-89c1-45fb-afca-92bb1a472982)


There are 3 special cases of the above equations:

1. `Vl = Vr`, in this case `R` is infinite and `ω = 0`, the robot is moving straight.
2. `Vl = -Vr`, in this case `R = 0` and the center of curvature is between the 2 wheels. The robot rotates in place.
3. `Vl = 0` or `Vr = 0`, in this case `R = l / 2`, the center of curvature is the standing wheel.


<p align="center">
  <img src="../Images/maths-again.gif" width="700">
</p>

`Idhar Bhi maths aa gaya`

The Gazebo plugin is in one hand responsible for calculating the wheel speeds from the control signal. In the other hand, it also implements inverse kinematics, the robot's odometry is calculated from the integral of the wheels speeds and the wheel distance.

Let's create a `dume_bot.gazebo` file in the URDF folder:
```xml
<?xml version="1.0"?>
<robot>
  <gazebo>
    <plugin
        filename="gz-sim-diff-drive-system"
        name="gz::sim::systems::DiffDrive">
        <!-- Topic for the command input -->
        <topic>/cmd_vel</topic>

        <!-- Wheel joints -->
        <left_joint>front_left_wheel_joint</left_joint>
        <left_joint>rear_left_wheel_joint</left_joint>
        <right_joint>front_right_wheel_joint</right_joint>
        <right_joint>rear_right_wheel_joint</right_joint>

        <!-- Wheel parameters -->
        <wheel_separation>0.3</wheel_separation>
        <wheel_radius>0.1</wheel_radius> 

        <!-- Control gains and limits (optional) -->
        <max_velocity>3.0</max_velocity> 
        <max_linear_acceleration>1</max_linear_acceleration>
        <min_linear_acceleration>-1</min_linear_acceleration>
        <max_angular_acceleration>2</max_angular_acceleration>
        <min_angular_acceleration>-2</min_angular_acceleration>
        <max_linear_velocity>0.5</max_linear_velocity>
        <min_linear_velocity>-0.5</min_linear_velocity>
        <max_angular_velocity>1</max_angular_velocity>
        <min_angular_velocity>-1</min_angular_velocity>
        
        <!-- Other parameters (optional) -->
        <odom_topic>odom</odom_topic> 
        <tf_topic>tf</tf_topic>
        <frame_id>odom</frame_id>
        <child_frame_id>base_footprint</child_frame_id>
        <odom_publish_frequency>30</odom_publish_frequency>
    </plugin>

    <plugin
        filename="gz-sim-joint-state-publisher-system"
        name="gz::sim::systems::JointStatePublisher">
        <topic>joint_states</topic> <!--from <ros><remapping> -->
        <joint_name>front_left_wheel_joint</joint_name>
        <joint_name>front_right_wheel_joint</joint_name>
        <joint_name>rear_left_wheel_joint</joint_name>
        <joint_name>rear_right_wheel_joint</joint_name>
    </plugin>
  </gazebo>
</robot>
```

The `gz-sim-diff-drive-system` plugin is handling the differential drive kinematics, and we will use another plugin `gz-sim-joint-state-publisher-system` to publish joint states from Gazebo to ROS2.

Let's include this new file in our robot's URDF. In the same way how we included the colors, let's add it to the top of our URDF within the `<robot>` tag.

```xml
  <!-- STEP 5 - Gazebo plugin -->
  <xacro:include filename="$(find erc_gazebo_basics)/urdf/dume_bot.gazebo" />
```

Rebuild the workspace and let's try it:

```bash
ros2 launch erc_gazebo_basics spawn_robot.launch.py 
```

We see that odometry is still not published for RViz, but at least in Gazebo we can already drive our robot with the `teleop` plugin:
![image17](https://github.com/user-attachments/assets/dac88c0a-4104-4b5a-b337-311ae56fc769)



But - just like the odometry - this message is also not forwarded between ROS and Gazebo.

## ROS gz bridge

First of all remove the `joint_state_publisher` from the `spawn_robot.launch.py` because as soon as we can forward the messages between ROS and Gazebo, Gazebo will handle updating the `joint_state` topic.

After that we add another node the `parameter_bridge` from the `ros_gz_bridge` package.

```python
    # Node to bridge messages like /cmd_vel and /odom
    gz_bridge_node = Node(
        package="ros_gz_bridge",
        executable="parameter_bridge",
        arguments=[
            "/clock@rosgraph_msgs/msg/Clock[gz.msgs.Clock",
            "/cmd_vel@geometry_msgs/msg/Twist@gz.msgs.Twist",
            "/odom@nav_msgs/msg/Odometry@gz.msgs.Odometry",
            "/joint_states@sensor_msgs/msg/JointState@gz.msgs.Model",
            "/tf@tf2_msgs/msg/TFMessage@gz.msgs.Pose_V"
        ],
        output="screen",
        parameters=[
            {'use_sim_time': True},
        ]
    )
```

> Make sure `ros_gz_bridge` is installed!
> ```bash
> sudo apt install ros-jazzy-ros-gz-bridge
> ```

You can find the detailed documentation of `ros_gz_bridge` [here](https://github.com/gazebosim/ros_gz/tree/ros2/ros_gz_bridge). It explains the syntax of bridging topics in details also you can see what kind of messages can be bridged.

We forward the following topics:
- `/clock`: The topic used for tracking simulation time or any custom time source.
- `/cmd_vel`: We'll control the simulated robot from this ROS topic.
- `/odom`: Gazebo's diff drive plugin provides this odometry topic for ROS consumers.
- `/joint_states`: Gazebo's other plugin provides the dynamic transformation of the wheel joints.
- `/tf`: Gazebo provides the real-time computation of the robot’s pose and the positions of its links, sensors, etc.

At this point, ROS and Gazebo are both running — but they are speaking different languages.

The bridge is the translator.
Commands go in.
Physics comes back.
And suddenly, the simulation becomes controllable from ROS.


> We forward the above messages bi-directionally between ROS2 and Gazebo except `/clock`. Clock should be published by Gazebo only if we are using simulated environment, but if another node already publishes to the `/clock` topic the bi-directional bridge won't be created. In the current complexity of the simulation this is not very important, but later this can cause problems. So we have to make sure that `/clock` is forwarded only in the Gazebo &#8594; ROS2 direction, this we can achieve with using the `[` symbol instead of `@` in the arguments.

Don't forget to add the new node to the `LaunchDescription()` object:

```python
    launchDescriptionObject.add_action(gz_bridge_node)
```

Then rebuild the workspace.

## Driving around

<p align="center">
  <img src="../Images/tony-stark-driving.gif" width="700">
</p>

`Tony Stark Driving Around`

Now it's time to launch the simulation:
```bash
ros2 launch erc_gazebo_basics spawn_robot.launch.py 
```

And in another terminal let's start the `teleop_twist_keyboard`:

We can try to drive the robot from ROS with the `teleop_twist_keyboard` node:

```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard 
```

> If `teleop_twist_keyboard` is not installed yet, you can install it with the following command:
> ```bash
> sudo apt install ros-jazzy-teleop-twist-keyboard
> ```


You will be able to run the bot through the terminal


The friction between the wheels and the ground plane can be unrealistic so we can adjust it inside our URDF, let's add the following physical simulation parameters to the end of our URDF before the `</robot>` tag is closed:

```xml
   <!-- STEP 6 - Gazebo frictions  -->
  <gazebo reference="front_left_wheel">
    <mu1>1.5</mu1>
    <mu2>0.7</mu2>
    <kp>200000.0</kp>
    <kd>5000.0</kd>
    <minDepth>0.002</minDepth>
    <maxVel>0.3</maxVel>
    <fdir1>0 1 0</fdir1>
  </gazebo>

  <gazebo reference="front_right_wheel">
    <mu1>1.5</mu1>
    <mu2>0.7</mu2>
    <kp>200000.0</kp>
    <kd>5000.0</kd>
    <minDepth>0.002</minDepth>
    <maxVel>0.3</maxVel>
    <fdir1>0 1 0</fdir1>
  </gazebo>

  <gazebo reference="rear_left_wheel">
    <mu1>1.5</mu1>
    <mu2>0.7</mu2>
    <kp>200000.0</kp>
    <kd>5000.0</kd>
    <minDepth>0.002</minDepth>
    <maxVel>0.3</maxVel>
    <fdir1>0 1 0</fdir1>
  </gazebo>

  <gazebo reference="rear_right_wheel">
    <mu1>1.5</mu1>
    <mu2>0.7</mu2>
    <kp>200000.0</kp>
    <kd>5000.0</kd>
    <minDepth>0.002</minDepth>
    <maxVel>0.3</maxVel>
    <fdir1>0 1 0</fdir1>
  </gazebo>

  <gazebo reference="base_link">
    <mu1>0.000002</mu1>
    <mu2>0.000002</mu2>
  </gazebo>
```
Built the package and source the workspace, run the robot launch file and teleop in the terminals


## Odometry and Trajectory server

If we use the `rqt_tf_tree` tool that we met earlier, we can see an additional transformation between the `robot_footprint` and the `odom` frame:

![image19](https://github.com/user-attachments/assets/4e491176-09f1-47d4-9baa-b5181898d454)


I created a node that helps visualizing the odometry and the trajectory of the robot. Clone the following repo into your workspace:
go to src and clone this repo 
```bash
git clone https://github.com/MOGI-ROS/mogi_trajectory_server
```

Rebuild you worksapce and also source the environment since we added a new package (or simply open a new terminal window and .bashrc does the job).

Then add the node to the `spawn_robot.launch.py`:


```python
    trajectory_node = Node(
        package='mogi_trajectory_server',
        executable='mogi_trajectory_server',
        name='mogi_trajectory_server',
    )
```


And also add it to the `LaunchDescription()` object:

```python
    launchDescriptionObject.add_action(trajectory_node)
```
>This package provides a node that saves trajectory data using a TransformListener between `reference_frame_id` and `robot_frame_id`, the trajectory is saved internally as a `nav_msgs/Path` and can be obtained through the `trajectory_topic` topic.

>There is some problem with the mogi_trajectory_server it uses bitbots_tf_buffer which does not work as expected therefore we will stick with the goated method which uses tf2_ros, navigate to the `src/mogi_trajectory_server/mogi_trajectory_server/trajectory.py` and replace the whole code with
```python
import rclpy
from rclpy.node import Node
from rclpy.duration import Duration
from nav_msgs.msg import Path
from geometry_msgs.msg import PoseStamped, TransformStamped
from tf2_ros import TransformListener, Buffer
import math

class TrajectoryPublisher(Node):
    def __init__(self):
        super().__init__('trajectory_publisher')
        self.declare_parameter("trajectory_topic", "trajectory")
        self.declare_parameter("reference_frame_id", "odom")
        self.declare_parameter("robot_frame_id", "base_link")
        self.declare_parameter("update_rate", 3.0) # in Hz
        self.declare_parameter("publish_rate", 2.0) # in Hz
        self.declare_parameter("min_distance", 0.1) # in meters

        # TF2 Listener - standard tf2_ros implementation
        self.tf_buffer = Buffer()
        self.tf_listener = TransformListener(self.tf_buffer, self)

        # Default values
        self.update_rate = self.get_parameter("update_rate").value
        self.publish_rate = self.get_parameter("publish_rate").value
        self.min_distance = self.get_parameter("min_distance").value
        self.reference_frame_id = self.get_parameter("reference_frame_id").value
        self.robot_frame_id = self.get_parameter("robot_frame_id").value
        self.use_sim_time = self.get_parameter('use_sim_time').get_parameter_value().bool_value

        # Create separate timers for publisher and transformation
        self.path_pub = self.create_publisher(Path, self.get_parameter("trajectory_topic").value, 10)
        self.publish_timer = self.create_timer(1.0/self.publish_rate, self.publish_trajectory)
        self.transform_timer = self.create_timer(1.0/self.update_rate, self.get_pose)
        
        # Trajectory message header is the same as target frame id
        self.last_pose = None
        self.path = Path()
        self.path.header.frame_id = self.get_parameter("reference_frame_id").value

    def get_pose(self):
        try:
            # Look up the transformation between reference frame and robot frame
            transform: TransformStamped = self.tf_buffer.lookup_transform(
                self.reference_frame_id,
                self.robot_frame_id,
                rclpy.time.Time(),
                timeout=rclpy.duration.Duration(seconds=1.0)
            )

            # This will always use the right clock based on value of use_sim_time
            now = self.get_clock().now().to_msg()

            # Extract the position
            pose = PoseStamped()
            pose.header.stamp = now
            pose.header.frame_id = self.reference_frame_id
            pose.pose.position.x = transform.transform.translation.x
            pose.pose.position.y = transform.transform.translation.y
            pose.pose.position.z = transform.transform.translation.z
            pose.pose.orientation = transform.transform.rotation

            # Append the first pose anyway to the path
            if self.last_pose == None:
                self.path.poses.append(pose)
                self.last_pose = pose
                return

            # Check the distance change in 3D
            distance_moved = math.sqrt(
                (pose.pose.position.x - self.last_pose.pose.position.x) ** 2 +
                (pose.pose.position.y - self.last_pose.pose.position.y) ** 2 +
                (pose.pose.position.z - self.last_pose.pose.position.z) ** 2
            )
            
            if distance_moved >= self.min_distance:
                # Append the pose to the path
                self.path.poses.append(pose)
                self.last_pose = pose

            return

        except Exception as e:
            self.get_logger().warn(f"Could not get transform: {e}")
            return

    def publish_trajectory(self):
        # Create timestamp for the header
        self.path.header.stamp = self.get_clock().now().to_msg()
        # Publish the path
        self.path_pub.publish(self.path)

def main(args=None):
    rclpy.init(args=args)
    trajectory_publisher = TrajectoryPublisher()
    rclpy.spin(trajectory_publisher)
    trajectory_publisher.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```
>Don't forget to build the package and source the workspace.

Then launch the simulation:
```bash
ros2 launch erc_gazebo_basics spawn_robot.launch.py 
```

And in another terminal let's start the `teleop_twist_keyboard`:

```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard 
```

And let's see how it looks like in RViz:

![image20](https://github.com/user-attachments/assets/2cc6c9a9-a308-4429-a566-a74bbf7ef7d4)


We can also see how the nodes are connected to each other using the tool we previously met, `rqt_graph`:

![image21](https://github.com/user-attachments/assets/8438dfed-d917-47ea-8377-d4a9f65a0dee)



# 3D models

Let's make our robot visually more appealing with some 3D models. I already created the 3D models that can be found in the `meshes` folder. We can either use `.stl` files or `.dae` collada meshes. I recommend the collada meshes because then we can individually color certain areas of meshes (e.g. the tyre, the hub and spokes in case of the wheel). With `.stl` files we can only assign a single color for the model.

This is where the robot stops being a box and starts looking like a machine.
Visuals don’t affect physics — but they *do* affect how real the system feels.

Creating a model consists of the following recomennded steps:
1. Create your model in SolidWorks or any other CAD program and save it to an `.stl` file.
2. Import `.stl` to Blender and export it as `.dea`, add the model to the URDF and always check it with the `check_urdf.launch.py`, usually the scale and the centerpoint of the model isn't right.
3. Iteratively rescale and move the model in Blender, always export to the same `.dae` file, when everything is in the right place, color the model in Blender as you wish.


The collada meshes are already available for this robot, so it's time to replace the box and cylinders in the URDF file:

```xml
    <visual name='base_link_visual'>
      <origin xyz="0 0 0" rpy=" 0 0 0"/>
      <geometry>
        <mesh filename = "package://erc_gazebo_basics/meshes/IronMan_ROS_Body.dae"/>
        <!-- <box size=".4 .2 .1"/> -->
      </geometry>
      <!-- <material name="orange"/> -->
    </visual>
```
Similarly do for wheels:

For left front and right rear wheel: 
```xml
<mesh filename = "package://erc_gazebo_basics/meshes/IronManWheelLeftFront.dae"/>
```
and for right front and left rear wheel:
```xml
<mesh filename = "package://erc_gazebo_basics/meshes/IronManWheelRightFront.dae"/>
```

ex: 
```xml
    <visual name='front_left_wheel_visual'>
      <origin xyz="0 0 0" rpy="0 1.5707 1.5707"/>
      <geometry>
        <!-- <cylinder radius=".1" length=".05"/> -->
        <mesh filename = "package://erc_gazebo_basics/meshes/IronManWheelLeftFront.dae"/>
      </geometry>
      <!-- <material name="green"/> -->
    </visual>
  </link>
```

This is the “try it on” moment — the system works, now it finally looks right.


> Note that also `<material>` tag is removed, if we don't remove it, Gazebo will still apply a single color on the model!


![image22](https://github.com/user-attachments/assets/fe7f1bce-afc5-48e0-bf31-61ef3acb550f)

---

### Part 1 — Done

You built a full robot pipeline:
URDF → RViz → Gazebo → ROS control → motion → visualization → real meshes.

This is a real simulated robot now.

<p align="center">
  <img src="../Images/tony-stark-approves.gif" width="700">
</p>

`Tony Stark Approves of Your Work`


Next up: giving the robot awareness — sensors and perception in Gazebo.
👉 **Part 2 — Gazebo with Sensors**
