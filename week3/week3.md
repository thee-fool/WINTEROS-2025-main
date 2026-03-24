# Week 3 — JARVIS: Vision & Navigation

Up until now, your robot could **move** and **sense** the world.  
But sensing alone isn’t enough.

This is the stage where raw data turns into **understanding**.

Before an Iron Man suit can navigate a battlefield or map an unknown structure, it has to interpret what it sees — edges, motion, obstacles, free space. Cameras stop being “images” and start becoming **information**. Sensor data stops being noise and starts forming **maps**.

This week, we step into that layer.

We’ll use **OpenCV** to process camera data and extract meaningful features, and we’ll introduce **navigation and SLAM algorithms** that allow a robot to build a map of its environment while figuring out where it is inside it — all at the same time.

This isn’t about flashy movement anymore.  
It’s about **reasoning**, **localization**, and **decision-making**.

The suit doesn’t just fly.  
It knows *where* it is.

<img width="498" height="189" alt="image" src="https://github.com/user-attachments/assets/b14284a0-a5c5-4d70-b0dc-8078b95f973b" />(iron-man-suits.gif)

## Part 1 - Vision: Teaching Stark OS to See
So far, our robot can move and sense distance — but it’s still blind.

In *Week 3 · Part 1*, we give StarkOS vision.

Using *ROS 2 + OpenCV*, we’ll process live camera feeds, extract useful information from raw pixels, and turn what the robot sees into motion. This is perception in its simplest and most powerful form.

By the end of this part, the robot won’t just drive —
it will *react visually* to its environment.

No mapping yet.
No planning yet.
Just *seeing → deciding → moving.*

<img width="800" height="344" alt="image" src="https://github.com/user-attachments/assets/ef9af9c3-bdb4-4e2b-b3bc-39fcd829a057" />(opencv.gif)

## OpenCV

<img width="341" height="148" alt="image" src="https://github.com/user-attachments/assets/5d9699e7-9152-4f53-8034-13593d1b16af" />



OpenCV (Open-Source Computer Vision Library) is an open-source library that includes several hundreds of computer vision algorithms. It helps us in performing various operations on images very easily.
### Installation and Setup

#### Installing OpenCV </br>
  
 Execute 
 
```bash
pip --version
```
 Ensure that pip is configured with python3.xx . If not you may have to use (```pip3 --version```).
 

Execute either
```bash
pip install opencv-contrib-python
#or
pip install opencv-python
```
Use ```pip3``` in the above commands, if python3 is configured with one of them.

Type ```python3``` in Terminal to start Python interactive session and type following codes there.
```bash
import cv2 as cv
print(cv.__version__)
```
If you're encountering an issue with the cv2 (OpenCV) library and its interaction with the numpy library, then execute the following.
We dont want the most recent version of numpy as it cannot interact with cv_bridge

```bash
pip install numpy==1.23.5
```

If the results are printed out without any errors, congratulations !!! You have installed OpenCV-Python successfully.

You may Install OpenCV from source. (Lengthy process)

