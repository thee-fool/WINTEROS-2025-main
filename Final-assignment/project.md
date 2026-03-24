# Final Assignment 

You’ve built the arm. You’ve given the system perception. Now it’s time to make StarkOS connect the dots. In this assignment, the robotic arm and the ball stop existing as separate problems and start operating as one integrated system. Ball data flows through ROS, drives control decisions, and directly shapes motion. No scripts. No isolation. Just perception, coordination, and execution fused into a single, responsive machine — exactly how Stark would build it.

Let's take our ending file after week3 as a starting code and we need to make modifications to that file 

First of all picking up a red ball is hard, so let's make red cylinder 
go to /home/ubuntu/gazebo_models, or wherever you saved the Gazebo Model in Week 2

There will be a folder called red_ball_10in.

Duplicate the folder and name it red_cylinder 

in model.config 
change this line from ball to cylinder
`
name>red_cylinder</name>
`

in model.sdf replace entire code with this block
```xml
<?xml version="1.0" ?>
<sdf version="1.5">
  <model name="red_cylinder">
    <pose>0 0 0.1 0 0 0</pose>
    <link name="link">
      <inertial>
        <mass>0.150</mass>
        <inertia>
          <ixx>0.00056</ixx>
          <ixy>0</ixy>
          <ixz>0</ixz>
          <iyy>0.00056</iyy>
          <iyz>0</iyz>
          <izz>0.00012</izz>
        </inertia>
      </inertial>
      
      <visual name="visual">
        <geometry>
          <cylinder>
            <radius>0.04</radius>
            <length>0.3</length>
          </cylinder>
        </geometry>
        <material>
          <script>
            <uri>file://media/materials/scripts/gazebo.material</uri>
            <name>Gazebo/Red</name>
          </script>
          <emissive>1 0 0 1</emissive>
        </material>
      </visual>
      
      <collision name="collision">
        <geometry>
          <cylinder>
            <radius>0.04</radius>
            <length>0.2</length>
          </cylinder>
        </geometry>
        <surface>
          <friction>
            <ode>
              <mu>1.0</mu>
              <mu2>1.0</mu2>
            </ode>
          </friction>
          <contact>
            <ode>
              <kp>1000000.0</kp>
              <kd>1.0</kd>
              <max_vel>0.1</max_vel>
              <min_depth>0.001</min_depth>
            </ode>
          </contact>
        </surface>
      </collision>
    </link>
  </model>
</sdf>
```

Now as we were gonna start with the Week 3 ending code workspace let's add erc_simple_arm and erc_simple_arm_py packages to the Week 3 workspace

Copy paste the mogi_arm.gazebo file 

From erc_simple_arm/urdf folder to erc_ros2_navigation/urdf folder because that's going to be our main bot which we are spawning with 

also copy paste all the meshes in erc_simple_arm/meshes folder to erc_ros2_navigation folder 

First we need to add the arm on the original bot we already have. We will be placing it over the lidar as otherwise the arm will block the lidar's view 


```xml
<joint name="arm_base_joint" type="fixed">
        <origin xyz="0.0 0 0.15" rpy="0 0 0"/>
        <parent link="base_link"/>
        <child link="arm_base_link"/>
    </joint>

    <link name="arm_base_link">
        <inertial>
        <mass value="2"/>
        <origin xyz="0.0 0.0 0.0"/>
        <inertia ixx="0.0117" ixy="0.0" ixz="0.0"
                iyy="0.0117" iyz="0.0"
                izz="0.0225"/>
        </inertial>
        <collision>
        <geometry>
            <cylinder radius="0.15" length="0.05"/>
        </geometry>
        <origin xyz="0 0 0"/>
        </collision>
        <visual>
        <geometry>
            <cylinder radius="0.15" length="0.05"/>
        </geometry>
        <material name="grey"/>
        <origin xyz="0 0 0"/>
        </visual>
    </link>
```
Add this code at the end of erc_bot.urdf file

now copy paste mogi_arm.xacro from erc_ros2_simple_arm from 
the shoulder 

`
<!-- STEP 4 - Shoulder -->
`

to 

```xml
<!-- End effector link -->
<link name="end_effector_link">
    <visual>
      <origin xyz="0 0 0" rpy="0 0 0"/>
      <geometry>
        <box size="0.01 0.01 0.01" />
      </geometry>
      <material name="red"/>
     </visual>

    <inertial>
      <origin xyz="0 0 0" />
      <mass value="1.0e-03" />
      <inertia ixx="1.0e-03" ixy="0.0" ixz="0.0"
               iyy="1.0e-03" iyz="0.0"
               izz="1.0e-03" />
    </inertial>
  </link>
```

