## Docker ROS Kinetic with V-REP

This image is used for testing the [V-REP](http://www.coppeliarobotics.com/index.html) simulator with ROS Kinetic from within a Docker container.

The image is also customized such that (for details see [this](https://github.com/gandrein/docker-xenial-with-terminator-zsh) repository):
* the docker image is created with a non-root user (default user-name `docker`)
* [Oh My ZSH](http://ohmyz.sh/) is installed and configured for the non-root user
* docker containers are launched with [`terminator`](https://gnometerminator.blogspot.nl/p/introduction.html) as the default terminal emulator (as opposed to default `gnome-terminal`)
* bash completion for Docker image names and tags when launching the container by using the `./run_docker.sh` script

---

### Contents
1. [Requirements](#1-requirements)
2. [Building the image](#2-building-the-image)
3. [Running the container](#3-running-the-container)
4. [V-REP usage](#4-v-rep-usage)
5. [V-REP - ROS interface](#4-v-rep---ros-interface)

### 1. Requirements

This docker container has been build and tested on a machine running Ubuntu 16.04 with the following packages installed
* `docker` version `17.09.0-ce`
* [`nvidia-docker`](https://github.com/NVIDIA/nvidia-docker/wiki/Installation-(version-1.0)) version `1.0`
* [V-REP PRO EDU V3.4.0 rev1](http://www.coppeliarobotics.com/downloads.html)

#### Dependencies
This Docker image uses as base the `codookie/xenial:ros-kinetic-base-nvidia` image. This has to be build previously in order to build this image. 

### 2. Building the image

There are two ways of building this image:
* using the top-level [`Makefile`](./../Makefile) and the `make` command [_Recommended_]
* using this folder's [build](./build.sh) script. 

The first method will handle the base image dependency for you, while in the second approach you have to make sure you have build 
the base image `ros-kinetic-base-nvidia` yourself.

### 3. Running the container

In a terminal enter [./run_docker.sh](./run_docker.sh) with the image name from the previous step. This will run and remove the docker image upon exit (i.e., it is ran with the `--rm` flag).
```
./run_docker.sh GIVEN_IMAGE_NAME
```

#### Bash auto-completion for `./run_docker.sh`

When using `./run_docker.sh` in a bash shell to launch the container, source the [bash_docker_images_completion.sh](./../configs/bash_docker_images_completion.sh) script. Now you should be able to get the names of the available docker images on your system whenever you type 
```
./run_docker.sh <TAB><TAB>
```

### 4.V-REP usage

Note that V-REP simulator is not installed nor copied in the Docker container. Rather, the host V-REP installation folder is volume mounted in the Docker container at runtime by adding the following line to the `nvidia-docker run` command (see the contents of the [run_docker.sh](./run_docker.sh) scrip),
```
  -v $HOME/Projects/devs/v-rep-edu:/home/docker/v-rep
```
where you should replace the source path `$HOME/Projects/devs/v-rep-edu` with your own V-REP installation location. 

A similar mounting can be done for the `CATKIN Workspace` directory, e.g., if you want to save the workspace data on the host side.

#### Testing functionality

##### Test V-REP

When the Docker container is launched, a [terminator](https://gnometerminator.blogspot.nl/p/introduction.html) window opens. Navigate to `$HOME/v-rep` and type 
```
./vrep.sh
``` 
The V-REP simulator GUI should launch. 


##### Test ROS

While inside the container call `roscore`. The `ros master` should be launched. 

### 5. V-REP - ROS Interface

There are two ways you can build the V-REP - ROS Interface
1. Follow the online V-REP tutorial at
	* [http://www.coppeliarobotics.com/helpFiles/en/rosTutorialIndigo.htm](http://www.coppeliarobotics.com/helpFiles/en/rosTutorialIndigo.htm)
	* the tutorial works fine also with ROS Kinetic and you should be able to build your own ROS Interface (**libv_repExtRosInterface.so**) and follow all the suggested examples. 
	* note that in the example based on scene **rosInterfaceTopicPublisherAndSubscriber.ttt** the camera data is published on the topic `/image` and not `visionSensorData`, hence the correct command to visualize the image data published on the rostopic is 
	```
	$ rosrun image_view image_view image:=/image
	```
2. Follow the instructions in the README file provided with the V-REP installation. 
	* For [V-REP PRO EDU V3.4.0 rev1](http://www.coppeliarobotics.com/downloads.html), this file is titled _ros_vrep_rosinterface_install_guide.txt_ and can be found in your own V-REP's installation folder, at [<v_rep_edu_folder>/programming/ros_packages/]()
	* Note that the _ros_vrep_rosinterface_install_guide.txt_ is not up to date and you may encounter the following issues when trying to build the ROS interface yourself:
	  * both mentioned `git` repositories have been moved to the [CoppeliaRobotics](https://github.com/CoppeliaRobotics) github account
	  * `xsltproc` package is required as mentioned in the dependency of the [v_repStubsGen](https://github.com/CoppeliaRobotics/v_repStubsGen) repository. The [Dockerfile](./Dockerfile) has been updated to perform this step for you.
	  * no need to perform Step 5 - _install v_repStubsGen (https://github.com/fferri/v_repStubsGen.git)_ if you clone the [v_repExtRosInterface](https://github.com/CoppeliaRobotics/v_repExtRosInterface) repository recursively by doing,
	    ```
	    git clone --recursive https://github.com/CoppeliaRobotics/v_repExtRosInterface.git vrep_ros_interface
	    ```
3. Update the binaries in the mounted V-REP folder with the ones from the built `catkin` workspace
4. Launch V-REP and monitor the terminal, you should see
```
./vrep.sh
...
Plugin 'RosInterface': loading...
Plugin 'RosInterface': load succeeded.
...
```
4. Upon successful RosInterface load, checking the available nodes gives this:
```
$ rosnode list
/rosout
/vrep_ros_interface
```
5. Test with the examples provided in the [online tutorial](http://www.coppeliarobotics.com/helpFiles/en/rosTutorialIndigo.htm)