Please refer to this [link](https://docs.opencv.org/4.5.0/d2/de6/tutorial_py_setup_in_ubuntu.html). This installation can take some time so have patience.

# Image Processing with OpenCV



(iron man proccesing his camera feed from his suit)

We'll learn how to implement our own node for image processing using ROS and OpenCV. To start, let's create a package alongside our pre-existing package `erc_gazebo_sensors` that we made in the previous week:

## Create new package
Ensure you have finished `erc_gazebo_sensors` and all its content from **Week 2**. 
Let's now create a new package `erc_gazebo_sensors_py` to run our python scripts from:

```bash
cd ~/erc_ws/src

ros2 pkg create --build-type=ament_python erc_gazebo_sensors_py
```

This will create the package inside your `src` folder.

## Creating new node
Open your workspace in VS Code/VS Codium and navigate to the `erc_gazebo_sensors_py` folder.

The directory will mostly look like this:
```bash
erc_gazebo_sensors_py/
├── erc_gazebo_sensors_py
│   └── __init__.py
├── package.xml
├── resource
│   └── erc_gazebo_sensors_py
├── setup.cfg
├── setup.py
└── test
    ├── test_copyright.py
    ├── test_flake8.py
    └── test_pep257.py
```

In this, navigate to the inner folder `erc_gazebo_sensors_py` which has the file `__init__.py`. 

Create a new file of the name `chase_the_ball.py`. Yes, this will make our robot chase a red ball around. Exciting, isn't it!!!

Let's do this step by step

## Step 1
Making the node which subscribes to `/camera/image` topic as that contains the camera feed. It then converts it to OpenCV compatible frame and displays it using OpenCV.

Paste this in `chase_the_ball.py`

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image
from cv_bridge import CvBridge
from geometry_msgs.msg import Twist
import cv2
import numpy as np
import threading

class ImageSubscriber(Node):
    def __init__(self):
        super().__init__('image_subscriber')
        
        # Create a subscriber with a queue size of 1 to only keep the last frame
        self.subscription = self.create_subscription(
            Image,
            'camera/image',
            self.image_callback,
            1  # Queue size of 1
        )

        self.publisher = self.create_publisher(Twist, 'cmd_vel', 10)
        
        # Initialize CvBridge
        self.bridge = CvBridge()
        
        # Variable to store the latest frame
        self.latest_frame = None
        self.frame_lock = threading.Lock()  # Lock to ensure thread safety
        
        # Flag to control the display loop
        self.running = True

        # Start a separate thread for spinning (to ensure image_callback keeps receiving new frames)
        self.spin_thread = threading.Thread(target=self.spin_thread_func)
        self.spin_thread.start()

    def spin_thread_func(self):
        """Separate thread function for rclpy spinning."""
        while rclpy.ok() and self.running:
            rclpy.spin_once(self, timeout_sec=0.05)

    def image_callback(self, msg):
        """Callback function to receive and store the latest frame."""
        # Convert ROS Image message to OpenCV format and store it
        with self.frame_lock:
            self.latest_frame = self.bridge.imgmsg_to_cv2(msg, "bgr8")

    def stop(self):
        """Stop the node and the spin thread."""
        self.running = False
        self.spin_thread.join()

    def display_image(self):
        """Main loop to process and display the latest frame."""
        # Create a single OpenCV window
        cv2.namedWindow("frame", cv2.WINDOW_NORMAL)
        cv2.resizeWindow("frame", 800,600)

        while rclpy.ok():
            # Check if there is a new frame available
            if self.latest_frame is not None:

                # Process the current image
                self.process_image(self.latest_frame)

                # Show the latest frame
                cv2.imshow("frame", self.latest_frame)
                self.latest_frame = None  # Clear the frame after displaying

            # Check for quit key
            if cv2.waitKey(1) & 0xFF == ord('q'):
                self.running = False
                break

        # Close OpenCV window after quitting
        cv2.destroyAllWindows()
        self.running = False

    def process_image(self, img):
        """Image processing task."""
        return

def main(args=None):

    print("OpenCV version: %s" % cv2.__version__)

    rclpy.init(args=args)
    node = ImageSubscriber()
    
    try:
        node.display_image()  # Run the display loop
    except KeyboardInterrupt:
        pass
    finally:
        node.stop()
        node.destroy_node()
        rclpy.shutdown()

if __name__ == '__main__':
    main()
```
> In OpenCV, each frame is handled individually and we have to perform operations such as converting it to a CV2 compatible frame for all the frames we get.

Run 
```bash
chmod +x chase_the_ball.py
```
So we can make it an executable file.

Don't forget to add the entry point in `setup.py`
```python
entry_points={
     'console_scripts': [
         'chase_the_ball = erc_gazebo_sensors_py.chase_the_ball:main'
     ],
 },
```

Build and source your workspace.

Run
```bash
ros2 launch erc_gazebo_sensors spawn_robot.launch.py
```
In another terminal sourcing your workspace run
```bash
ros2 run erc_gazebo_sensors_py chase_the_ball
```

You can run this after launching your robot as you did in the previous week and you should be able to see a camera feed. 
> To end the `chase_the_ball` program, just clicking **X** on the window isn't enough. Close it with `Ctrl + C` in the terminal you opened it in.

Don't worry too much about the threads part of the code. It just ensures that the image processing happens on a separate thread as it is quite an intensive task.

## Step 2

As you may have noticed, the `process_image` function doesn't really do anything right now. Let's fix that

Change your `process_image` function to the one given below:

```python
def process_image(self, img):
    """Image processing task."""
    msg = Twist()
    msg.linear.x = 0.0
    msg.linear.y = 0.0
    msg.linear.z = 0.0
    msg.angular.x = 0.0
    msg.angular.y = 0.0
    msg.angular.z = 0.0

    rows,cols = img.shape[:2]

    R,G,B = self.convert2rgb(img)

    redMask = self.threshold_binary(R, (220, 255))
    stackedMask = np.dstack((redMask, redMask, redMask))
    contourMask = stackedMask.copy()
    crosshairMask = stackedMask.copy()

    # return value of findContours depends on OpenCV version
    (contours, hierarchy) = cv2.findContours(redMask.copy(), 1, cv2.CHAIN_APPROX_NONE)

    # Find the biggest contour (if detected)
    if len(contours) > 0:
        
        c = max(contours, key=cv2.contourArea)
        M = cv2.moments(c)

        # Make sure that "m00" won't cause ZeroDivisionError: float division by zero
        if M["m00"] != 0:
            cx = int(M["m10"] / M["m00"])
            cy = int(M["m01"] / M["m00"])
        else:
            cx, cy = 0, 0

        # Show contour and centroid
        cv2.drawContours(contourMask, contours, -1, (0,255,0), 10)
        cv2.circle(contourMask, (cx, cy), 5, (0, 255, 0), -1)

        # Show crosshair and difference from middle point
        cv2.line(crosshairMask,(cx,0),(cx,rows),(0,0,255),10)
        cv2.line(crosshairMask,(0,cy),(cols,cy),(0,0,255),10)
        cv2.line(crosshairMask,(int(cols/2),0),(int(cols/2),rows),(255,0,0),10)

    # Return processed frames
    return redMask, contourMask, crosshairMask
```

Don't run it quite yet though as we have used some helper functions in the above code that we need to define

#### `convert2RGB` and `threshold_binary`:

Write this function above the `process_image` function.
> Please don't write it inside the process image function. They are two fully separate functions which have no overlap.

```python
def convert2rgb(self, img):
    R = img[:, :, 2]
    G = img[:, :, 1]
    B = img[:, :, 0]

    return R, G, B

def threshold_binary(self, img, thresh=(200, 255)):
    binary = np.zeros_like(img)
    binary[(img >= thresh[0]) & (img <= thresh[1])] = 1

    return binary*255
```

## How does it work?
The essence behind OpenCV is that it dissects each frame of the video into three different photo channels - **Red, Blue and Green**.  
We then perform operations on each of the channels separately using the intensity values of the different colours.

This is classic Stark engineering — break a complex signal into simple components, isolate what matters, and ignore the noise.
No magic. Just math, thresholds, and a lot of iteration.

### Change your `display_image` function to show all the different frames that `process_image` returns

```python
def display_image(self):
    """Main loop to process and display the latest frame."""
    # Create a single OpenCV window
    cv2.namedWindow("frame", cv2.WINDOW_NORMAL)
    cv2.resizeWindow("frame", 800,600)

    while rclpy.ok():
        # Check if there is a new frame available
        if self.latest_frame is not None:

            # Process the current image
            mask, contour, crosshair = self.process_image(self.latest_frame)

            # Show the latest frame
            cv2.imshow("frame", self.latest_frame)
            cv2.imshow("mask", mask)
            cv2.imshow("contour", contour)
            cv2.imshow("crosshair", crosshair)
            self.latest_frame = None  # Clear the frame after displaying

        # Check for quit key
        if cv2.waitKey(1) & 0xFF == ord('q'):
            self.running = False
            break

    # Close OpenCV window after quitting
    cv2.destroyAllWindows()
    self.running = False
```

Okay let's run it once

This time the node will open 4 OpenCV windows and try to find the red ball on the image. Let's add a red ball to the simulation first using the `Resource Spawner` plugin of Gazebo:

<img width="2558" height="1336" alt="image" src="https://github.com/user-attachments/assets/2056ae45-2a0c-4ef5-95d9-99af93cb8815" />

The 4 windows

<img width="2130" height="1156" alt="image" src="https://github.com/user-attachments/assets/0f6d103e-5d61-4bd5-a468-3bf1df941ccd" />

Handling many OpenCV windows can be uncomfortable, so before we start following the ball, let's overlay the output of the image processing on the camera frame:

```python
    # Add small images to the top row of the main image
    def add_small_pictures(self, img, small_images, size=(160, 120)):
    
        x_base_offset = 40
        y_base_offset = 10
    
        x_offset = x_base_offset
        y_offset = y_base_offset
    
        for small in small_images:
            small = cv2.resize(small, size)
            if len(small.shape) == 2:
                small = np.dstack((small, small, small))
    
            img[y_offset: y_offset + size[1], x_offset: x_offset + size[0]] = small
    
            x_offset += size[0] + x_base_offset
    
        return img
```
Let's modify `display_image()` function so we can use the above function

```python
    def display_image(self):
        """Main loop to process and display the latest frame."""
        # Create a single OpenCV window
        cv2.namedWindow("frame", cv2.WINDOW_NORMAL)
        cv2.resizeWindow("frame", 800,600)

        while rclpy.ok():
            # Check if there is a new frame available
            if self.latest_frame is not None:

                # Process the current image
                mask, contour, crosshair = self.process_image(self.latest_frame)

                # Add processed images as small images on top of main image
                result = self.add_small_pictures(self.latest_frame, [mask, contour, crosshair])

                # Show the latest frame
                cv2.imshow("frame", result)
                self.latest_frame = None  # Clear the frame after displaying

            # Check for quit key
            if cv2.waitKey(1) & 0xFF == ord('q'):
                self.running = False
                break

        # Close OpenCV window after quitting
        cv2.destroyAllWindows()
        self.running = False
```

Now, on running it, we can see that all images are in one window, making it much easier to handle.

<img width="801" height="635" alt="image" src="https://github.com/user-attachments/assets/f3dbe9e2-b18c-49c1-bd9a-7e1d971f6ee5" />

Wait but all of this is just to recognize the ball. Where is the code to follow the ball ? 

Add this code snippet in `proccess_image()` function right after creating the crosshair image

```python
            cv2.line(crosshairMask, (cx, 0), (cx, rows), (0, 0, 255), 10)
            cv2.line(crosshairMask, (0, cy), (cols, cy), (0, 0, 255), 10)
            cv2.line(
                crosshairMask,
                (int(cols / 2), 0),
                (int(cols / 2), rows),
                (255, 0, 0),
                10,
            )
            # Chase the ball
            if abs(cols / 2 - cx) > 20:
                msg.linear.x = 0.0
                if cols / 2 > cx:
                    msg.angular.z = 0.2
                else:
                    msg.angular.z = -0.2

            else:
                msg.linear.x = 0.2
                msg.angular.z = 0.0

        else:
            msg.linear.x = 0.0
            msg.angular.z = 0.0
```

The second else is for the  
```python
    if len(countors) > 0:
```

And now the robot should be able to follow the red ball!!
<img width="1281" height="720" alt="image" src="https://github.com/user-attachments/assets/3480e70c-a69b-45de-9c80-23c71bc62e57" />



# Part 2 — Autonomy: Letting StarkOS Decide

In Part 1, we taught StarkOS how to see.
Now, we stop telling it what to do.

In *Week 3 · Part 2*, we move from perception to *autonomous navigation* — where the robot uses sensor data to make decisions on its own.

We’ll explore how robots:

understand their position,

reason about their surroundings,

and move purposefully without manual control.

This is where the system starts behaving less like a remote-controlled machine and more like an *independent agent*.

No flashy tricks — just the fundamentals that power real-world robots.

<img width="800" height="394" alt="image" src="https://github.com/user-attachments/assets/7f713ac4-7fe1-4933-9a2e-8d5631356ca0" />(navigating-ironman.gif)
`Iron man navigating through the city`

## Autonomous Navigation

In this lesson we'll learn how to map the robot's environment, how to do localization on an existing map and we'll learn to use ROS2's navigation stack.

## Download ROS Package
To download the package, go to `Downloads` and clone the repo with the following command:
```bash
git clone https://github.com/Sagarv812/winteros_week3
```

This will make a folder named `winteros_week3` in your Downloads. In that folder there will be three more folders named:
- `erc_ros2_navigation`
- `erc_ros2_navigation_py`
- `erc_trajectory_server`

Copy all these folders and paste them in `~/erc_ws/src`, i.e., inside the `src` folder of you ROS2 workspace. 

Now **build** and **source** your workspace.

You can now test it by running:
```bash
ros2 launch erc_ros2_navigation spawn_robot.launch.py
```

## Mapping
Let's learn how to create the map of the robot's surrounding. In practice we are using SLAM algorithms, SLAM stands for Simultaneous Localization and Mapping. It is a fundamental technique in robotics (and other fields) that allows a robot to:

1. Build a map of an unknown environment (mapping).
2. Track its own pose (position and orientation) within that map at the same time (localization).

Usually SLAM algorithms consists of 4 core functionalities:
1. Sensor inputs
SLAM typically uses sensor data (e.g., LIDAR scans, camera images, or depth sensor measurements) to detect features or landmarks in the environment.
2. State estimation
An internal state (the robot’s pose, including x, y, yaw) is estimated using algorithms like Extended Kalman Filters, Particle Filters, or Graph Optimization.
3. Map building
As the robot moves, it accumulates new sensor data. The SLAM algorithm integrates that data into a global map (2D grid map, 3D point cloud, or other representations).
4. Loop closure
When the robot revisits a previously mapped area, the SLAM algorithm detects that it’s the same place (loop closure). This knowledge is used to reduce accumulated drift and refine both the map and pose estimates.

<img width="1200" height="522" alt="image" src="https://github.com/user-attachments/assets/64d53d98-0577-4541-8c65-489abb7b98bf" />
`Tony Stark mapping the city`

For doing all this, we will use the `slam_toolbox` package that has to be installed first:
```bash
sudo apt update
sudo apt install ros-jazzy-slam-toolbox
```

Now we will make a new launch file for mapping.

Let's also move the RViz related functions into the new launch file from `spawn_robot.launch.py`. Go to that file and comment out:
```python
# launchDescriptionObject.add_action(rviz_launch_arg)
# launchDescriptionObject.add_action(rviz_config_arg)
launchDescriptionObject.add_action(world_arg)
launchDescriptionObject.add_action(model_arg)
launchDescriptionObject.add_action(x_arg)
launchDescriptionObject.add_action(y_arg)
launchDescriptionObject.add_action(yaw_arg)
launchDescriptionObject.add_action(sim_time_arg)
launchDescriptionObject.add_action(world_launch)
# launchDescriptionObject.add_action(rviz_node)
launchDescriptionObject.add_action(spawn_urdf_node)
launchDescriptionObject.add_action(gz_bridge_node)
launchDescriptionObject.add_action(gz_image_bridge_node)
launchDescriptionObject.add_action(relay_camera_info_node)
launchDescriptionObject.add_action(robot_state_publisher_node)
launchDescriptionObject.add_action(trajectory_node)
launchDescriptionObject.add_action(ekf_node)
```
> Note: You can add a comment or make a pre-existing a line a comment in python by adding a '#' before it.

Also change the `reference_frame_id` of `erc_trajectory_server` from `odom` to `map` becase this will be our new reference frame when we have a map!

```python
trajectory_node = Node(
    package='erc_trajectory_server',
    executable='erc_trajectory_server',
    name='erc_trajectory_server',
    parameters=[{'reference_frame_id': 'map'}] # Change in this line
)
```

Let's create `mapping.launch.py`. Create the file inside the `launch` folder of `erc_ros2_navigation`:

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

    pkg_erc_ros2_navigation = get_package_share_directory('erc_ros2_navigation')

    gazebo_models_path, ignore_last_dir = os.path.split(pkg_erc_ros2_navigation)
    os.environ["GZ_SIM_RESOURCE_PATH"] += os.pathsep + gazebo_models_path

    rviz_launch_arg = DeclareLaunchArgument(
        'rviz', default_value='true',
        description='Open RViz'
    )

    rviz_config_arg = DeclareLaunchArgument(
        'rviz_config', default_value='mapping.rviz',
        description='RViz config file'
    )

    sim_time_arg = DeclareLaunchArgument(
        'use_sim_time', default_value='True',
        description='Flag to enable use_sim_time'
    )

    # Path to the Slam Toolbox launch file
    slam_toolbox_launch_path = os.path.join(
        get_package_share_directory('slam_toolbox'),
        'launch',
        'online_async_launch.py'
    )

    slam_toolbox_params_path = os.path.join(
        get_package_share_directory('erc_ros2_navigation'),
        'config',
        'slam_toolbox_mapping.yaml'
    )

    # Launch rviz
    rviz_node = Node(
        package='rviz2',
        executable='rviz2',
        arguments=['-d', PathJoinSubstitution([pkg_erc_ros2_navigation, 'rviz', LaunchConfiguration('rviz_config')])],
        condition=IfCondition(LaunchConfiguration('rviz')),
        parameters=[
            {'use_sim_time': LaunchConfiguration('use_sim_time')},
        ]
    )

    slam_toolbox_launch = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(slam_toolbox_launch_path),
        launch_arguments={
                'use_sim_time': LaunchConfiguration('use_sim_time'),
                'slam_params_file': slam_toolbox_params_path,
        }.items()
    )

    launchDescriptionObject = LaunchDescription()

    launchDescriptionObject.add_action(rviz_launch_arg)
    launchDescriptionObject.add_action(rviz_config_arg)
    launchDescriptionObject.add_action(sim_time_arg)
    launchDescriptionObject.add_action(rviz_node)
    launchDescriptionObject.add_action(slam_toolbox_launch)

    return launchDescriptionObject
```

Build the workspace and open two terminals. Make sure to **source** your workspace in both. 

In one terminal,
```bash
ros2 launch erc_ros2_navigation spawn_robot.launch.py
```

And in another,
```bash
ros2 launch erc_ros2_navigation mapping.launch.py
```

You will be able to see an additional frame `map` over the `odom` odometry frame. This can also be visualized in RViz.
<img width="2560" height="1337" alt="image" src="https://github.com/user-attachments/assets/d93eea20-f820-4a9e-bfc8-91bb2717a066" />

With SLAM Toolbox we can also save the maps, we have two options:
1) `Save Map`: The map is saved as a `.pgm` file and a `.yaml` file. This is a black and white image file that can be used with other ROS nodes for localization as we will see later. Since it's only an image file it's impossible to continue the mapping with such a file because SLAM Toolbox handles the map in the background as a graph that cannot be restored from an image.
2) `Serialize Map`: With this feature we can serialize and later deserialize SLAM Toolbox's graph, so it can be loaded and the mapping can be continued. Although other ROS nodes won't be able to read or use it for localization.