There will be some changes that need to be made 
1) Meshes file path
2) The base_link for this part of the code should not be the same as base_link for the whole bot so wherever there is base_link in this part of the code it should be replaced with something guess what?

now go to spawn_robot.launch.py 

add the following nodes 

```python

robot_controllers = PathJoinSubstitution(
        [
            get_package_share_directory('erc_ros2_simple_arm'),
            'config',
            'controller_position.yaml',
        ]
    )
    
    joint_trajectory_controller_spawner = Node(
        package='controller_manager',
        executable='spawner',
        arguments=[
            'arm_controller',
            'gripper_controller',
            '--param-file',
            robot_controllers,
            ],
        parameters=[
            {'use_sim_time': LaunchConfiguration('use_sim_time')},
        ]
    )
    joint_state_broadcaster_spawner = Node(
        package='controller_manager',
        executable='spawner',
        arguments=['joint_state_broadcaster'],
        parameters=[
            {'use_sim_time': LaunchConfiguration('use_sim_time')},
        ]
    )
```

dont forget to add launchdescriptionobject at the end

To calculate coordinates of object we will be using rgbd camera instead of normal camera so go to erc_bot.gazebo 
and change the camera code to this 

```xml
<gazebo reference="camera_link">
  <sensor name="camera" type="rgbd_camera">
    <camera>
      <horizontal_fov>1.25</horizontal_fov>
      <image>
        <width>640</width>
        <height>480</height>
      </image>
      <clip>
        <near>0.1</near>
        <far>15</far>
      </clip>
      <optical_frame_id>camera_link_optical</optical_frame_id>
      <camera_info_topic>camera/rgb/camera_info</camera_info_topic>
    </camera>

    <depth_camera>
      <image>
        <width>640</width>
        <height>480</height>
      </image>
      <clip>
        <near>0.1</near>
        <far>15</far>
      </clip>
      <camera_info_topic>camera/depth/camera_info</camera_info_topic>
    </depth_camera>

    <always_on>1</always_on>
    <update_rate>20</update_rate>
    <visualize>true</visualize>

    <topic>camera</topic> <!-- Base topic, Gazebo will create subtopics like camera/rgb/image_raw and camera/depth/image_raw -->
    <gz_frame_id>camera_link</gz_frame_id>
  </sensor>
</gazebo>
```

Hmm rgb image travels through camera/image but the depth_image travels through camera/depth_image this needs to be added in the gazebo to ROS bridge so in 
gz_bridge.yaml

```yaml
- ros_topic_name: "camera/depth_image"
  gz_topic_name: "camera/depth_image"
  ros_type_name: "sensor_msgs/msg/Image"
  gz_type_name: "gz.msgs.Image"
  direction: "GZ_TO_ROS"
```

Add the following at the end of the file (be careful about formatting otherwise ur bridge will crash)

Okay now changes in chase_the_ball.py 

```python
self.found_object_pb = self.create_publisher(
    Bool,
    "/arm_controller/found_object",
    10
)

self.object_picked_sub = self.create_subscription(
    Bool,
    "/arm_controller/picked_object",
    self.picked_callback,
    10
)
```

In chase_the_ball.py 
`self.area_threshold` variable should be changed  according to the shape of the new cylinder change this according to how close you want it to be to the cylinder 
added new variable 
```python
self.picking_ball = False
```
This variable is to denote that the cylinder is being picked up as if the cylinder goes outside camera view the bot continues exploration but we don't want that to happen as otherwise it would not be able pick up the cylinder 

