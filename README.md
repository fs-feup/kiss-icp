<div align="center">
    <h1>KISS-ICP</h1>
    <a href="https://github.com/PRBonn/kiss-icp/releases"><img src="https://img.shields.io/github/v/release/PRBonn/kiss-icp?label=version" /></a>
    <a href="https://github.com/PRBonn/kiss-icp/blob/main/LICENSE"><img src="https://img.shields.io/github/license/PRBonn/kiss-icp" /></a>
    <a href="https://github.com/PRBonn/kiss-icp/blob/main/"><img src="https://img.shields.io/badge/Linux-FCC624?logo=linux&logoColor=black" /></a>
    <a href="https://github.com/PRBonn/kiss-icp/blob/main/"><img src="https://img.shields.io/badge/Windows-0078D6?st&logo=windows&logoColor=white" /></a>
    <a href="https://github.com/PRBonn/kiss-icp/blob/main/"><img src="https://img.shields.io/badge/mac%20os-000000?&logo=apple&logoColor=white" /></a>
    <br />
    <br />
    <a href=https://user-images.githubusercontent.com/21349875/219626075-d67e9165-31a2-4a1b-8c26-9f04e7d195ec.mp4>Demo</a>
    <span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
    <a href="https://github.com/PRBonn/kiss-icp/blob/main/README.md#Install">Install</a>
    <span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
    <a href="https://github.com/PRBonn/kiss-icp/blob/main/ros">ROS 1</a>
    <span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
    <a href="https://github.com/PRBonn/kiss-icp/blob/main/ros">ROS 2</a>
    <span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
    <a href=https://user-images.githubusercontent.com/21349875/214578180-b1d2431c-8fff-440e-aa6e-99a1d85989b5.mp4
>ROS Demo</a>
    <span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
    <a href=https://www.ipb.uni-bonn.de/wp-content/papercite-data/pdf/vizzo2023ral.pdf>Paper</a>
    <span>&nbsp;&nbsp;•&nbsp;&nbsp;</span>
    <a href=https://github.com/PRBonn/kiss-icp/issues>Contact Us</a>
  <br />
  <br />

[KISS-ICP](https://www.ipb.uni-bonn.de/wp-content/papercite-data/pdf/vizzo2023ral.pdf) is a LiDAR Odometry pipeline that **just works** on most of the cases without tunning any parameter.

  <p align="center">
    <a href="https://user-images.githubusercontent.com/21349875/219626075-d67e9165-31a2-4a1b-8c26-9f04e7d195ec.mp4"><img alt="KISS-ICP Demo" src="https://user-images.githubusercontent.com/21349875/211829074-474bec08-0129-4e34-85e7-62265e44a7de.png"></a>
  </p>
</div>

<hr />

## FS-FEUP AS Documentation

### Install kiss-icp pipeline

If you just wish to install the pipeline, without ROS (unadvised):

```sh
pip install kiss-icp
```

Next to test if the system is working and to understand instructions on how to run it, do the following command

```sh
kiss_icp_pipeline --help
```

### Install kiss-icp ROS wrappers

From the root folder of the [AS repo](https://github.com/fs-feup/autonomous-systems/tree/main) folder:

```sh
git submodule init
git submodule update --recursive --remote
```

This will clone the kiss-icp source repository to the ext/ folder.

Now go to the wd (root folder) and build it by running the following command:

```sh
colcon build
```

If you are having problems with other packages, you can try to only build the kiss-icp

```sh
colcon build --packages-select kiss_icp
```

Finally run the following command to finish the build

```sh
source ./install/setup.bash
```

### How to run

The only required argument to provide is the
 topic name so KISS-ICP knows which PointCloud2 to process:

```sh
ros2 launch kiss_icp odometry.launch.py bagfile:=<path_to_rosbag> topic:=<topic_name>
```

You can optionally launch the node with any bagfile, and play the bagfiles on a different shell:
```sh
ros2 launch kiss_icp odometry.launch.py topic:=<topic_name>
```
and then,

```sh
ros2 bag play <path>*.bag
```

The topic_name should be the topic of the bagfile. In case you do not know, run the following commands and see which topic has the lidar data from the bagfile, and then repeat the commands above:

```sh
ros2 bag play <path>*.bag
ros2 topic list
```

### Internal structure of the package

There are two nodes, one run kiss-icp and another used to just send data to rviz, in order to visualize the simulation.

The kiss-icp node is the node where the point cloud is recieved and the kiss-icp is executed. Inside this node there are several parameters that can be changed, from both ROS2 and kiss-icp.

**ROS2 parameters:** topic, bagfile, visualize, odom_frame, base_frame, publish_odom_tf.

**Kiss-icp parameters:** deskew, preprocess, max_range, min_range, voxel_size, max_points_per_voxel, fixed_threshold


* deskew - Compensate the frame by estimatng the velocity between the given poses
* preprocess - Crop the frame with max/min ranges
* max_range - This sets the maximum range for valid points in the LiDAR scan. Points beyond this range will be ignored
* min_range - This sets the minimum range for valid points in the LiDAR scan. Points closer than this will be ignored
* voxel_size - This parameter controls the size of voxels (3D grids used to represent the environment). A smaller size leads to higher resolution but also increases computational cost
* max_points_per_voxel - This limits the number of points allowed within each voxel
* fixed_threshold - The fixed_threshold defines the maximum allowed distance difference between a point and its candidate match (comparing the previous to the current frame)

In most of the cases it should be enough to use the default parameters "as is". With this said, if needed, they can be changed inside the files (see the README in "autonomous-systems/lib/kiss-icp/config/README.md" to understand how to create a new config file), as well as using services available by the wrapper. To see this services just do:

```sh
ros2 service list
```

This should be the output

![Service list](./Screenshot%20from%202024-02-09%2009-35-54.png)

Kiss-icp node does not have any ros2 actions.

This node publishes to some topics which are the following:

* /kiss/frame: Gives the pointcloud at that exact moment (frame)
* /kiss/keypoints: Outputs data of the reduced pointcloud only composed by keypoints
* /kiss/local_map: Gives all the pointclouds of previous scans from the sensor
* /kiss/odometry: Outputs the Pose and Orietation of the vehicle in the current frame
* /kiss/trajectory: Returns an array of all the odometry (Pose and Orientation) since the beginning until the current frame

The rviz2 node is a simple node that only serves to visualize the map, and can read from a bagfile.

To have a better understanding of kiss-icp see this git repository https://github.com/PRBonn/kiss-icp?tab=readme-ov-file

