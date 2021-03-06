aruco_ros
=========

Software package and ROS wrappers of the [Aruco][1] Augmented Reality marker detector library.


### Features

<img align="right" src="https://raw.github.com/pal-robotics/aruco_ros/master/aruco_ros/etc/marker_in_hand.jpg" />

 * High-framerate tracking of AR markers
 
 * Generate AR markers with given size and optimized for minimal perceptive ambiguity (when there are more markers to track)
 
 * Enhanced precision tracking by using boards of markers
 
 * ROS wrappers


### Applications

 * Object pose estimation
 * Visual servoing: track object and hand at the same time


### Install a driver USB_CAM
    
You can use `sudo apt-get install ros-<distribution>-usb-cam`, replacing <distribution> with your desired distribution. This will install the package to your `/opt/ros/kinetic/lib directory`.

```
sudo apt-get install ros-kinetic-usb-cam
```

Another option is make a download the repository the usb_cam that has source code for wet build (catkin build). you just need to do the following to build it from source, here you go:

```
mkdir -p ~/catkin-ws/src
cd ~/catkin-ws/src
git clone https://github.com/bosch-ros-pkg/usb_cam.git
cd ..
catkin_make
source ~/catkin-ws/devel/setup.bash
roscd usb_cam
```

* Test the usb_cam

You should rus the roscore on a new terminal 

```
roscore
```

and then in the terminal where you $ source 'd your usb_cam code run:

```
rosrun usb_cam usb_cam_node
```

Make sure you have your camera connected, before running the above command!!!

```
ls /dev/video0
```

or

```
ls /dev/video1
```

You view the captured image on rviz

```
rosrun rviz rviz
```

### Create the file 'head_camera.yaml'

First you should create a folder in '.ros/camera_info'. After that, save the file 'head_camera.yaml'

```
cd .ros/
mkdir camera_info
subl camera_info/head_camera.yaml
```

```
image_width: 856
image_height: 480
camera_name: usb_cam
camera_matrix:
  rows: 3
  cols: 3
  data: [537.292878, 0.000000, 427.331854, 0.000000, 527.000348, 240.226888, 0.000000, 0.000000, 1.000000]
distortion_model: plumb_bob
distortion_coefficients:
  rows: 1
  cols: 5
  data: [0.004974, -0.000130, -0.001212, 0.002192, 0.000000]
rectification_matrix:
  rows: 3
  cols: 3
  data: [1.000000, 0.000000, 0.000000, 0.000000, 1.000000, 0.000000, 0.000000, 0.000000, 1.000000]
projection_matrix:
  rows: 3
  cols: 4
  data: [539.403503, 0.000000, 429.275072, 0.000000, 0.000000, 529.838562, 238.941372, 0.000000, 0.000000, 0.000000, 1.000000, 0.000000]
```

### Calibration

You should follow the steps of documentation above:

* [Tutorials - MonocularCalibration](http://wiki.ros.org/camera_calibration/Tutorials/MonocularCalibration)

* [Tutorials(Another Option) - MonocularCalibration](http://ros-developer.com/2017/04/23/camera-calibration-with-ros/)

```
roslaunch usb_cam usb_cam-test.launch
```

```
rosrun camera_calibration cameracalibrator.py --size 8x6 --square 0.02415 image:=/usb_cam/image_raw camera:=/usb_cam --no-service-check
```

Make the download of Calibration Checkerboard:

* [Calibration Checkerboard Collection](https://markhedleyjones.com/projects/calibration-checkerboard-collection)

<img align="bottom" src="https://github.com/alanprodam/aruco_ros/blob/aruco-3.0.4/aruco_ros/etc/calibration.jpg" />

### Create Dataset and Export Image Data from a Bag File

```
roslaunch usb_cam usb_cam-test.launch
```

```
roslaunch usb_cam usb_cam-test.launch
```


* [How to export image](http://wiki.ros.org/rosbag/Tutorials/Exporting%20image%20and%20video%20data)

### ROS API

#### Messages

 * aruco_ros/Marker.msg

        Header header
        uint32 id
        geometry_msgs/PoseWithCovariance pose
        float64 confidence

 * aruco_ros/MarkerArray.msg

        Header header
        aruco_ros/Marker[] markers

### Kinetic changes

* Updated the [Aruco][1] library to version 3.0.4

* Changed the coordinate system to match the library's, the convention is shown
  in the image below, following rviz conventions, X is red, Y is green and Z is
  blue.
<img align="bottom" src="/aruco_ros/etc/new_coordinates.png"/>

### Test it with REEM

 * Open a REEM in simulation with a marker floating in front of the robot. This will start the stereo cameras of the robot too. Since this is only a vision test, there is nothing else in this world apart from the robot and a marker floating in front of it. An extra light source had to be added to compensate for the default darkness.

    ```
    roslaunch reem_gazebo reem_gazebo.launch world:=floating_marker
    ```
 * Launch the `image_proc` node to get undistorted images from the cameras of the robot.
 
    ```
    ROS_NAMESPACE=/stereo/right rosrun image_proc image_proc image_raw:=image
    ```
 * Start the `single` node which will start tracking the specified marker and will publish its pose in the camera frame
 
    ```
    roslaunch aruco_ros single.launch markerId:=26 markerSize:=0.08 eye:="right"
    ```

    the frame in which the pose is refered to can be chosen with the 'ref_frame' argument. The next example forces the marker pose to
    be published with respect to the robot base_link frame:

    ```
    roslaunch aruco_ros single.launch markerId:=26 markerSize:=0.08 eye:="right" ref_frame:=/base_link
    ```
    
 * Visualize the result
 
    ```    
    rosrun image_view image_view image:=/aruco_single/result
    ```

<img align="right" src="https://raw.github.com/pal-robotics/aruco_ros/master/aruco_ros/etc/reem_gazebo_floating_marker.png"/>


[1]: http://www.sciencedirect.com/science/article/pii/S0031320314000235 "Automatic generation and detection of highly reliable fiducial markers under occlusion by S. Garrido-Jurado and R. Muñoz-Salinas and F.J. Madrid-Cuevas and M.J. Marín-Jiménez 2014"