```python
def picked_callback(self, msg):
        if(msg.data == True):
```
We get this message from the arm_controller (haven't made file yet, we will make it a little later) 
when we finish picking up the cylinder 
think about  all the things that should be changed in the code 
Remember we want the bot to continue exploring after it picked up the cylinder.
Is the bot picking the cylinder ? Is it stopping near the ball ? etc are things you should consider.

Think about what to keep in this if statement

```pythonif area_percentage >= self.area_threshold:
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
```

This is the old code of chase_the_ball.py
Try to making the following changes 
use the variable self.picking_ball 
update the variable when you are going to pick up the cylinder 
also send a message in the 
self.found_object_pb so the arm_controller knows when to start picking up the cylinder 
also think about how to use self.picking_ball to change the if statements so that the bot doesn't move when arm is picking up cylinder

Finally now we need to make the logic for picking up the cylinder 
make a file called `integrated_pickup.py` in erc_ros2_simple_arm_py which takes topic information from chase_the_ball.py and picksup the cylinder 

This file has some blanks, you need to fill those. 

```python
#whole file is a change 
import rclpy
import math
import numpy as np
from rclpy.node import Node
from trajectory_msgs.msg import JointTrajectory, JointTrajectoryPoint
from std_msgs.msg import Bool
from sensor_msgs.msg import Image, CameraInfo
from cv_bridge import CvBridge
import cv2
from tf2_ros import TransformListener, Buffer
from tf2_geometry_msgs import do_transform_point
from geometry_msgs.msg import PointStamped


class BallPickupController(Node):
    def __init__(self):
        super().__init__('ball_pickup_controller')
        
        # Publisher for joint trajectories
        self.trajectory_pub = self.create_publisher(
            JointTrajectory, 
            '/arm_controller/joint_trajectory', 
            10
        )
        
        # Publisher for picked object confirmation
        self.picked_pub = self.create_publisher(
            Bool,
            '/arm_controller/picked_object',
            10
        )
        
        # Subscriber for object detection trigger
        self.detection_sub = self.create_subscription(
            Bool,
            '/arm_controller/found_object',
            self.detection_callback,
            10
        )
        
        # Subscribers for depth camera
        self.depth_sub = self.create_subscription(
            Image,
            '/camera/depth_image',
            self.depth_callback,
            10
        )
        
        self.rgb_sub = self.create_subscription(
            Image,
            '/camera/image',
            self.rgb_callback,
            10
        )
        
        self.camera_info_sub = self.create_subscription(
            CameraInfo,
            '/camera/camera_info',
            self.camera_info_callback,
            10
        )
        
        # TF2 for coordinate transformation
        self.tf_buffer = Buffer()
        self.tf_listener = TransformListener(self.tf_buffer, self)
        
        # CV Bridge for image conversion
        self.bridge = CvBridge()
        
        # State variables
        self.depth_image = None
        self.rgb_image = None
        self.camera_info = None
        self.pickup_in_progress = False
        
        self.get_logger().info('Ball Pickup Controller initialized')
    
    def camera_info_callback(self, msg):
        """Store camera intrinsic parameters"""
        self.camera_info = msg
    
    def depth_callback(self, msg):
        """Store latest depth image"""
        try:
            self.depth_image = self.bridge.imgmsg_to_cv2(msg, desired_encoding='passthrough')
        except Exception as e:
            self.get_logger().error(f'Error converting depth image: {e}')
    
    def rgb_callback(self, msg):
        """Store latest RGB image"""
        try:
            self.rgb_image = self.bridge.imgmsg_to_cv2(msg, desired_encoding='bgr8')
        except Exception as e:
            self.get_logger().error(f'Error converting RGB image: {e}')
    
    def detection_callback(self, msg):
        """Triggered when object is detected"""
        if msg.data and not self.pickup_in_progress:
            self.get_logger().info('Object detected! Starting pickup sequence...')
            # TODO:
            # < Need to change one variable what is that >
            self.execute_pickup()
            # < In case the picking up fails and comes outside we want to tell the process we finished our work 
            # (unsuccesfully) (doesn't matter succesfuly or unsuccesfully just msg should be same I finished my work)>
            # END TODO
            
            
    def find_ball_centroid(self):
        """
        Detect red ball in RGB image and get its 3D coordinates
        Returns: [x, y, z] in camera frame or None if not found
        """
        if self.rgb_image is None or self.depth_image is None or self.camera_info is None:
            self.get_logger().warn('Missing camera data')
            return None
        
        # Convert to HSV for red color detection
        hsv = cv2.cvtColor(self.rgb_image, cv2.COLOR_BGR2HSV)
        
        # Red color range (red wraps around in HSV, so we need two ranges)
        lower_red1 = np.array([0, 100, 100])
        upper_red1 = np.array([10, 255, 255])
        lower_red2 = np.array([160, 100, 100])
        upper_red2 = np.array([180, 255, 255])
        
        mask1 = cv2.inRange(hsv, lower_red1, upper_red1)
        mask2 = cv2.inRange(hsv, lower_red2, upper_red2)
        mask = cv2.bitwise_or(mask1, mask2)
        
        # Find contours
        contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
        
        if not contours:
            self.get_logger().warn('No red ball detected in image')
            return None
        
        # Get largest contour (assumed to be the ball)
        largest_contour = max(contours, key=cv2.contourArea)
        
        # Calculate centroid
        M = cv2.moments(largest_contour)
        if M['m00'] == 0:
            return None
        
        cx = int(M['m10'] / M['m00'])
        cy = int(M['m01'] / M['m00'])
        
        self.get_logger().info(f'Ball centroid in image: ({cx}, {cy})')
        
        # Get depth at centroid
        
        self.get_logger().info(f'RAW DEPTH: {self.depth_image[cy, cx]}')
        
        depth = self.depth_image[cy, cx]
        
        if depth == 0 or np.isnan(depth):
            self.get_logger().warn('Invalid depth value')
            return None
        
        # Convert pixel coordinates to 3D camera coordinates
        fx = self.camera_info.k[0]
        fy = self.camera_info.k[4]
        cx_cam = self.camera_info.k[2]
        cy_cam = self.camera_info.k[5]
        
        # Camera frame coordinates
        x_cam = (cx - cx_cam) * depth / fx
        y_cam = (cy - cy_cam) * depth / fy
        z_cam = depth
        
        self.get_logger().info(f'Ball position in camera frame: ({x_cam:.3f}, {y_cam:.3f}, {z_cam:.3f})')
        
        return [x_cam, y_cam, z_cam]
    
    def transform_to_base(self, point_camera):
        try:
            # 1. Create the PointStamped
            point_stamped = PointStamped()
            point_stamped.header.frame_id = 'camera_link_optical'
            point_stamped.header.stamp = self.get_clock().now().to_msg()
            point_stamped.point.x = point_camera[0]
            point_stamped.point.y = point_camera[1]
            point_stamped.point.z = point_camera[2]
            
            # 2. Lookup the transform
            transform = self.tf_buffer.lookup_transform(
                'arm_base_link',
                'camera_link_optical',
                rclpy.time.Time(),
                timeout=rclpy.duration.Duration(seconds=1.0)
            )
            
            # 3. USE THE LIBRARY TO TRANSFORM (This handles the rotation!)
            point_base = do_transform_point(point_stamped, transform)
            
            # 4. Extract the results
            x_base = point_base.point.x
            y_base = point_base.point.y
            z_base = point_base.point.z
            
            self.get_logger().info(f'Ball position in base frame: ({x_base:.3f}, {y_base:.3f}, {z_base:.3f})')
            
            return [x_base, y_base, z_base]
            
        except Exception as e:
            self.get_logger().error(f'Transform failed: {e}')
            return None
    
    def execute_pickup(self):
        """Execute the complete pickup sequence"""
        # Find ball in camera frame
        ball_camera = self.find_ball_centroid()
        import time
        if ball_camera is None:
            self.get_logger().error('Failed to detect ball')
            self.pickup_in_progress = False
            return
        
        # Transform to base frame
        ball_base = self.transform_to_base(ball_camera)
        
        if ball_base is None:
            self.get_logger().error('Failed to transform coordinates')
            self.pickup_in_progress = False
            return
        
        x, y, z = ball_base
        
        #TODO:
        
        # <x y z are the cylinder cordinates now use those coordinates experiment around and make sure the arm is picking up the cylinder >
        # <If any motion is happening to fast remember to change duration>
    
        # Execute pickup sequence with hardcoded offsets
        # Step 1: Approach from top
        # Stay high enough to avoid hitting the ball while moving horizontally
        self.get_logger().info('Step 1: Approach from top')
        self.publish_trajectory(x, y, z , "open", 1)

        # Step 2: Move down to ball
        # We want the gripper 'palm' to be slightly above the ball center
        # If z is the center, z + 0.02 keeps the gripper from smashing into the ball
        # The x offset depends on your gripper length; -0.01 keeps it centered
        self.get_logger().info('Step 2: Move down to ball')
        self.publish_trajectory(0.2, 0.0, 0.4, "open", 1)
        time.sleep(1.5)
        # Step 3: Close gripper
        # Same position as Step 2, just close the fingers
        self.get_logger().info('Step 3: Close gripper')
        self.publish_trajectory(x, y, z , "closed", 1)
        time.sleep(0.5)
        # Step 4: Lift up
        # Standard lift to clear any nearby obstacles
        self.get_logger().info('Step 4: Lift up')
        self.publish_trajectory(0.0, 0.0, 1.0, "open", 1)

        # Step 5: Move back (Home/Drop position)
        # Move it closer to the robot base and high up
        self.get_logger().info('Step 5: Move back')
        self.publish_trajectory(0.2, 0.0, 0.4, "closed", 0.5)
        
        
        
        # Publish completion message
        # <We finished picking up the ball succesfully so publish message (guess which topic )>
        # <Again need to change one variable what was it ???>
        #END TODO
        
        self.get_logger().info('Pickup sequence complete!')
        
    
    def publish_trajectory(self, x, y, z, gripper_status, duration):
        """Publish a single trajectory point and wait for completion"""
        import time
        
        trajectory = JointTrajectory()
        trajectory.joint_names = [
            'shoulder_pan_joint', 
            'shoulder_lift_joint', 
            'elbow_joint', 
            'wrist_joint', 
            'left_finger_joint', 
            'right_finger_joint'
        ]
        #TODO:
        
        #<Check inverse_kinematics.py from week 4 check how we defined points and what would be the equivalent code here>
        
        # END TODO
        trajectory.points = [point]
        
        self.trajectory_pub.publish(trajectory)
        self.get_logger().info(f'Published trajectory to ({x:.3f}, {y:.3f}, {z:.3f}), gripper: {gripper_status}')
        
        # Wait for the trajectory to complete (duration + small buffer)
        time.sleep(duration + 0.5)
        self.get_logger().info(f'Trajectory completed, waited {duration + 0.5}s')
    
    def inverse_kinematics(self, coords, gripper_status, gripper_angle = 0):
        '''
        Calculates the joint angles according to the desired TCP coordinate and gripper angle
        :param coords: list, desired [X, Y, Z] TCP coordinates
        :param gripper_status: string, can be `closed` or `open`
        :param gripper_angle: float, gripper angle in woorld coordinate system (0 = horizontal, pi/2 = vertical)
        :return: list, the list of joint angles, including the 2 gripper fingers
        '''
        # link lengths
        ua_link = 0.2
        fa_link = 0.25
        tcp_link = 0.175
        # z offset (robot arm base height)
        z_offset = 0.1
        # default return list
        angles = [0,0,0,0,0,0]

        # Calculate the shoulder pan angle from x and y coordinates
        j0 = math.atan(coords[1]/coords[0])

        # Re-calculate target coordinated to the wrist joint (x', y', z')
        x = coords[0] - tcp_link * math.cos(j0) * math.cos(gripper_angle)
        y = coords[1] - tcp_link * math.sin(j0) * math.cos(gripper_angle)
        z = coords[2] - z_offset + math.sin(gripper_angle) * tcp_link

        # Solve the problem in 2D using x" and z'
        x = math.sqrt(y*y + x*x)

        # Let's calculate auxiliary lengths and angles
        c = math.sqrt(x*x + z*z)
        alpha = math.asin(z/c)
        beta = math.pi - alpha
        # Apply law of cosines
        gamma = math.acos((ua_link*ua_link + c*c - fa_link*fa_link)/(2*c*ua_link))

        j1 = math.pi/2.0 - alpha - gamma
        j2 = math.pi - math.acos((ua_link*ua_link + fa_link*fa_link - c*c)/(2*ua_link*fa_link)) # j2 = 180 - j2'
        delta = math.pi - (math.pi - j2) - gamma # delta = 180 - j2' - gamma

        j3 = math.pi + gripper_angle - beta - delta

        angles[0] = j0
        angles[1] = j1
        angles[2] = j2
        angles[3] = j3

        if gripper_status == "open":
            angles[4] = 0.04
            angles[5] = 0.04
        elif gripper_status == "closed":
            angles[4] = 0.01
            angles[5] = 0.01
        else:
            angles[4] = 0.04
            angles[5] = 0.04

        return angles


def main(args=None):
    rclpy.init(args=args)
    node = BallPickupController()
    
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

Don't forget to add the python file in setup.py of ros2_simple_arm_py 
For execution run all the 5 terminal commands from week 3 and also 

```bash
ros2 run ros2_simple_arm_py integrated_pickup
```

<img src="iron-man-tony-stark.gif" width="800">  

It's better to run this command before the chase_the_ball command as otherwise if chase_the_ball.py sends found_object before integrated pickup node is set up then integrated_pickup never gets the topic message

This is the final build. The last time you open the workshop doors and power everything on with the intent to make it all work together. Every concept you’ve touched — perception, kinematics, control, ROS architecture, timing, failure handling — converges here. This assignment isn’t about proving that individual pieces function. It’s about proving you can integrate them into a coherent machine. When it works, it won’t feel like finishing homework. It will feel like closing the arc — knowing that what you built can sense the world, reason about it, and act with purpose. This is the moment you stop being a student of robotics and start thinking like a systems engineer.

BEST OF LUCK
