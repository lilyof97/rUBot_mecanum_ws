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

```
