# Gazebo Tutorial (Model Building + Simple Plugins)
## Contents
### Intro to Gazebo GUI
- [Gazebo GUI](http://gazebosim.org/tutorials?cat=guided_b&tut=guided_b2)
- [Model Editor](http://gazebosim.org/tutorials?cat=guided_b&tut=guided_b3)
- [Building Editor](http://gazebosim.org/tutorials?cat=guided_b&tut=guided_b3)
### Building Robot Model

- [Building a robot model through URDF](#building-a-robot-model-through-urdf)
#### Building a robot model through URDF (Universal Robotic Description Format)
![Components](http://wiki.ros.org/urdf?action=AttachFile&do=get&target=urdf_diagram.png)
##### 1. What is URDF?
Universal Robotic Description Format (URDF) is an XML file format used in ROS to describe all elements of a robot. To use a URDF file in Gazebo
##### 2. Why do we start with using URDF?
- It's a useful and standardized format in ROS to describe all elements of a robot.
- It can be directly use to publish TF or visualize in RVIZ.
- It has tools like **xacro** to handle the math and simplify the writing of the urdf.
- There are tools to convert URDF to SDF or Collada, SolidWorks files to URDF, etc.
##### 3. Why not using URDF directly in simulation?
- URDF can be used for specifying only the kinematic and dynamic properties of a single robot in isolation. 
- URDF can not specify the pose of the robot itself within a world. It is also not a "universal" description format since it cannot specify joint loops, and it lacks friction and other properties. Additionally, it cannot specify things that are not robots, such as lights, heightmaps, etc.
- On an implementation side, the URDF syntax breaks proper formatting with heavy attributes which in turns makes URDF more inflexible. There is also no mechanism for backward compatibility.

**SDF** solves all these problems. It is a complete description for everything from the world level down to the robot level. It is highly scalable, and extremely easy to add and modify elements. The SDF format is itself described using XML, which facilitates a simple upgrade tool to migrate old versions to new versions. It is also self descriptive.

####   Task 1: Box (Link)
Documentation: http://wiki.ros.org/urdf/XML/link

Code for the box:
```xml
<?xml version="1.0"?>
<robot name="box">
  <link name="base_link">
    <!-- Visual Components -->
    <visual name="base_link_visual">
      <geometry>
        <box size="3.0 2.0 0.4"/>
      </geometry>
    </visual>
    <!-- Collision Components -->
    <collision name="base_link_collision">
      <geometry>
        <box size="3.0 2.0 0.4"/>
      </geometry>
    </collision>
  </link>
</robot>
```
To visualize the box, run:
```bash
roslaunch omni_description display.launch model:=$(find omni_description)/urdf/box.urdf
```

#### Task 2: Lidar and Box (Joint)
Documentation: http://wiki.ros.org/urdf/XML/joint.

Code for task 2:
```xml
<?xml version="1.0" ?>
<robot name="robot">
  <link name="base_link">
    <visual name="base_link_visual">
      <geometry>
        <box size="3.0 2.0 0.4"/>
      </geometry>
      <material name="orange">
          <color rgba="1 0.5 0 1"/>
      </material>
    </visual>
    <collision name="base_link_collision">
      <geometry>
        <box size="3.0 2.0 0.4"/>
      </geometry>
    </collision>
  </link>
    
  <!-- Using mesh file as the visual components -->
  <link name="base_laser_link">
    <visual name="base_laser_link_visual">
      <origin xyz="0 0 0.0115"/>
      <geometry>
        <mesh filename="package://omni_description/meshes/hokuyo_utm_30lx.dae"/>
      </geometry>
    </visual>
    <collision name="base_laser_link_collision">
      <origin xyz="0 0 0.0115"/>
      <geometry>
        <box size="0.06 0.06 0.087"/>
      </geometry>
    </collision>
  </link>
    
  <!-- By default the Joint is on the same position -->
  <joint name="base_to_laser" type="fixed">
    <parent link="base_link"/>
    <child link="base_laser_link"/>
    <origin xyz="0 0 0.241"/>
  </joint>
</robot>
```

#### Task 3: URDF Dynamic Properties
Info: https://en.wikipedia.org/wiki/List_of_moments_of_inertia

A brief Explanation: http://farside.ph.utexas.edu/teaching/336k/Newtonhtml/node64.html

Code 
```xml
<?xml version="1.0" ?>
<robot name="robot" xmlns:xacro="http://www.ros.org/wiki/xacro">
  <link name="base_link">
    <inertial>
      <!-- The Mass of the Link -->
      <mass value="10.0"/>
        
      <!-- The Moment of inertia tensor -->
      <inertia ixx="0.346666666667" ixy="0" ixz="0" iyy="0.763333333333" iyz="0" izz="1.08333333333"/>
    </inertial>
    <visual name="base_link_visual">
      <geometry>
        <box size="3.0 2.0 0.4"/>
      </geometry>
      <material name="orange"/>
    </visual>
    <collision name="base_link_collision">
      <geometry>
        <box size="3.0 2.0 0.4"/>
      </geometry>
    </collision>
  </link>
  <link name="base_laser_link">
    <inertial>
      <!-- The Mass of the Link -->
      <mass value="0.037"/>
      <!-- The Moment of Inertia Tensor -->
      <inertia ixx="0.00093075" ixy="0" ixz="0" iyy="0.00093075" iyz="0" izz="0.0006"/>
    </inertial>
    <visual name="base_laser_link_visual">
      <origin xyz="0 0 0.0115"/>
      <geometry>
        <mesh filename="package://omni_description/meshes/hokuyo_utm_30lx.dae"/>
      </geometry>
    </visual>
    <collision name="base_laser_link_collision">
      <origin xyz="0 0 0.0115"/>
      <geometry>
        <box size="0.06 0.06 0.087"/>
      </geometry>
    </collision>
  </link>
  <joint name="base_to_laser" type="fixed">
    <parent link="base_link"/>
    <child link="base_laser_link"/>
    <origin xyz="0 0 0.241"/>
  </joint>
</robot>
```

#### Task 4: Clean up the Code using XACRO
Documentation: http://wiki.ros.org/xacro
##### 1. Indicate that your urdf xml is using xacro
Change the `<robot>` tag into the following:
```xml
<robot name="robot" xmlns:xacro="http://www.ros.org/wiki/xacro">
```
or 
```xml
<robot name="robot" xmlns:xacro="http://wiki.ros.org/xacro">
```
##### 2. Automatically export a urdf file during catkin_make:
Add the following to the end of the CMakelists.txt
```makefile
file(GLOB xacro_files urdf/<your-main-xacro-filename>)
xacro_add_files(${xacro_files} TARGET media_files)
```
So your CMakeLists.txt will look something like this
```makefile
cmake_minimum_required(VERSION 2.8.3)
project(<project-name>)

## Compile as C++11, supported in ROS Kinetic and newer
# add_compile_options(-std=c++11)

find_package(catkin REQUIRED COMPONENTS
  urdf
  xacro
)

include_directories(
# include
  ${catkin_INCLUDE_DIRS}
)

file(GLOB xacro_files urdf/<your-main-xacro-filename>)
xacro_add_files(${xacro_files} TARGET media_files)
```
In the exported filename, `.xacro` will be stripped off. For instance, `robot.urdf.xacro` will become `robot.urdf`

##### 3. Math, Property, Property block, Include and Macro
Documentation: http://wiki.ros.org/xacro

#### Task 5: Add in components from sdf (Gazebo)
Additional Components for Link: http://gazebosim.org/tutorials/?tut=ros_urdf#%3Cgazebo%3EElementsForLinks

Additional Components for Joint: http://gazebosim.org/tutorials/?tut=ros_urdf#Joints

SDF documentation: http://sdformat.org/spec

Useful Gazebo sensor and Plugins: http://gazebosim.org/tutorials?tut=ros_gzplugins#Pluginsavailableingazebo_plugins
##### Add in friction with the environment
```xml
<!-- Choose the link that you want to add in gazebo property -->
<gazebo reference="base_link">
    <mu1>0.2</mu1>
    <mu2>0.2</mu2>
</gazebo>
```

##### Planner Move Plugin
**Description:** model plugin that allows arbitrary objects (for instance cubes, spheres and cylinders) to be moved along a horizontal plane using a geometry_msgs/Twist message. The plugin works by imparting a linear velocity (XY) and an angular velocity (Z) to the object every cycle.

```xml
<gazebo>
  <plugin name="object_controller" filename="libgazebo_ros_planar_move.so">
    <commandTopic>cmd_vel</commandTopic>
    <odometryTopic>odom</odometryTopic>
    <odometryFrame>odom</odometryFrame>
    <odometryRate>20.0</odometryRate>
    <robotBaseFrame>base_footprint</robotBaseFrame>
  </plugin>
</gazebo>
```
##### Laser Plugin
**Description:** simulates laser range sensor by broadcasting LaserScan message as described in sensor_msgs.

With GPU support:
```xml
    <gazebo reference="base_laser_link">
        <sensor type="gpu_ray" name="base_laser_sensor">
            <pose>0 0 0 0 0 0</pose>
            <visualize>true</visualize>
            <update_rate>40</update_rate>
            <ray>
                <scan>
                    <horizontal>
                        <samples>1440</samples>
                        <resolution>1</resolution>
                        <min_angle>-2.35619</min_angle>
                        <max_angle>2.35619</max_angle>
                    </horizontal>
                </scan>
                <range>
                    <min>0.10</min>
                    <max>30.0</max>
                </range>
                <noise>
                    <type>gaussian</type>
                    <mean>0.0</mean>
                    <stddev>0.01</stddev>
                </noise>
            </ray>
            <plugin name="base_laser_controller" filename="libgazebo_ros_gpu_laser.so">
                <topicName>/scan</topicName>
                <frameName>base_laser_link</frameName>
            </plugin>
        </sensor>
    </gazebo>

```
Without GPU support:
Replace:
```xml
<sensor type="gpu_ray" name="base_laser_sensor">
...
<plugin name="base_laser_controller" filename="libgazebo_ros_gpu_laser.so">
```
to 
```xml
<sensor type="ray" name="base_laser_sensor">
...
<plugin name="base_laser_controller" filename="libgazebo_ros_laser.so">
```

Other Useful Plugins please refer to http://gazebosim.org/tutorials?tut=ros_gzplugins#Pluginsavailableingazebo_plugins.

The source code for all the officially maintained plugins: https://bitbucket.org/osrf/gazebo/src/default/plugins/.

How to write a plugin: http://gazebosim.org/tutorials?cat=write_plugin.

#### Task 6: Run in Gazebo!
