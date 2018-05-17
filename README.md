# ARDebug
ARDebug is a tool for monitoring and debugging swarm robotics and multi-robot experiments in real time using augmented reality techniques. The software provides a GUI for displaying internal data reported wirelessly by each robot within the swarm or group. This data is used to augment a real-time video feed of the robots and their environment, using textual representations of the data. The software aims to reduce the time taken to develop and debug swarm robotics experiments and behaviours by giving the developer more immediate access to decision making varaibles, sensor readings, and other key data, when compared to standard console or log-based debugging techniques. The software relies on a tracking system to locate the robots within the image in order to correctly overlay the augmented elements, and a modular software architecture is used to allow for easy integration with a variety of tracking systems. The tracking system used in this reference implementation makes use of [ARuCo](https://www.uco.es/investiga/grupos/ava/node/26) fiducial markers and the associated [OpenCV](https://docs.opencv.org/3.1.0/d5/dae/tutorial_aruco_detection.html) image processing library.

## Installation
Currently ARDebug requires compiling from source, and may require minor modifications to work with your tracking set up. Luckily compiling and building the application is made easy by the Qt toolchain.

###Ubuntu 18.04
Installation under Ubuntu 18.04 is made easy by the up to date versions of libraries available in the default repositories. To install all required libraries and tools simply run:

```
sudo apt-get install libopencv-dev libopencv-contrib-dev qt5-default qtconnectivity5-dev libqt5charts5-dev git python
```

Once this install process has finished you should be able to compile the application using the steps below.

###Ubuntu pre-18.04
Installation on older versions of Ubuntu is marginally more involved but is not overly complex.

ARDebug depends on the presence of further software packages, including OpenCV 3.X.X and opencv_contrib to handle ARuCo-tags.

The following commands can be used to download and build the appropriate source files

```
sudo apt-get update
sudo apt-get upgrade

sudo apt-get install build-essential cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev git qt5-default qtconnectivity5-dev libqt5charts5-dev

git clone https://github.com/Itseez/opencv.git
git clone https://github.com/Itseez/opencv_contrib.git

cd opencv_contrib
git checkout 3.4.1

cd ../opencv
git checkout 3.4.1

mkdir build
cd build

cmake \
-D CMAKE_BUILD_TYPE=Release \
-D CMAKE_INSTALL_PREFIX=/usr/local \
-D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib/modules \
-D BUILD_opencv_bgsegm=OFF \
-D BUILD_opencv_bioinspired=OFF \
-D BUILD_opencv_ccalib=OFF \
-D BUILD_opencv_cnn_3dobj=OFF \
-D BUILD_opencv_cvv=OFF \
-D BUILD_opencv_datasets=OFF \
-D BUILD_opencv_dnn_objdetect=OFF \
-D BUILD_opencv_dnns_easily_fooled=OFF \
-D BUILD_opencv_dpm=OFF \
-D BUILD_opencv_face=OFF \
-D BUILD_opencv_fuzzy=OFF \
-D BUILD_opencv_freetype=OFF \
-D BUILD_opencv_hdf=OFF \
-D BUILD_opencv_line_descriptor=OFF \
-D BUILD_opencv_matlab=OFF \
-D BUILD_opencv_optflow=OFF \
-D BUILD_opencv_ovis=OFF \
-D BUILD_opencv_plot=OFF \
-D BUILD_opencv_reg=OFF \
-D BUILD_opencv_rgbd=OFF \
-D BUILD_opencv_saliency=OFF \
-D BUILD_opencv_sfm=OFF \
-D BUILD_opencv_stereo=OFF \
-D BUILD_opencv_structured_light=OFF \
-D BUILD_opencv_surface_matching=OFF \
-D BUILD_opencv_text=OFF \
-D BUILD_opencv_tracking=OFF \
-D BUILD_opencv_xfeatures2d=OFF \
-D BUILD_opencv_ximgproc=OFF \
-D BUILD_opencv_xobjdetect=OFF \
-D BUILD_opencv_xphoto=OFF \
..

make -j 4

sudo make install
```

At this point all of the required OpenCV libraries should be installed.

You can now download QtCreator from [here](https://www.qt.io/download), choosing the open source option.

### Building (common)
Clone this repository, or download it as a zip file and extract it into a directory. Within this directory run the following commands:

```
mkdir build
cd build
qmake ..
```

At this point you should have a Makefile in the build directory. You can now build the application by running:

```
make -j 4
```

The application can then be launched by running:

```
./ARDebug
```

## Features
The primary feature of ARDebug is providing access to internal robot data. Any data can be submitted using [JSON](https://www.json.org/). The only requirement on the JSON submitted is that each submitted JSON object must have an 'id' value which identifies which robot the data pertains to.

The second main feature of the software is taking this data and displaying it to the user in a nubmer of forms. This includes the augmented video view of the robots, which can be overalaid with all of the data mentioned above, as well as robot's names, IDs, position and orientation. 

For wireless communication ARDebug supports both WiFi and Bluetooth, as well as a combination of both, which can be useful for non-homogenous swarms.

## Interfacing with ARDebug
In order to maximise performance and minimize latency ARDebug receives data from the robots as a JSON object which can be sent using a standard Bluetooth connection, or by submitting UDP packets to the port specified in the GUI. The data-strings received can be any JSON object that contains an 'id' attribute. This attribute should be a string value which identifies the robot.

Code for communicating packets in this format via WiFi from a linux-based robot is provided in <i>ardebug/RobotAPI</i>, primarily for illustrative purposes. An example of how this can be integrated into an example argos robot controller is provided in <i>ardebug/ExampleRobotController/e-puck</i>. An example controller for PsiSwarm robots using bluetooth communication is also provided in <i>ardebug/ExampleRobotController/psi-swarm</i>. Only the main file is provided here. Example projects for the PsiSwarm robot and the used libraries can be found at the [mbed team page](http://os.mbed.com/teams/Psi-Swarm-Robot/).

## Tracking System
Tracking information regarding the position and orientation of each robot is passed through the same interface as the other data, meaning that the code can be easily extended to support a different tracking system from the ARuCo based one included, or even receive tracking data via the network from a seperate machine.

When using the ArUco interface a configuration file is used to map the numeric IDs of the ArUco tags to the textual IDs of the robots used within the application. This configuration file is a JSON file containing an array of objects with "aruco_id" and "robot_id" attributes. If no configuration file is found on application startup then a default one is created which can be used as a basis for your own configuration.

Robot position is described using a simple 'proportional' coordinate system, where both the X and Y coordinate of the robot are stored as a value between 0 and 1, describing the robots position on that axis as a proportion of the length of the camera viewport in that direction. Orientation is simply an angle in degrees, measured clockwise from zero pointing straight up along the Y axis.