<img width="2560" height="1339" alt="image" src="https://github.com/user-attachments/assets/cd117ed9-ef20-4f09-a543-f0fa66617e15" />

After saving a serialized map next time we can load (deserialize it):

<img width="2558" height="1337" alt="image" src="https://github.com/user-attachments/assets/e948762b-5e72-4343-9af8-b9a323242bb6" />


And we can also load the map that is in the starter package of this lesson:

<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/bc7b2163-ffe7-4f4b-9dd2-ecb04d6d5809" />

The paths provided are relative, assuming that your packages are located in the src directory and you are executing mapping.launch.py from the workspace root. If the launch fails, verify your current working directory and ensure the relative path to the maps is correct from that location.

Best Practice: When saving files, specify the full directory path rather than just the filename to ensure data is stored in the intended location.


## Localization

While mapping was the process of creating a representation (a map) of an environment. Localization is the process by which a robot determines its own position and orientation within a known environment (map). In other words:

- The environment or map is typically already available or pre-built.
- The robot’s task is to figure out “Where am I?” or “Which direction am I facing?” using sensor data, often by matching its current perceptions to the known map.

The suit already knows the city — now it just needs to know where it’s standing.

### Localization with AMCL

AMCL (Adaptive Monte Carlo Localization) is a particle filter–based 2D localization algorithm. The robot’s possible poses (position + orientation in 2D) are represented by a set of particles. It adaptively samples the robot’s possible poses according to sensor readings and motion updates, converging on an accurate estimate of where the robot is within a known map.

Jarvis isn’t guessing — he’s narrowing down possibilities until only one makes sense.

