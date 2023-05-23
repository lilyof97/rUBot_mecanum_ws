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

*MQ2 Gas sensor*: 

![](./Images/air_quality/mq2.PNG)

- The first sensor used is an MQ2 gas sensor,  that detects concentrations of different air gases such as propane, methane, hydrogen, alcohol, smoke and carbon monoxide. The gas' concentration is measured using a voltage divider network and the values are given in ppm (parts per million)

*DHT22 temperature and humidity sensor*: 

![](./Images/air_quality/dht22.PNG)

- The second sensor is a temperature and humidity sensor, the DHT22. It offers a high precision data acquisiton and stability due to it's temperature compensation and calibration. Humidity values are given in percentage units (%) and temperature values are given in degrees celcius (ºC).

*SCD30 CO2, temperature and humidity sensor*:

![](./Images/air_quality/scd30.PNG)

- The third and biggest sensor is the SCD30 Sensirion sensor, that measures carbon dioxide, temperature and humidity air values. The CO2 levels are given in ppm. Its main characteristic is its small size and height, which allows easy integration into different applications.
This particular sensor will be key for the whole air quality control device since the CO2 air concentration value is what will switch the ozone generator on and off.

## **4. Ozone generator device description**

An ozone generator is a device that intentionally produces ozone gas. Ozone (O3) is a gas that has an extra oxygen atom in comparison with the air we breathe (O2), and it is actually harmful for health. Nonetheless, it can be benefitial to purify air when it is used in small quantities (up to 50 ppb). Ozone generators are able to produce ozone by applying an electrical charge to the air that goes through it. This splits apart regular O2 atoms and forces them to attach to other O2 atoms, creating O3. It is important that no animals or humans are present in rooms where ozone generators are working (generating O3), that's why most of these devices are switched on at night or when rooms are empty, to prevent any kind of health harm.

The ozone generator used in this application is the "Handy Ozone Generator" by Feel Lagoom. It is a small device (15 x 15 cm) that works thanks to an integrated lithium battery that lasts up to 15h before needing to be recharged. For the circuit being, this integrated battery will not be used and the generator will be connected to a 5V pin from the ESP32. 

*Handy Ozone Generator*: 

![](./Images/air_quality/generador_ozo.PNG)

The device will be switched off by default, and will only be switched on when the CO2 levels detected by the SCD30 sensor are above a chosen threshold value. The CO2 air value is usually around 800 ppm in normal conditions, and will be considered harmful for health when it reaches values above 1200 ppm (for this specific application and its scope). Every time the 1200 ppm threshold is surpassed, the ozone generator will switch on and start to purify air, when the CO2 value goes down again under 1200 ppm, the ozone generator will switch off.

## **5. PCB Board design**

## **6. Arduino Code**

The air quality control circuit is controlled by the ESP32. Every circuit's component will be connected to the Arduino, including the ozone generator. 
An Arduino code is generated via Arduino Cloud to fetch and control every sensor's data. Its main aim is to display the fetched data on to the Arduino Cloud's Dashboard, so users can have access to CO2, gas, humidity and temperature air levels via IoT.

The first step is to include and define every sensor's library and ID. It is also important to identify the ESP32's pins that will be connected to every sensor:

```shell
#include <Wire.h>
#include <DHT.h>         //Include library for the DTH22 sensor
#define DHTTYPE  DHT22   //Define DHT22 sensor as DHT22
#define DHTPIN    4      // Define the ESP32 pin that enables connection with the DHT22 (in this case, pin number 4)
#include <SparkFun_SCD30_Arduino_Library.h> //Include library for the SCD30 sensor
#include <arduino-timer.h>

SCD30 airSensor;                            //Define the SCD30 sensor as airSensor

int Sensor_input = 34;                      //Define the ESP32 pin that enables connection with the MQ2 (pin number 34)

auto sensor_timer = timer_create_default(); //Create auto timer to fetch data periodically

DHT dht(DHTPIN, DHTTYPE, 22);               //Define the DHT22

#include "thingProperties.h"
```

Next, a boolean function is created to allow periodical data fetching from the sensors:

```shell
bool get_sensor_readings(void *) {
  readValues();
  return true; // repeat? true
}
```

A setup void is needed to initiate the sensors' and Arduino Cloud's connection. In the same void, the speed of data readings is indicated.

```shell
void setup()
{
  Serial.begin(115200);
  
  pinMode(23, OUTPUT);    //Define pin 23 from the ESP32 as an output, this will control the ozone generator's switch on and off
  digitalWrite(23, LOW);  //The output has a low value (0) by default
  
  // Defined in thingProperties.h
  initProperties();

  // Connect to Arduino IoT Cloud
  ArduinoCloud.begin(ArduinoIoTPreferredConnection);
  
  /*
     The following function allows you to obtain more information
     related to the state of network and IoT Cloud connection and errors
     the higher number the more granular information you’ll get.
     The default is 0 (only errors).
     Maximum is 4
 */
 
  setDebugMessageLevel(2);
  ArduinoCloud.printDebugInfo();
    
  dht.begin();                      //Begin the DHT22 sensor
  Wire.begin();                     //Begin wire

  if (airSensor.begin() == false)   //If the SCD30 sensor is not detected, the code will freeze
  {
    Serial.println("Air sensor not detected. Please check wiring. Freezing...");
    while (1)
      ;
  }

  sensor_timer.every(5000, get_sensor_readings);  //Fetch data every 5000 ms
}
```

