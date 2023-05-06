# **Air Quality control**

We have designed and assembled an IoT closed loop air quality control device to be mounted on rUBot_2.0 prototype.

The mechanical structure is descrived bellow:
![](./Images/rubot_custom/1_osoyoo.png)

His main characteristics are: 
- contains a Raspberri Pi 4 that will allow the rubot's connection to the IoT and will control it's movement
- has four omnidirectional wheels that allow the robot to move in any desired direction
- contains a LIDAR (Laser Imaging Detection and Ranging), which determines ranges by targeting an object or surface via an infrared laser
- has a camera that allows the visualization of the rubot's environment via 3D visualizations software tools
- *to be continued*

In this document we will describe:
- Device structure
- Web arduino SW description
- Sensors description
- Ozone generation device description
- PCB board design
- Arduino code 
- Experimental results


## **1. Device structure**

First of all, we have to create the "rubot_mecanum_description" package where we will create the rUBot model. In case you want to create it from scratch, type:
```shell
cd ~/Desktop/ROS_github/rubot_mecanum_ws/src
catkin_create_pkg rubot_mecanum_description rospy
cd ..
catkin_make
```

## **3. Sensors description**

In this project's air quality control circuit, three different sensors will be used to assure accurate and reliable CO2 atmosphere levels as well as humidity and temperature values:
*foto mq2*
- The first sensor used is an MQ2 gas sensor,  that detects concentrations of different air gases such as propane, methane, hydrogen, alcohol, smoke and carbon monoxide. The gas' concentration is measured using a voltage divider network and the values are given in ppm (parts per million)
*foto dht22*
- The second sensor is a temperature and humidity sensor, the DHT22. It offers a high precision data acquisiton and stability due to it's temperature compensation and calibration. Humidity values are given in percentage units (%) and temperature values are given in degrees celcius (ºC).
*foto scd30*
- The third and biggest sensor is the SCD30 Sensirion sensor, that measures carbon dioxide, temperature and humidity air values. The CO2 levels are given in ppm. Its main characteristic is its small size and height, which allows easy integration into different applications.
```