AMCL is part of the ROS2 navigation stack, let's install it first:
```bash
sudo apt install ros-jazzy-nav2-bringup 
sudo apt install ros-jazzy-nav2-amcl
```

And then let's create a new launch file `localization.launch.py`:
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

    pkg_erc_ros2_navigation = get_package_share_directory('erc_ros2_navigation')

    gazebo_models_path, ignore_last_dir = os.path.split(pkg_erc_ros2_navigation)
    os.environ["GZ_SIM_RESOURCE_PATH"] += os.pathsep + gazebo_models_path

    rviz_launch_arg = DeclareLaunchArgument(
        'rviz', default_value='true',
        description='Open RViz'
    )

    rviz_config_arg = DeclareLaunchArgument(
        'rviz_config', default_value='localization.rviz',
        description='RViz config file'
    )

    sim_time_arg = DeclareLaunchArgument(
        'use_sim_time', default_value='True',
        description='Flag to enable use_sim_time'
    )

    # Path to the Slam Toolbox launch file
    nav2_localization_launch_path = os.path.join(
        get_package_share_directory('nav2_bringup'),
        'launch',
        'localization_launch.py'
    )

    localization_params_path = os.path.join(
        get_package_share_directory('erc_ros2_navigation'),
        'config',
        'amcl_localization.yaml'
    )

    map_file_path = os.path.join(
        get_package_share_directory('erc_ros2_navigation'),
        'maps',
        'my_map.yaml'
    )

    # Launch rviz
    rviz_node = Node(
        package='rviz2',
        executable='rviz2',
        arguments=['-d', PathJoinSubstitution([pkg_erc_ros2_navigation, 'rviz', LaunchConfiguration('rviz_config')])],
        condition=IfCondition(LaunchConfiguration('rviz')),
        parameters=[
            {'use_sim_time': LaunchConfiguration('use_sim_time')},
        ]
    )

 
    localization_launch = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(nav2_localization_launch_path),
        launch_arguments={
                'use_sim_time': LaunchConfiguration('use_sim_time'),
                'params_file': localization_params_path,
                'map': map_file_path,
        }.items()
    )

    launchDescriptionObject = LaunchDescription()

    launchDescriptionObject.add_action(rviz_launch_arg)
    launchDescriptionObject.add_action(rviz_config_arg)
    launchDescriptionObject.add_action(sim_time_arg)
    launchDescriptionObject.add_action(rviz_node)
    launchDescriptionObject.add_action(localization_launch)

    return launchDescriptionObject
```

Following the same procedure as earlier, this file should be made in the `launch` folder. Now **build** the workspace and **source** it in both the terminals.

In one terminal,
```bash
ros2 launch erc_ros2_navigation spawn_robot.launch.py
```

And in another,
```bash
ros2 launch erc_ros2_navigation localization.launch.py
```

To start using AMCL, we have to provide an initial pose to the `/initialpose` topic. It's basically like telling the robot that hey i'm starting here at these coordinates. We can use RViz's built in tool for that.

<img width="2560" height="1334" alt="image" src="https://github.com/user-attachments/assets/a0b689dd-34c0-43d3-8772-e60b4538ae15" />

This will initialize AMCL's particles around the initial pose which can be displayed in RViz as a particle cloud where each particle rerpresnts a pose (position + orientation in 2D).

<img width="2560" height="1334" alt="image" src="https://github.com/user-attachments/assets/b65f3e6c-4d08-4b2b-a814-0a1f83198f4d" />

The main purpose of the localization algorithm is establishing the transformation between the fixed map and the robot's odometry frame based on real time sensor data. We can visualize this in RViz as we saw it during mapping:

<img width="2560" height="1334" alt="image" src="https://github.com/user-attachments/assets/9ba25a40-2275-48d8-a7a2-ed75e917af2e" />

## Localization with SLAM toolbox

It's possible to use SLAM toolbox in localization mode, it requires a small adjustment on the parameters which is already part of this lesson, `slam_toolbox_localization.yaml`. This requires the path to the serialized map file.

> Make sure the `map_file_name` parameter is changed to the path on your machine to the serialized map file!

Let's create the launch file for localization, `localization_slam_toolbox.launch.py`, it's very similar to the SLAM toolbox mapping launch file:

```python
import os
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument, IncludeLaunchDescription, GroupAction
from launch.conditions import IfCondition
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch.substitutions import LaunchConfiguration, PathJoinSubstitution, Command
from launch_ros.actions import Node
from ament_index_python.packages import get_package_share_directory

def generate_launch_description():

    pkg_erc_ros2_navigation = get_package_share_directory('erc_ros2_navigation')

    gazebo_models_path, ignore_last_dir = os.path.split(pkg_erc_ros2_navigation)
    os.environ["GZ_SIM_RESOURCE_PATH"] += os.pathsep + gazebo_models_path

    rviz_launch_arg = DeclareLaunchArgument(
        'rviz', default_value='true',
        description='Open RViz'
    )

    rviz_config_arg = DeclareLaunchArgument(
        'rviz_config', default_value='mapping.rviz',
        description='RViz config file'
    )

    sim_time_arg = DeclareLaunchArgument(
        'use_sim_time', default_value='True',
        description='Flag to enable use_sim_time'
    )

    # Path to the Slam Toolbox launch file
    slam_toolbox_launch_path = os.path.join(
        get_package_share_directory('slam_toolbox'),
        'launch',
        'localization_launch.py'
    )

    slam_toolbox_params_path = os.path.join(
        get_package_share_directory('erc_ros2_navigation'),
        'config',
        'slam_toolbox_localization.yaml'
    )

    # Launch rviz
    rviz_node = Node(
        package='rviz2',
        executable='rviz2',
        arguments=['-d', PathJoinSubstitution([pkg_erc_ros2_navigation, 'rviz', LaunchConfiguration('rviz_config')])],
        condition=IfCondition(LaunchConfiguration('rviz')),
        parameters=[
            {'use_sim_time': LaunchConfiguration('use_sim_time')},
        ]
    )

    slam_toolbox_launch = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(slam_toolbox_launch_path),
        launch_arguments={
                'use_sim_time': LaunchConfiguration('use_sim_time'),
                'slam_params_file': slam_toolbox_params_path,
        }.items()
    )

    launchDescriptionObject = LaunchDescription()

    launchDescriptionObject.add_action(rviz_launch_arg)
    launchDescriptionObject.add_action(rviz_config_arg)
    launchDescriptionObject.add_action(sim_time_arg)
    launchDescriptionObject.add_action(rviz_node)
    launchDescriptionObject.add_action(slam_toolbox_launch)

    return launchDescriptionObject