In the main code loop, there are two essential functions:
- ArduinoCloud.update(): it is called to update the values on the cloud and the status of the update is displayed on the screen (or dashboard)
- sensor_timer.tick(): it sets a timer that calls a function which indicates when it is time to fetch data (in this case, every 5000 ms)

```shell
void loop()
{
  ArduinoCloud.update();
  
  sensor_timer.tick();
}
```
The readValues() void is responsible for every data reading, for each one of the sensors used. Here, the values will be read from the sensors and written onto the ArduinoCloud's display and dashboard created for this specific application.

```shell
void readValues(){  
  humedad = dht.readHumidity();                 //Read humidity from DHT22. The value will be saved as "humedad"
  temperatura = dht.readTemperature();          //Read temperature from DHT22. The value will be saved as "temperatura"
  gas_combustible = analogRead(Sensor_input);   //Read gas from the MQ2. The value will be saved as "gas_combustible"
  humedad_SCD = airSensor.getHumidity();        //Read humidity from the SC30. The value will be saved as "humedad_SCD"
  temperatura_SCD = airSensor.getTemperature(); //Read temperature from the SC30. The value will be saved as "temperatura_SCD"
  cO2 = airSensor.getCO2();                     //Read CO2 from the SC30. The value will be saved as "cO2"
  
  //Data is printed
  Serial.println("Humedad: "); 
  Serial.println(humedad);
  Serial.println("Temperatura: ");
  Serial.println(temperatura);
  Serial.println("Gas Sensor: ");  
  Serial.print(gas_combustible);   
  Serial.print("\t");
  Serial.print("\t");
  
  //If MQ2's gas values are above 1500 ppm, there will be a warning indicating that there's too much gas
  if (gas_combustible > 1500) {    
    Serial.println("Gas");  
  }
  else {
    Serial.println("No Gas");
  }
  
  Serial.println(cO2);
  Serial.println(temperatura_SCD, 1);
  Serial.println(humedad_SCD, 1);
  Serial.println("Waiting for new data");
  
  //If the CO2 SC30's levels are above 1000 ppm, the ozone generator will switch on
  if (cO2 > 1000)
  {
  pinMode(23, OUTPUT);
  digitalWrite(23, HIGH);
  }
  else {
  digitalWrite(23, LOW);   //Otherwise, it will be switched off
  }
  
  delay(500);
}
```
Finally, there are voids that are automatically created when new variables are defined. These variables correspond to each one of the sensors' data collection type. These voids are empty.

```shell
/*
  Since Humedad is READ_WRITE variable, onHumedadChange() is
  executed every time a new value is received from IoT Cloud.
*/
void onHumedadChange()  {
  // Add your code here to act upon Humedad change
}
/*
  Since Temperatura is READ_WRITE variable, onTemperaturaChange() is
  executed every time a new value is received from IoT Cloud.
*/
void onTemperaturaChange()  {
  // Add your code here to act upon Temperatura change
}
/*
  Since GasCombustible is READ_WRITE variable, onGasCombustibleChange() is
  executed every time a new value is received from IoT Cloud.
*/
void onGasCombustibleChange()  {
  // Add your code here to act upon GasCombustible change
}
/*
  Since HumedadSCD is READ_WRITE variable, onHumedadSCDChange() is
  executed every time a new value is received from IoT Cloud.
*/
void onHumedadSCDChange()  {
  // Add your code here to act upon HumedadSCD change
}
/*
  Since TemperaturaSCD is READ_WRITE variable, onTemperaturaSCDChange() is
  executed every time a new value is received from IoT Cloud.
*/
void onTemperaturaSCDChange()  {
  // Add your code here to act upon TemperaturaSCD change
}
/*
  Since CO2SCD is READ_WRITE variable, onCO2SCDChange() is
  executed every time a new value is received from IoT Cloud.
*/

void onCO2Change()  {
  // Add your code here to act upon CO2 change
}
```



Here's the full application code:

