# ProjectKurinda_PCB
This repository contains the PCB design data for the Project Kurinda landslide sensor.

## Introduction

The PCB has been specifically designed to be simple to understand for beginners, while also being extremely cheap to manufacture. All of the modules are basic Arduino development boards which can be soldered onto the base PCB by hand.

Over time, this PCB will be developed further to offer users the choice between using the Arduino development board sensors (for simplicity and educational purposes), or using the underlying components soldered directly onto the PCB to minimise costs further and shrink the sensor.

## Architecture

As with all designs like this, there are many options to choose from with various advantages and disadvantages. The following components were chosen for this version for a variety of reasons including cost, availability, interface simplicity, and additional features. Most sensors could be replaced with similar components with minimal rework.

### Microcontroller

The landslide sensor is based around the ESP32-C3 Supermini microcontroller development board. These boards provide a reasonable number of GPIO ports, analogue inputs, as well as an I2C connection for some of the sensors. They also provide WiFi connectivity natively, which is used in this version for development and transmitting sensor readings (but may be superceded by LoRa technologies in the future).

The software for these boards can be developed within the Arduino ecosystem, which provides a large support community and many users will already be familiar with. This will help the Project Kurinda ecosystem improve over time, as the community can contribute improvements.

These development boards can be purchased cheaply from many sources, which will hopefully help with the international uptake of this project.

### Temperature and humidity sensor

For this version, the DHT11 temperature and humidity sensor has been chosen.

https://www.instructables.com/DHT11-Temperature-Humidity-Sensor-With-Arduino/

### Accelerometer

There are lots of accelerometer sensors available for Arduino projects, but many are

https://howtomechatronics.com/tutorials/arduino/arduino-and-mpu6050-accelerometer-and-gyroscope-tutorial/

### Real-Time Clock (RTC)

The sensors use the RTC to timestamp the sensor readings.

The RTC can also be configured to provide a pulse on the INT connection to the microcontroller, at a chosen interval. This will be used to wake the microcontroller from deep sleep to take a sensor reading. Turning off the sensors and putting the microcontroller into deep sleep mode allows the landlide sensor to minimise its power consumption.

## PCB design software

The PCB was designed in KiCAD 10.0.

If you wish to view or modify the design, please download KiCAD from https://www.kicad.org/

## PCB manufacture

This prototype was manufactured by Eurocircuits, who generously sponsored this project with funding for the PCBs. We would like to thank Eurocircuits for their support.

https://www.eurocircuits.com/

## Sources

Several of the footprints and models of the development modules have been drawn from online resources. These are listed below.

1. ESP32-C3 Supermini:
- Schematic component, PCB footprint, 3D model: https://www.snapeda.com/parts/ESP32-C3%20SuperMini_TH/Espressif+Systems/view-part/?ref=snap

2. Water sensor:
- 3D model: https://grabcad.com/library/water-level-sensor-5
- 3D model: https://grabcad.com/library/capacitive-soil-moisture-sensor-v1-2-2

3. Temperature and humidity sensor:
- 3D model: https://grabcad.com/library/ky-015-dht11-1

4. Accelerometer:
- 3D model: https://grabcad.com/library/mpu6050-accelerometer-module-2

# Improvements for next version

1. Add battery voltage monitoring to get and transmit battery status.
2. Flip soil moisture sensor side.
3. DHT11 sensor without carrier board?


# Updated sensor chips

1. RTC -> RV3028: https://thepihut.com/products/rv3028-real-time-clock-rtc-breakout
- Very accurate, extremely low power consumption
2. T&H -> SHT40: https://thepihut.com/products/adafruit-sensirion-sht40-temperature-humidity-sensor-stemma-qt-qwiic
- Accurate and faster sampling
3. Accelerometer -> ADXL362: https://www.ebay.co.uk/itm/388949393444
- Extremely low power, interrupt outputs to wake microcontroller on motion
4. Accelerometer option 2 (i2C) -> LIS2DW12: PiHub
- Extremely low power, interrupt outputs to wake microcontroller on motion
5. Still need to think about the soil moisture sensor, the problem with what we have now is that it uses a lot of power and is fairly low frequency, so susceptible to changes in soil conductivity too. Ideally we'd like one which is lower power and higher frequency. More research needed.


# Next steps

## New PCB

i2c pull-ups
Power rail switch
Battery voltage monitoring
Flip soil moisture sensor side
Add LoRa module