```

Rebuild the workspace and try it, we'll see that the map keeps updating unlike with AMCL, this is the normal behavior of the localization with SLAM toolbox. According to [this paper](https://joss.theoj.org/papers/10.21105/joss.02783), the localization mode does continue to update the pose-graph with new constraints and nodes, but the updated map expires over some time. SLAM toolbox describes it as "elastic", which means it holds the updated graph for some amount of time, but does not add it to the permanent graph.

<img width="2560" height="1338" alt="image" src="https://github.com/user-attachments/assets/a0f39aa7-acf7-4e4b-8bbb-607287ef4a49" />

# Navigation

Navigation in robotics is the overall process that enables a robot to move from one location to another in a safe, efficient, and autonomous manner. It typically involves:
1.	Knowing where the robot is (localization or SLAM),
2.	Knowing where it needs to go (a goal pose or waypoint),
3.	Planning a path to reach that goal (path planning), and
4.	Moving along that path while avoiding dynamic and static obstacles (motion control and obstacle avoidance).

ROS's nav2 navigation stack implements the above points 2 to 4, for the first point we already met several possible solutions.

Let's create a launch file that uses both AMCL and the nav2 navigation stack, `navigation.launch.py`:

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

    pkg_erc_ros2_navigation = get_package_share_directory('erc_ros2_navigation')

    gazebo_models_path, ignore_last_dir = os.path.split(pkg_erc_ros2_navigation)
    os.environ["GZ_SIM_RESOURCE_PATH"] += os.pathsep + gazebo_models_path

    rviz_launch_arg = DeclareLaunchArgument(
        'rviz', default_value='true',
        description='Open RViz'
    )

    rviz_config_arg = DeclareLaunchArgument(
        'rviz_config', default_value='navigation.rviz',
        description='RViz config file'
    )

    sim_time_arg = DeclareLaunchArgument(
        'use_sim_time', default_value='True',
        description='Flag to enable use_sim_time'
    )

    # Path to the Slam Toolbox launch file
    nav2_localization_launch_path = os.path.join(
        get_package_share_directory('nav2_bringup'),
        'launch',
        'localization_launch.py'
    )

    nav2_navigation_launch_path = os.path.join(
        get_package_share_directory('nav2_bringup'),
        'launch',
        'navigation_launch.py'
    )

    localization_params_path = os.path.join(
        get_package_share_directory('erc_ros2_navigation'),
        'config',
        'amcl_localization.yaml'
    )

    navigation_params_path = os.path.join(
        get_package_share_directory('erc_ros2_navigation'),
        'config',
        'navigation.yaml'
    )

    map_file_path = os.path.join(
        get_package_share_directory('erc_ros2_navigation'),
        'maps',
        'my_map.yaml'
    )

    # Launch rviz
    rviz_node = Node(
        package='rviz2',
        executable='rviz2',
        arguments=['-d', PathJoinSubstitution([pkg_erc_ros2_navigation, 'rviz', LaunchConfiguration('rviz_config')])],
        condition=IfCondition(LaunchConfiguration('rviz')),
        parameters=[
            {'use_sim_time': LaunchConfiguration('use_sim_time')},
        ]
    )

    localization_launch = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(nav2_localization_launch_path),
        launch_arguments={
                'use_sim_time': LaunchConfiguration('use_sim_time'),
                'params_file': localization_params_path,
                'map': map_file_path,
        }.items()
    )

    navigation_launch = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(nav2_navigation_launch_path),
        launch_arguments={
                'use_sim_time': LaunchConfiguration('use_sim_time'),
                'params_file': navigation_params_path,
        }.items()
    )

    launchDescriptionObject = LaunchDescription()

    launchDescriptionObject.add_action(rviz_launch_arg)
    launchDescriptionObject.add_action(rviz_config_arg)
    launchDescriptionObject.add_action(sim_time_arg)
    launchDescriptionObject.add_action(rviz_node)
    launchDescriptionObject.add_action(localization_launch)
    launchDescriptionObject.add_action(navigation_launch)

    return launchDescriptionObject
```

We'll need 2 terminals as before, one for the simulation:
```bash
ros2 launch erc_ros2_navigation spawn_robot.launch.py
```

And in another terminal we launch the new `navigation.launch.py`:

```bash
ros2 launch erc_ros2_navigation navigation.launch.py
```

We are using AMCL, so first we'll have to publish an initial pose, then we have to tell the pose goal to the navigation stack. For that we can also use RViz's other built-in feature:

<img width="2560" height="1335" alt="image" src="https://github.com/user-attachments/assets/870a8b60-ae1d-43e6-b3b3-24bb0ba6a164" />


As soon as the pose goal is received the navigation stack plans a global path to the goal and the controller ensures locally that the robot follows the global path while it avoids dynamic obstacles. The controller calculates a cost map around the robot that determines the ideal trajectory of the robot. If there aren't any obstacles around the robot this cost map weighs the global plan. 

Target locked. Calculating path.

<img width="2560" height="1336" alt="image" src="https://github.com/user-attachments/assets/b2b9a6d4-cf0c-407a-b1d4-a6f480d21d63" />


If obstacles are detected around the robot those can be visualized as a cost map too:

<img width="2560" height="1335" alt="image" src="https://github.com/user-attachments/assets/0d7df78c-1749-4a61-a619-f0a2b9bc32c8" />

## Waypoint navigation

We can use the navigation stack for waypoint navigation, this can be done through the GUI of RViz or writing a custom node. Let's start with the first one.

We'll need 2 terminals as before, one for the simulation:
```bash
ros2 launch erc_ros2_navigation spawn_robot.launch.py
```

And in another terminal we launch the `navigation.launch.py`:

```bash
ros2 launch erc_ros2_navigation navigation.launch.py
```
First, we have to make sure that the `Nav2 Goal` toolbar is added to RViz! If not, we can add it under the `+` sign.
<img width="1459" height="1229" alt="image" src="https://github.com/user-attachments/assets/36b45023-921c-4e44-9411-c6034bcaf488" />


Then we have to switch nav2 to waypoint following mode:
<img width="715" height="513" alt="image" src="https://github.com/user-attachments/assets/a2dfcb48-34a0-4156-b710-67038f527a92" />


Using the `Nav2 Goal` tool we can define the waypoints:
<img width="2560" height="1334" alt="image" src="https://github.com/user-attachments/assets/d38af408-bc20-4b0e-a2d5-19e40b5cdd08" />


And when we are done with the waypoints we can start the navigation through them:
<img width="2560" height="1337" alt="image" src="https://github.com/user-attachments/assets/79b9b18d-6f6e-4ee7-be90-45e6fa9e8a06" />

<img width="2560" height="1332" alt="image" src="https://github.com/user-attachments/assets/90c1a146-dee7-4340-8a32-859cb5426346" />

It's possible to run multiple loops through the waypoints and it's also possible to save and load the waypoints. One example is already in the `config` folder: `waypoints.yaml`. You can see a video about it here(optional):

<a href="https://youtu.be/ED6AXnAR2sc"><img width="1280" height="719" alt="image" src="https://github.com/user-attachments/assets/5ee9bf49-567a-41f1-b036-dc926dc25889" /></a> 

It's also possible to follow waypoints through the nav2 navigation stack's API with a custom node. Let's create `follow_waypoints.py` in the `erc_ros2_navigation_py` package:

```python
import rclpy
from rclpy.node import Node
from geometry_msgs.msg import PoseStamped
from nav2_msgs.action import FollowWaypoints
from rclpy.action import ActionClient
from tf_transformations import quaternion_from_euler

class WaypointFollower(Node):
    def __init__(self):
        super().__init__('waypoint_follower')
        self._action_client = ActionClient(self, FollowWaypoints, 'follow_waypoints')

    def define_waypoints(self):
        waypoints = []

        # Waypoint 1
        wp1 = PoseStamped()
        wp1.header.frame_id = 'map'
        wp1.pose.position.x = 6.0
        wp1.pose.position.y = 1.5
        q = quaternion_from_euler(0, 0, 0)
        wp1.pose.orientation.x = q[0]
        wp1.pose.orientation.y = q[1]
        wp1.pose.orientation.z = q[2]
        wp1.pose.orientation.w = q[3]
        waypoints.append(wp1)

        return waypoints

    def send_goal(self):
        waypoints = self.define_waypoints()
        goal_msg = FollowWaypoints.Goal()
        goal_msg.poses = waypoints

        self._action_client.wait_for_server()
        self._send_goal_future = self._action_client.send_goal_async(
            goal_msg, feedback_callback=self.feedback_callback
        )
        self._send_goal_future.add_done_callback(self.goal_response_callback)

    def goal_response_callback(self, future):
        goal_handle = future.result()
        if not goal_handle.accepted:
            self.get_logger().info('Goal rejected.')
            return

        self.get_logger().info('Goal accepted.')
        self._get_result_future = goal_handle.get_result_async()
        self._get_result_future.add_done_callback(self.get_result_callback)

    def feedback_callback(self, feedback_msg):
        feedback = feedback_msg.feedback
        current_waypoint = feedback.current_waypoint
        self.get_logger().info(f'Navigating to waypoint {current_waypoint}')

    def get_result_callback(self, future):
        result = future.result().result
        self.get_logger().info('Waypoint following completed.')
        rclpy.shutdown()

def main(args=None):
    rclpy.init(args=args)
    waypoint_follower = WaypointFollower()
    waypoint_follower.send_goal()
    rclpy.spin(waypoint_follower)

if __name__ == '__main__':
    main()
```

The definition of the waypoints are identical to the config file we have, we define the `x`, `y` and `z` position and the orientation in a quaternion.

Let's add the entry point to the `setup.py`:
```python
    entry_points={
        'console_scripts': [
            'send_initialpose = erc_ros2_navigation_py.send_initialpose:main',
            'slam_toolbox_load_map = erc_ros2_navigation_py.slam_toolbox_load_map:main',
            'follow_waypoints = erc_ros2_navigation_py.follow_waypoints:main',
        ],
    },
```

Build the workspace, and we'll need 3 terminals this time, one for the simulation:
```bash
ros2 launch erc_ros2_navigation spawn_robot.launch.py
```

