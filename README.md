## This is the ROS2 Port of the uuv_plume_simulator package.

It currently works with ROS2 Humble.
Main Branch is compatible with Gazebo Garden.


### Current Status:
- Particle Plume is working in ROS2

- Ocean Current from Gazebo is considered in the simulation
  - Ocean Current is taken from the [hydrodynamics plugin in Gazebo](https://gazebosim.org/api/gazebo/6.9/classignition_1_1gazebo_1_1systems_1_1Hydrodynamics.html)
- PointCloud2 Bridge to Gazebo
  - The PointCloud2 is published to the gz topic /particles2


### HowTo.


Bridge the Topics:
    
    - ros_topic_name: "/ocean_current"
      gz_topic_name: "/ocean_current"
      ros_type_name: "geometry_msgs/msg/Point"
      gz_type_name: "gz.msgs.Vector3d"
      direction: GZ_TO_ROS

    - ros_topic_name: "/particles2"
      gz_topic_name: "/particles2"
      ros_type_name: "sensor_msgs/msg/PointCloud2"
      gz_type_name: "gz.msgs.PointCloudPacked"
      direction: ROS_TO_GZ

    - ros_topic_name: "/time_creation"
      gz_topic_name: "/time_creation"
      ros_type_name: "ros_gz_interfaces/msg/Float32Array"
      gz_type_name: "gz.msgs.Float_V"
      direction: ROS_TO_GZ

Start the particle plume

    ros2 service call /create_passive_scalar_turbulent_plume uuv_plume_msgs/srv/CreatePassiveScalarTurbulentPlume "turbulent_diffusion_coefficients: {x: 0.05, y: 0.05, z: 0.05}  
    source: {x: 0.0, y: 0.0, z: -40.0}  
    buoyancy_flux: 0.05 
    stability_param: 0.001 
    n_points: 100000 
    max_particles_per_iter: 100 
    x_min: -200.0 
    x_max: 200.0 
    y_min: -75.0 
    y_max: 75.0 
    z_min: -50.0 
    z_max: 0.0 
    max_life_time: -1"

Start the Ocean Current

    ign topic -t /ocean_current -m gz.msgs.Vector3d -p 'x: 0.2, y:0, z:0'

### TODO: 
- Another branch for Gazebo Garden.
- Particle Visualizer in Gazebo (Currently only works in RViz)
  - Subscribe to gz topic /particles2 and publish to /particles as Markers?
  - https://github.com/gazebosim/gz-sim/tree/gz-sim8/examples/standalone/marker
  - or use https://github.com/gazebosim/gz-gui/pull/346 (only available from gazebo garden ?)
- Add a launch file for the plume server
  - Set initial plume in the simulation 
- Add a launch file for the current
  - Set initial Current in Simulation
-  Port [uuv_cpc_sensor](uuv_cpc_sensor) to ROS2 
   - is this already done [here](https://github.com/Liquid-ai/Plankton/blob/master/uuv_sensor_plugins/uuv_sensor_ros_plugins/src/CPCROSPlugin.cpp)?


---

# Original ReadMe:


# Plume Simulator

[![Build Status](https://travis-ci.org/uuvsimulator/uuv_plume_simulator.svg?branch=master)](https://travis-ci.org/uuvsimulator/uuv_plume_simulator)
[![GitHub issues](https://img.shields.io/github/issues/uuvsimulator/uuv_plume_simulator.svg)](https://github.com/uuvsimulator/uuv_plume_simulator/issues)
![License](https://img.shields.io/badge/license-Apache%202-blue.svg)

> Link to the `uuv_plume_simulator` repository [here](https://github.com/uuvsimulator/uuv_plume_simulator)

> Link to the [documentation page](https://uuvsimulator.github.io/packages/uuv_plume_simulator/intro/)

> Chat on [Discord](https://discord.gg/zNauF2F)

This repository contains ROS nodes and messages necessary to simulate a turbulent
plume, which is an implementation of the algorithm described in [1]. This repository
is complementary to the [Unmanned Underwater Vehicle Simulator (UUV Simulator)](https://github.com/uuvsimulator/uuv_simulator),
an open-source project extending the simulation capabilities of the robotics
simulator Gazebo to underwater vehicles and environments. For installation and
usage instructions, please refer to the [documentation pages](https://uuvsimulator.github.io/).

![Visualization in RViz](images/plume.png)

[[1] Yu Tian and Aiqun Zhang, "Simulation environment and guidance system for
    AUV tracing chemical plume in 3-dimensions," 2010 2nd International
    Asia Conference on Informatics in Control, Automation and Robotics
    (CAR 2010), Mar. 2010.](http://ieeexplore.ieee.org/document/5456812/)

## Purpose of the project

This software is a research prototype, originally developed for the EU ECSEL
Project 662107 [SWARMs](http://swarms.eu/).

The software is not ready for production use. However, the license conditions of the
applicable Open Source licenses allow you to adapt the software to your needs.
Before using it in a safety relevant setting, make sure that the software
fulfills your requirements and adjust it according to any applicable safety
standards (e.g. ISO 26262).

## Requirements

To simulate the plume, please refer to the [UUV Simulator](https://github.com/uuvsimulator/uuv_simulator)
repository and follow the installation instructions of the package. Then you can clone
this package in the `src` folder of you catkin workspace

```
cd ~/catkin_ws/src
git clone https://github.com/uuvsimulator/uuv_plume_simulator.git
```

and then build your catkin workspace

```bash
cd ~/catkin_ws
catkin_make # or <catkin build>, if you are using catkin_tools
```

## Example of usage

The plume simulator package are installed, one demonstration can be run to
visualize the plume particles in RViz. To start it, run

```
roslaunch uuv_plume_simulator start_plume_example.launch
```

To start an example of a turbulent plume model, run the script

```
roslaunch uuv_plume_simulator start_demo_turbulent_plume.launch
```

In this case, the plume runs independently from the [UUV Simulator](https://github.com/uuvsimulator/uuv_simulator).
In order to start this example using one of the Gazebo worlds from UUV Simulator and
therefore use the current velocity topic generated by [a Gazebo plugin](https://github.com/uuvsimulator/uuv_simulator/blob/master/uuv_world_plugins/uuv_world_ros_plugins/include/uuv_world_ros_plugins/UnderwaterCurrentROSPlugin.hh),
start the example first as

```
roslaunch uuv_plume_simulator start_plume_example.launch use_gazebo:=true
```

and then

```
roslaunch uuv_plume_simulator start_demo_turbulent_plume.launch use_gazebo:=true
```

If no current topic is available, the particles will accumulate
around the plume source.

To add a particle concentration sensor to the vehicle, check the URDF macro snippets
found [here](https://github.com/uuvsimulator/uuv_simulator/blob/master/uuv_sensor_plugins/uuv_sensor_plugins_ros/urdf/chemical_concentration.xacro)
that can be used to add one to the robot description.

## License

`uuv_plume_simulator` is open-sourced under the Apache-2.0 license. See the
[LICENSE](https://github.com/uuvsimulator/uuv_plume_simulator/blob/master/LICENSE) file for details.