```shell
/*
  Reading CO2, humidity and temperature from the SCD30
  Reading Gas from MQ2
  Reading humidity and temperature from DHT22
  By: Liliana Oliveira 
  Universitat de Barcelona
  Date: May 2023
*/

#include <Wire.h>
#include <DHT.h>         //Include library for the DTH22 sensor
#define DHTTYPE  DHT22   //Define DHT22 sensor as DHT22
#define DHTPIN    4      // Define the ESP32 pin that enables connection with the DHT22 (in this case, pin number 4)
#include <SparkFun_SCD30_Arduino_Library.h> //Include library for the SCD30 sensor
#include <arduino-timer.h>

SCD30 airSensor;                            //Define the SCD30 sensor as airSensor

int Sensor_input = 34;                      //Define the ESP32 pin that enables connection with the MQ2 (pin number 34)

auto sensor_timer = timer_create_default(); //Create auto timer to fetch data periodically

DHT dht(DHTPIN, DHTTYPE, 22);               //Define the DHT22

#include "thingProperties.h"

//Boolean function that allows periodical data fetching from the sensors
bool get_sensor_readings(void *) {
  readValues();
  return true; // repeat? true
}

void setup()
{
  Serial.begin(115200);
  
  pinMode(23, OUTPUT);    //Define pin 23 from the ESP32 as an output. This will control the ozone generator's switch on and off.
  digitalWrite(23, LOW);  //The output has a low value (0) by default
  
  // Defined in thingProperties.h
  initProperties();

  // Connect to Arduino IoT Cloud
  ArduinoCloud.begin(ArduinoIoTPreferredConnection);
  
  /*
     The following function allows you to obtain more information
     related to the state of network and IoT Cloud connection and errors
     the higher number the more granular information you’ll get.
     The default is 0 (only errors).
     Maximum is 4
 */
  setDebugMessageLevel(2);
  ArduinoCloud.printDebugInfo();
    
  dht.begin();                      //Begin the DHT22 sensor
  Wire.begin();                     //Begin wire

  if (airSensor.begin() == false)   //If the SCD30 sensor is not detected, the code will freeze
  {
    Serial.println("Air sensor not detected. Please check wiring. Freezing...");
    while (1)
      ;
  }

  sensor_timer.every(5000, get_sensor_readings);  //Fetch data every 5000 ms
}

void loop()
{
  ArduinoCloud.update();
  
  sensor_timer.tick();
}

//ReadValues void where data is fetched and displayed on the Arduino Cloud's monitor and Dashboard 
void readValues(){  
  humedad = dht.readHumidity();                 //Read humidity from DHT22. The value will be saved as "humedad"
  temperatura = dht.readTemperature();          //Read temperature from DHT22. The value will be saved as "temperatura"
  gas_combustible = analogRead(Sensor_input);   //Read gas from the MQ2. The value will be saved as "gas_combustible"
  humedad_SCD = airSensor.getHumidity();        //Read humidity from the SC30. The value will be saved as "humedad_SCD"
  temperatura_SCD = airSensor.getTemperature(); //Read temperature from the SC30. The value will be saved as "temperatura_SCD"
  cO2 = airSensor.getCO2();                     //Read CO2 from the SC30. The value will be saved as "cO2"
  
  //Data is printed
  Serial.println("Humedad: "); 
  Serial.println(humedad);
  Serial.println("Temperatura: ");
  Serial.println(temperatura);
  Serial.println("Gas Sensor: ");  
  Serial.print(gas_combustible);   
  Serial.print("\t");
  Serial.print("\t");
  
  //If MQ2's gas values are above 1500 ppm, there will be a warning indicating that there's too much gas
  if (gas_combustible > 1500) {    
    Serial.println("Gas");  
  }
  else {
    Serial.println("No Gas");
  }
  
  Serial.println(cO2);
  Serial.println(temperatura_SCD, 1);
  Serial.println(humedad_SCD, 1);
  Serial.println("Waiting for new data");
  
  //If the CO2 SC30's levels are above 1000 ppm, the ozone generator will switch on
  if (cO2 > 1000)
  {
  pinMode(23, OUTPUT);
  digitalWrite(23, HIGH);
  }
  else {
  digitalWrite(23, LOW);   //Otherwise, it will be switched off
  }
  
  delay(500);
}

/*
  Since Humedad is READ_WRITE variable, onHumedadChange() is
  executed every time a new value is received from IoT Cloud.
*/
void onHumedadChange()  {
  // Add your code here to act upon Humedad change
}
/*
  Since Temperatura is READ_WRITE variable, onTemperaturaChange() is
  executed every time a new value is received from IoT Cloud.
*/
void onTemperaturaChange()  {
  // Add your code here to act upon Temperatura change
}
/*
  Since GasCombustible is READ_WRITE variable, onGasCombustibleChange() is
  executed every time a new value is received from IoT Cloud.
*/
void onGasCombustibleChange()  {
  // Add your code here to act upon GasCombustible change
}
/*
  Since HumedadSCD is READ_WRITE variable, onHumedadSCDChange() is
  executed every time a new value is received from IoT Cloud.
*/
void onHumedadSCDChange()  {
  // Add your code here to act upon HumedadSCD change
}
/*
  Since TemperaturaSCD is READ_WRITE variable, onTemperaturaSCDChange() is
  executed every time a new value is received from IoT Cloud.
*/
void onTemperaturaSCDChange()  {
  // Add your code here to act upon TemperaturaSCD change
}
/*
  Since CO2SCD is READ_WRITE variable, onCO2SCDChange() is
  executed every time a new value is received from IoT Cloud.
*/

void onCO2Change()  {
  // Add your code here to act upon CO2 change
}