Another terminal to launch the `navigation.launch.py`:

```bash
ros2 launch erc_ros2_navigation navigation.launch.py
```

And in the third one:
```bash
ros2 run erc_ros2_navigation_py follow_waypoints
```

We can add more waypoints easily:
```python
...
        # Waypoint 2
        wp2 = PoseStamped()
        wp2.header.frame_id = 'map'
        wp2.pose.position.x = -2.0
        wp2.pose.position.y = -8.0
        q = quaternion_from_euler(0, 0, 1.57)
        wp2.pose.orientation.x = q[0]
        wp2.pose.orientation.y = q[1]
        wp2.pose.orientation.z = q[2]
        wp2.pose.orientation.w = q[3]
        waypoints.append(wp2)

        # Waypoint 3
        wp3 = PoseStamped()
        wp3.header.frame_id = 'map'
        wp3.pose.position.x = 0.0
        wp3.pose.position.y = 0.0
        q = quaternion_from_euler(0, 0, 0)
        wp3.pose.orientation.x = q[0]
        wp3.pose.orientation.y = q[1]
        wp3.pose.orientation.z = q[2]
        wp3.pose.orientation.w = q[3]
        waypoints.append(wp3)

        # Add more waypoints as needed
...
```

<a href="https://youtu.be/3OhAyDFqBIs"><img width="1280" height="719" alt="image" src="https://github.com/user-attachments/assets/138b4a0f-0b9d-4937-9bf9-ccf708b6900b" /></a> 

## Navigation with SLAM

As we saw to navigate a mobile robot we need to know 2 things:
1.	Knowing where the robot is (localization or SLAM),
2.	Knowing where it needs to go (a goal pose or waypoint)

In the previous examples we used localization on a know map, but it's also possible to navigate together with an online SLAM. It means we don't know the complete environment around the robot but we can already navigate in the known surrounding.

Let's create a `navigation_with_slam.launch.py` launch file where we start both the SLAM toolbox and the navigation stack:

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

    pkg_erc_ros2_navigation = get_package_share_directory('erc_ros2_navigation')

    gazebo_models_path, ignore_last_dir = os.path.split(pkg_erc_ros2_navigation)
    os.environ["GZ_SIM_RESOURCE_PATH"] += os.pathsep + gazebo_models_path

    rviz_launch_arg = DeclareLaunchArgument(
        'rviz', default_value='true',
        description='Open RViz'
    )

    rviz_config_arg = DeclareLaunchArgument(
        'rviz_config', default_value='navigation.rviz',
        description='RViz config file'
    )

    sim_time_arg = DeclareLaunchArgument(
        'use_sim_time', default_value='True',
        description='Flag to enable use_sim_time'
    )

    nav2_navigation_launch_path = os.path.join(
        get_package_share_directory('nav2_bringup'),
        'launch',
        'navigation_launch.py'
    )

    navigation_params_path = os.path.join(
        get_package_share_directory('erc_ros2_navigation'),
        'config',
        'navigation.yaml'
    )

    slam_toolbox_params_path = os.path.join(
        get_package_share_directory('erc_ros2_navigation'),
        'config',
        'slam_toolbox_mapping.yaml'
    )

    # Launch rviz
    rviz_node = Node(
        package='rviz2',
        executable='rviz2',
        arguments=['-d', PathJoinSubstitution([pkg_erc_ros2_navigation, 'rviz', LaunchConfiguration('rviz_config')])],
        condition=IfCondition(LaunchConfiguration('rviz')),
        parameters=[
            {'use_sim_time': LaunchConfiguration('use_sim_time')},
        ]
    )

    # Path to the Slam Toolbox launch file
    slam_toolbox_launch_path = os.path.join(
        get_package_share_directory('slam_toolbox'),
        'launch',
        'online_async_launch.py'
    )

    slam_toolbox_launch = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(slam_toolbox_launch_path),
        launch_arguments={
                'use_sim_time': LaunchConfiguration('use_sim_time'),
                'slam_params_file': slam_toolbox_params_path,
        }.items()
    )

    navigation_launch = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(nav2_navigation_launch_path),
        launch_arguments={
                'use_sim_time': LaunchConfiguration('use_sim_time'),
                'params_file': navigation_params_path,
        }.items()
    )

    launchDescriptionObject = LaunchDescription()

    launchDescriptionObject.add_action(rviz_launch_arg)
    launchDescriptionObject.add_action(rviz_config_arg)
    launchDescriptionObject.add_action(sim_time_arg)
    launchDescriptionObject.add_action(rviz_node)
    launchDescriptionObject.add_action(slam_toolbox_launch)
    launchDescriptionObject.add_action(navigation_launch)

    return launchDescriptionObject
```

Build the workspace and let's try it!

We'll need 2 terminals as before, one for the simulation:
```bash
ros2 launch erc_ros2_navigation spawn_robot.launch.py
```

And in another terminal we launch the new `navigation_with_slam.launch.py`:

```bash
ros2 launch erc_ros2_navigation navigation_with_slam.launch.py
```

<img width="2560" height="1333" alt="image" src="https://github.com/user-attachments/assets/905cc6c3-6059-4061-81d1-929e2bf3cf3d" />

<a href="https://youtu.be/gZrYEP2ctfY"><img width="1281" height="719" alt="image" src="https://github.com/user-attachments/assets/611a4391-51e1-46e6-81ac-f473b881cd70" /></a> 

<img width="498" height="289" alt="image" src="https://github.com/user-attachments/assets/9de2b882-2423-4e0c-9478-308c49ab3281" />(tony-proud.gif)

Tony Stark is surprised you made it this far.

Vision, mapping, localization, navigation — you’ve crossed almost major subsystem that turns code into autonomy.

There’s only one piece left now. 

# Exploration

Exploration is a process by which a robot operating in an unknown or partially known environment actively searches the space to gather new information. This is a real life use case to use SLAM together with the navigation stack.

There is a simple exploration package that we will use in this lesson, you can download it from the GitHub:
```bash
git clone https://github.com/Jadeninja-23a/exploring.git
```

Copy the package from here and keep it in the src folder of your workspace

Build the workspace and source the `setup.bash` to make sure ROS is aware about the new package!

We'll need 3 terminals, one for the simulation:
```bash
ros2 launch erc_ros2_navigation spawn_robot.launch.py
```

In another terminal we launch the `navigation_with_slam.launch.py`:
```bash
ros2 launch erc_ros2_navigation navigation_with_slam.launch.py
```

And in the third one we launch the exploration:
```bash
ros2 launch explore_lite explore.launch.py
```

The exploration node will identify the boundaries of the know surrounding and will navigate the robot until it finds all the physical boundaries of the environment.
<img width="2560" height="1335" alt="image" src="https://github.com/user-attachments/assets/e2c629bb-f70f-4dc9-8d79-33d5671bf682" />


We can take a look on the `rqt_graph` of the simulation, but we'll see it's quite big! We can find the exploration node and we can see that it subscribes to the `/map` and to the action status and feedback of the navigation stack.
<img width="2250" height="1045" alt="image" src="https://github.com/user-attachments/assets/ae37c93b-1c87-4fa8-a301-0c278d307519" />


And finally here is a video about exploration:

<a href="https://youtu.be/1jlpu-zfNac"><img width="1280" height="719" alt="image" src="https://github.com/user-attachments/assets/65ae2ad4-368a-4fe6-bce7-d0ee4dbd8a3b" /></a>

# Assignment ??

You’ve been working incredibly hard, so there is no formal assignment this week!

![tony-stark-tony-phew](https://github.com/user-attachments/assets/6fc10b50-16dc-4be9-ab3f-bfc3c6ca9ea8) 

`No Assignment`

Instead, we’re doing a cool integration activity: combining Exploration with Chase the Ball. Your goal is to have the bot autonomously search the entire house until it finds the red ball.

## 🔄 Continuous Exploration
Before we integrate the chase logic, we need to fix a limitation in our exploration code. Currently, once the bot finishes mapping the house, it returns to its starting position and stops.

In a real-world scenario, the environment is dynamic—people move, and objects might appear or disappear. To ensure we find the ball, we are modifying `explore.cpp` to run in an infinite loop. This keeps the bot re-exploring the house indefinitely until the target is located.

`explore.cpp` is in the src folder in the explore package

First we add this to `explore.cpp` at the top with the other `#include`s:

```cpp
#include "nav2_msgs/srv/clear_entire_costmap.hpp"
#include "slam_toolbox/srv/reset.hpp"
```

To enable map resetting for re-exploration, the above service definitions are required.

Disabled parameters and functions related to returning to the initial position to prioritize continuous exploration.

```cpp
 double timeout;
  double min_frontier_size;
  this->declare_parameter<float>("planner_frequency", 1.0);
  this->declare_parameter<float>("progress_timeout", 30.0);
  this->declare_parameter<bool>("visualize", false);
  this->declare_parameter<float>("potential_scale", 1e-3);
  this->declare_parameter<float>("orientation_scale", 0.0);
  this->declare_parameter<float>("gain_scale", 1.0);
  this->declare_parameter<float>("min_frontier_size", 0.5);
  // this->declare_parameter<bool>("return_to_init", false);

  
  this->get_parameter("planner_frequency", planner_frequency_);
  this->get_parameter("progress_timeout", timeout);
  this->get_parameter("visualize", visualize_);
  this->get_parameter("potential_scale", potential_scale_);
  this->get_parameter("orientation_scale", orientation_scale_);
  this->get_parameter("gain_scale", gain_scale_);
  this->get_parameter("min_frontier_size", min_frontier_size);
  // this->get_parameter("return_to_init", return_to_init_);
  this->get_parameter("robot_base_frame", robot_base_frame_);
```

```cpp
// void Explore::returnToInitialPose()
// {
//   RCLCPP_INFO(logger_, "Returning to initial pose.");
//   auto goal = nav2_msgs::action::NavigateToPose::Goal();
//   goal.pose.pose.position = initial_pose_.position;
//   goal.pose.pose.orientation = initial_pose_.orientation;
//   goal.pose.header.frame_id = costmap_client_.getGlobalFrameID();
//   goal.pose.header.stamp = this->now();

//   auto send_goal_options =
//       rclcpp_action::Client<nav2_msgs::action::NavigateToPose>::SendGoalOptions();
//   move_base_client_->async_send_goal(goal, send_goal_options);
// }
```
Don't add this code we are just commenting/ removing it the from the `explore.cpp` file.

Now let's add the function that resets the exploration state and reset the map. Add this function after `void Explore::reachedGoal` function .

```cpp
void Explore::resetExplorationState() {
  // costmap_client_.clear();
  prev_goal_ = geometry_msgs::msg::Point();
  progress_timeout_ = this->get_parameter("progress_timeout").as_double();
  last_markers_count_ = 0;
  RCLCPP_INFO(logger_, "Exploration state RESET — starting over!");
  auto global_costmap_client = this->create_client<nav2_msgs::srv::ClearEntireCostmap>("/global_costmap/clear_entirely_global_costmap");
  if (global_costmap_client->wait_for_service(std::chrono::seconds(2))) {
    auto request = std::make_shared<nav2_msgs::srv::ClearEntireCostmap::Request>();
    global_costmap_client->async_send_request(request);
    RCLCPP_INFO(this->get_logger(), "Requested global costmap clear.");
  } else {
    RCLCPP_WARN(this->get_logger(), "Global costmap clear service not available.");
  }

  // --- Clear local costmap too (optional) ---
  auto local_costmap_client = this->create_client<nav2_msgs::srv::ClearEntireCostmap>("/local_costmap/clear_entirely_local_costmap");
  if (local_costmap_client->wait_for_service(std::chrono::seconds(2))) {
    auto request = std::make_shared<nav2_msgs::srv::ClearEntireCostmap::Request>();
    local_costmap_client->async_send_request(request);
    RCLCPP_INFO(this->get_logger(), "Requested local costmap clear.");
  } else {
    RCLCPP_WARN(this->get_logger(), "Local costmap clear service not available.");
  }

  // --- Reset SLAM map ---
  auto slam_reset_client = this->create_client<slam_toolbox::srv::Reset>("/slam_toolbox/reset");
  if (slam_reset_client->wait_for_service(std::chrono::seconds(2))) {
    auto request = std::make_shared<slam_toolbox::srv::Reset::Request>();
    slam_reset_client->async_send_request(request);
    RCLCPP_INFO(this->get_logger(), "Requested SLAM map reset.");
  } else {
    RCLCPP_WARN(this->get_logger(), "SLAM reset service not available.");
  }
  
  RCLCPP_INFO(logger_, "Waiting for sensors map update...");
  // Instead of plain resume:
  spinGoal();

}
```

When we delete the map, we also lose the current sensor data. This can make the robot confused because it suddenly sees an empty world. To fix this, the spinGoal function makes the robot perform a 180-degree turn in place.

This rotation allows the sensors (like LIDAR) to scan the surroundings and update the map immediately so exploration can continue smoothly.

Add this after the previous function.

```cpp
void Explore::spinGoal()
{
  RCLCPP_INFO(logger_, "Spinning in place to refresh sensors.");

  auto goal = nav2_msgs::action::NavigateToPose::Goal();
  goal.pose.header.frame_id = costmap_client_.getGlobalFrameID();
  goal.pose.header.stamp = this->now();

  // Keep same position
  auto pose = costmap_client_.getRobotPose();
  goal.pose.pose.position = pose.position;

  // Just change orientation: 180 deg spin for example
  tf2::Quaternion q_orig, q_rot, q_new;
  tf2::fromMsg(pose.orientation, q_orig);
  q_rot.setRPY(0, 0, M_PI);  // Rotate 180 deg
  q_new = q_rot * q_orig;
  q_new.normalize();
  goal.pose.pose.orientation = tf2::toMsg(q_new);

  auto send_goal_options = rclcpp_action::Client<nav2_msgs::action::NavigateToPose>::SendGoalOptions();
  send_goal_options.result_callback = [this](const NavigationGoalHandle::WrappedResult& result) {
    if (result.code == rclcpp_action::ResultCode::SUCCEEDED) {
      RCLCPP_INFO(logger_, "Spin goal succeeded, resuming exploration.");
    } else {
      RCLCPP_WARN(logger_, "Spin goal failed or canceled, resuming anyway.");
    }
    // After spin, continue normal exploration
    resume();
  };

  move_base_client_->async_send_goal(goal, send_goal_options);
}
```

We added 2 new functions in `explore.cpp` so let's add them in `explore.h` (located in include folder):

```cpp
  geometry_msgs::msg::Pose initial_pose_;
  //void returnToInitialPose(void);
  void resetExplorationState();
  void spinGoal();
```

We also removed the `returnToInitialPose` function.

Remember we added some random includes at the starting of `explore.cpp` now we need to tell the package where to find those files are 
therefore make the following changes in the following files.

In `CMakeLists.txt` add slam_toolbox and nav2_msgs:

```txt
# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(tf2 REQUIRED)
find_package(tf2_geometry_msgs REQUIRED)
find_package(nav2_msgs REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(map_msgs REQUIRED)
find_package(visualization_msgs REQUIRED)
find_package(nav2_costmap_2d REQUIRED)
find_package(slam_toolbox REQUIRED)

set(DEPENDENCIES
  rclcpp
  std_msgs
  sensor_msgs
  tf2
  tf2_ros
  tf2_geometry_msgs
  nav2_msgs
  nav_msgs
  map_msgs
  nav2_costmap_2d
  visualization_msgs
  slam_toolbox
)
```

In `package.xml` make sure these dependencies are added:

```xml
  <depend>map_msgs</depend>
  <depend>nav2_costmap_2d</depend>
  <depend>nav2_msgs</depend>
  <depend>nav_msgs</depend>
  <depend>rclcpp</depend>
  <depend>sensor_msgs</depend>
  <depend>std_msgs</depend>
  <depend>tf2</depend>
  <depend>tf2_geometry_msgs</depend>
  <depend>tf2_ros</depend>
  <depend>visualization_msgs</depend>
  <depend>slam_toolbox</depend> 
```
Even though most probably the libraries are installed there is a chance for them to be missing so just for safety run the following 2 commands:

```bash
sudo apt install ros-jazzy-nav2-msgs
sudo apt install ros-jazzy-slam-toolbox
```

Now the map should automatically reset after it's fully explored build the workspace and run the same commands you did for exploration.

## Controller_Node

To make our life simple let's add a controller node which checks from `chase_the_ball.py` wether a ball has been found or not and depending on that give msg to explore.cpp wether to continue exploring or stop exploring and run the chase the ball code.

In erc_ros2_navigation_py package in the erc_ros2_navigation_py folder make file called `controller_node.py` and copy paste the following code. 

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import Bool
import time

class ExploreController(Node):
    def __init__(self):
        super().__init__('explore_controller')
        self.sub = self.create_subscription(Bool, '/object_detected', self.detected_callback, 10)
        self.pub = self.create_publisher(Bool, '/explore/resume', 10)
        
        self.current_state = None  # Track current state to avoid republishing
        self.last_publish_time = 0
        self.publish_cooldown = 1.0  # 1 second cooldown between state changes

    def detected_callback(self, msg):
        current_time = time.time()
        
        # Avoid rapid state changes
        if current_time - self.last_publish_time < self.publish_cooldown:
            return
        
        # Only publish if state actually changed
        if msg.data == self.current_state:
            return
        
        control_msg = Bool()
        
        if msg.data:
            # Object detected - STOP exploration
            self.get_logger().info("🔴 Object detected! Stopping exploration...")
            control_msg.data = False  # False = stop exploring
            self.pub.publish(control_msg)
            self.current_state = True
            
        else:
            # Object lost - RESUME exploration
            self.get_logger().info("🟢 Object lost! Resuming exploration...")
            control_msg.data = True  # True = resume exploring
            self.pub.publish(control_msg)
            self.current_state = False
        
        self.last_publish_time = current_time


def main(args=None):
    rclpy.init(args=args)
    node = ExploreController()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        rclpy.shutdown()

if __name__ == '__main__':
    main()
```

Add an entry point in `setup.py`:

```python
'console_scripts': [
            'send_initialpose = erc_ros2_navigation_py.send_initialpose:main',
            'slam_toolbox_load_map = erc_ros2_navigation_py.slam_toolbox_load_map:main',
            'follow_waypoints = erc_ros2_navigation_py.follow_waypoints:main',
            'controller_node = erc_ros2_navigation_py.controller_node:main', 
        ],
```

Include the following dependencies in `package.xml`

```xml
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
  <name>erc_ros2_navigation_py</name>
  <version>1.0.0</version>
  <description>Python nodes for slam, localization and navigation with Gazebo Harmonic and ROS Jazzy for BME MOGI ROS2 course</description>
  <maintainer email="sagarv812@gmail.com">ERC IITB</maintainer>
  <license>Apache License 2.0</license>

  <!-- changed -->

  <exec_depend>rclpy</exec_depend>
  <exec_depend>std_msgs</exec_depend>
  <exec_depend>launch</exec_depend>
  <exec_depend>launch_ros</exec_depend>

  <test_depend>ament_copyright</test_depend>
  <test_depend>ament_flake8</test_depend>
  <test_depend>ament_pep257</test_depend>
  <test_depend>python3-pytest</test_depend>

  <export>
    <build_type>ament_python</build_type>
  </export>
</package>

```

That's well and good but no point of this node if it's not getting information in the `/object_detected` topic from  `chase_the_ball.py`.
Let's change `chase_the_ball.py` (It was in `erc_gazebo_sensors_py` package)

Add the following variables in `__init__` function:

```python
self.object_detected_pub = self.create_publisher(Bool, "/object_detected", 10)
self.ball_detected = False
self.stopped_near_ball = False        
self.area_threshold = 0.25
```

And replace the `process_image` function with the following function: 
```python
def process_image(self, img):
        """Image processing task."""
        msg = Twist()
        msg.linear.x = 0.0
        msg.linear.y = 0.0
        msg.linear.z = 0.0
        msg.angular.x = 0.0
        msg.angular.y = 0.0
        msg.angular.z = 0.0

        rows, cols = img.shape[:2]
        total_area = rows * cols

        
        R, G, B = self.convert2rgb(img)

        redMask = self.threshold_binary(R, (220, 255))
        stackedMask = np.dstack((redMask, redMask, redMask))
        contourMask = stackedMask.copy()
        crosshairMask = stackedMask.copy()

        # return value of findContours depends on OpenCV version
        (contours, hierarchy) = cv2.findContours(
            redMask.copy(), 1, cv2.CHAIN_APPROX_NONE
        )
        
        detection_msg = Bool()

        # Find the biggest contour (if detected)
        if len(contours) > 0:

            c = max(contours, key=cv2.contourArea)
            contour_area = cv2.contourArea(c)
            area_percentage = contour_area / total_area
            
            M = cv2.moments(c)

            # Make sure that "m00" won't cause ZeroDivisionError: float division by zero
            if M["m00"] != 0:
                cx = int(M["m10"] / M["m00"])
                cy = int(M["m01"] / M["m00"])
            else:
                cx, cy = 0, 0

            # Show contour and centroid
            cv2.drawContours(contourMask, contours, -1, (0, 255, 0), 10)
            cv2.circle(contourMask, (cx, cy), 5, (0, 255, 0), -1)

            # Show crosshair and difference from middle point
            cv2.line(crosshairMask, (cx, 0), (cx, rows), (0, 0, 255), 10)
            cv2.line(crosshairMask, (0, cy), (cols, cy), (0, 0, 255), 10)
            cv2.line(
                crosshairMask,
                (int(cols / 2), 0),
                (int(cols / 2), rows),
                (255, 0, 0),
                10,
            )
            
            # Check if ball is close enough (based on area)
            if area_percentage >= self.area_threshold:
                # Ball is close enough - STOP
                msg.linear.x = 0.0
                msg.angular.z = 0.0
                
                if not self.stopped_near_ball:
                    self.get_logger().info(f"Ball detected and close enough! Area: {area_percentage:.2%} - STOPPING")
                    self.stopped_near_ball = True
                    self.ball_detected = True
                    
                    # Publish object detected = True (state changed)
                    detection_msg.data = True
                    self.object_detected_pub.publish(detection_msg)
                
            else:
                # Ball detected but not close enough - Chase it
                if self.stopped_near_ball:
                    # State change: was stopped, now chasing again
                    self.get_logger().info(f"Ball moved away, chasing again... Area: {area_percentage:.2%}")
                    detection_msg.data = True
                    self.object_detected_pub.publish(detection_msg)
                
                self.stopped_near_ball = False
                
                if not self.ball_detected:
                    # State change: ball just appeared
                    self.get_logger().info(f"Ball detected! Starting chase... Area: {area_percentage:.2%}")
                    detection_msg.data = True
                    self.object_detected_pub.publish(detection_msg)
                    self.ball_detected = True
                
                # Chase the ball
                if abs(cols / 2 - cx) > 20:
                    msg.linear.x = 0.0
                    if cols / 2 > cx:
                        msg.angular.z = 0.2
                    else:
                        msg.angular.z = -0.2
                else:
                    msg.linear.x = 0.2
                    msg.angular.z = 0.0

        else:
            # No ball detected
            
            if self.ball_detected or self.stopped_near_ball:
                self.get_logger().info("Ball lost! Resuming exploration...")
                self.ball_detected = False
                self.stopped_near_ball = False
                
                # Publish object detected = False (ball lost - state changed)
                detection_msg.data = False
                self.object_detected_pub.publish(detection_msg)

        # Publish cmd_vel
        self.publisher.publish(msg)

        # Return processed frames
        return redMask, contourMask, crosshairMask

```

Now finally we can run and test it!!

First of all don't forget to build and source the workspace .

We need to open 5 terminals (and source in all terminals don't forget) and write the following commands in each terminal.

Terminal 1:
```bash
ros2 launch erc_ros2_navigation spawn_robot.launch.py
```

Terminal 2:
```bash
ros2 launch erc_ros2_navigation navigation_with_slam.launch.py
```

Terminal 3:
```bash
ros2 launch explore_lite explore.launch.py
```

Terminal 4:
```bash
ros2 run erc_ros2_navigation_py controller_node
```

Terminal 5:
```bash
ros2 launch erc_gazebo_sensors_py chase_the_ball
```

And voila!! It should be working use the resource spawner to spawn the ball and see how our bot finds the ball.
You can also remove the ball to see how the bot continues to explore if the ball is lost. Experiment around and enjoy :)

<img width="800" height="348" alt="image" src="https://github.com/user-attachments/assets/6a01084a-63af-4f60-b8ff-6134daf51700" />(Ironmancool.gif)

Suit up for the Final Week!
