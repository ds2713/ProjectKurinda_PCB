# ProjectKurinda_PCB
This repository contains the PCB design data for the Project Kurinda landslide sensor.

## Introduction

The PCB has been specifically designed to be simple to understand for beginners, while also being extremely cheap to manufacture. All of the modules are basic Arduino development boards which can be soldered onto the base PCB by hand.

Over time, this PCB will be developed further to offer users the choice between using the Arduino development board sensors (for simplicity and educational purposes), or using the underlying components soldered directly onto the PCB to minimise costs further and shrink the sensor.

## Architecture

As with all designs like this, there are many options to choose from with various advantages and disadvantages. The following components were chosen for this version for a variety of reasons including cost, availability, interface simplicity, and additional features. Most sensors could be replaced with similar components with minimal rework.

### Microcontroller

The landslide sensor is based around the ESP32-C3 Supermini microcontroller development board. These boards provide a reasonable number of GPIO ports, analogue inputs, as well as i2c and SPI connections for some of the sensors. They also provide WiFi connectivity natively, which is used in this version for development and transmitting sensor readings (but will be superceded by LoRa technologies in the future).

The software for these boards can be developed within the Arduino ecosystem, which provides a large support community and many users will already be familiar with. This will help the Project Kurinda ecosystem improve over time, as the community can contribute improvements.

These development boards can be purchased cheaply from many sources, which will hopefully help with the international uptake of this project. By keeping the design basic (for example using a two layer PCB), they should be easy to manufacture and assemble by hand.

## Sensors

The first version of the sensor (v1.0) used standard Arduino modules to measure the temperature, humidity, acceleration and soil moisture. Although these sensors worked sufficiently well for the first prototype, they all had various issues which made them undesirable for the new design. One of the biggest challenges was power consumption, with the sensors drawing current all the time even when not collecting readings. On the second version of the sensor (v1.1), the modules were replaced with components which draw much less power. Additionally, a simple switch was added to the sensor to turn off the 3.3 V rail to the sensors when they are not required (for example, when the sensor is asleep between readings). This massively reduces the power consumption of the system, and will allow it to either last longer on battery power, or use a smaller battery plus solar panel to save cost.

All of the sensors chosen (with the exception of the generic soil moisture probe) can be bought as individual integrated circuits (cheaper) or as ready-made breakout boards (easier to assembly by hand). This allows the same sensor architecture and software to be used across sensor designs, whether that is the larger sensor for manual assembly or the smaller sensor planned in the future for automated assembly.

### Temperature and humidity sensor

Originally, the DHT11 temperature and humidity sensor was chosen. However, this sensor has various limitations and is a relatively old component. For example, the sample rate is limited to around once per second, and the serial interface was fairly unreliable during development tests.

In the second version of the PCB, this sensor was replaced with the **SHT40** sensor from Sensirion. The sensor provides an i2c interface for retrieving the readings like the DHT11, but has no 1 Hz limit and has a much better temperature sensitivity.

| Parameter                         | Value               |
|-----------------------------------|---------------------|
| **Humidity**                      |                     |
| Typ. relative humidity accuracy   | 1.8 %RH             |
| Operating relative humidity range | 0 - 100 %RH         |
| Response time (τ63%)              | 4	s                 |
| Calibration certificate           | Factory calibration |
| **Temperature**                   |                     |
| Typ. temperature accuracy         | 0.2 °C              |
| Response time (τ63%)              | 2	s                 |

### Accelerometer

There are lots of accelerometer sensors available for Arduino projects. In the first version, the MPU6050 was used but this was replaced by the **LIS2DW12** from ST Microelectronics in the latest design. The largest improvement here is the low power mode of the accelerometer, which the datasheet lists as 1 µA in active low-power mode.

| Parameter                  | Value                                                         |
|----------------------------|---------------------------------------------------------------|
| Ultralow power consumption | 50 nA in power-down mode, below 1 µA in active low-power mode |
| Noise                      | Down to 1.3 mg RMS in low-power mode                          |
| Supply voltage             | 1.62 V to 3.6 V                                               |
| Acceleration range         | ±2g/±4g/±8g/±16g full scale                                   |

This accelerometer also provides two interrupt outputs, which will allow the accelerometer to alert the microcontroller if it detects motion. This will help the sensor to reduce its power consumption even further, because the microcontroller can be put into sleep mode and then awoken by the accelerometer.

### Real-Time Clock (RTC)

The sensors use the RTC to timestamp the sensor readings.

The RTC can also be configured to provide an interrupt to the microcontroller, at a chosen interval. This will be used to wake the microcontroller from  sleep to take a sensor reading. Turning off the sensors and putting the microcontroller into deep sleep mode allows the landlide sensor to minimise its power consumption.

The first prototype used a generic RTC module, but this was replaced on the latest version of the board by the **RV3028** RTC from Microcrystal. This device is fully integrated with the crystal, and is specifically designed to be ultra low power draw (~ 100 nA). This should allow the RTC to operate from a tiny battery for an extremely long time (many years).

| Parameter    | Value                                             |
|--------------|---------------------------------------------------|
| Current draw | ~100nA typical                                    |
| Accuracy     | ±1 second drift per million seconds at 25 degrees |

The accuracy of this RTC means that it should only drift by around 1 second every 11.5 days (around 31 seconds per year). This is important because the timekeeping will be used to tell the sensor when to transmit its readings on the wireless network.

### Soil Moisture Sensor

The soil moisture sensor is the only sensor which remained the same between versions of the board, partly because it is cheap and easy to use, and easily available. Better sensors are considerably more expensive or would be harder to interface with, so it was decided to keep the same sensor.

The soil moisture sensor is the component with the largest power draw, because it is based around a relatively old timing chip providing the high frequency signal to the capacitive probe. However, this sensor is only required for short periods of time to monitor the soil moisture when the microcontroller needs a reading, so it can be turned off for the rest of the time and only use power for a few seconds when the reading is taken.

These soil moisture sensors can be purchased from many places, but one example listing on eBay is provided below.

https://www.ebay.co.uk/itm/205646455998

### Micro SD card

The micro SD card is used to store the sensor readings for long duration. This isn't strictly necessary for the sensor if the wireless network is operating properly and the readings are transmitted quickly, but if the microcontroller loses power or can't transmit for some reason, the recent readings would be lost. The micro SD card allows the sensor to store the readings until either the wireless network is operating again, or someone can manually collect the data.

On the prototype sensor, the **DFR0229** micro SD module from DFRobot was used, but any breakout board should be adequate.

## Changes from v1.0 to v1.1

Apart from changing the sensor modules themselves, some extra improvements were made to the second version of the sensor (v1.1). These are listed below.

### Power switching

Instead of providing 3.3 V to all the sensors all the time, a small P-channel MOSFET switch was added to the PCB to allow the microcontroller to switch the power rail off when not required. Specifically, this switched rail now provides the power to the soil moisture and the temperature & humidity sensors, and the micro SD card. When the microcontroller wants to take a reading, it can enable this rail to turn on the sensors. After the readings have been collected, the rail can be turned off to save power.

The schematic for this power switch is shown below.

![Power switch circuit schematic](/Images/power_switch.png)
*Power switch on 3.3 V rail to selectively power sensors.*

### Battery voltage monitoring

It is useful for the microcontroller to be able to read its own battery voltage. In the simplest form, this can be achieved by using a potential divider to reduce the raw battery voltage and provide that to an analogue input on the microcontroller. Although this does add a small constant draw on the battery, this can be minimised by using very large resistor values to reduce the current at the expense of higher noise on the signal. However, this should be adequate for a very basic voltage reading to give an indication of the battery level.

![Battery voltage monitoring circuit schematic](/Images/battery_monitor.png)
*Potential divider on battery voltage rail.*

### Filtering on soil moisture sensor analogue input

The output of the soil moisture sensor is a simple analogue signal, which is connected to the microcontroller through the PCB. This signal should be filtered near the microcontroller input to reduce noise. A simple RC network is used to attentuate any high frequency noise. The component values can be adjusted if necessary, but the existing values give a corner frequency of ~160 Hz which should be a reasonable starting point.

### i2c pull-up resistor footprints

These footprints can be used if the i2c rail needs pulling up for reliable operation. In theory, the various breakout boards already provide these resistors so they shouldn't be necessary, but the footprints have been added to the PCB in case they are required in the future.

## Surface mount components

The battery voltage monitoring, power switching and analogue filtering all require surface mount components to be fitted to the PCB. However, this may be challenging in an educational setting or with users who are new to soldering. Therefore, simple solder jumpers have been added to the PCB to bypass the surface mount components if necessary.

By shorting out solder jumper JP1 on the PCB, the 3.3 V switched rail is directly fed from the 3.3 V main power rail. This means that the microcontroller cannot turn off the sensors to save power, but the sensors can be read by the microcontroller at any time. This should be convenient for educational settings.

Solder jumper JP2 connects the analogue output of the soil moisture sensor directly to the analogue input of the microcontroller (without the filter). This means the signal will be more susceptible to noise, but again it should be perfectly adequate for normal use.

![Solder jumper JP1 and JP2.](/Images/jumpers.png)
*Solder jumper JP1 to permanently power the 3.3 V switched rail, and JP2 to connect the analogue output from the soil moisture sensor to the analogue input on the microcontroller.*

## PCB design software

The PCB was designed in KiCAD 10.0.

If you wish to view or modify the design, please download KiCAD from https://www.kicad.org/

## PCB manufacture

This prototype was manufactured by Eurocircuits, who generously sponsored this project with funding for the PCBs. We would like to thank Eurocircuits for their support.

https://www.eurocircuits.com/

# Updated sensor modules

The new sensor modules listed above can be purchased on the following links.

1. RTC -> RV3028: https://thepihut.com/products/rv3028-real-time-clock-rtc-breakout
2. T&H -> SHT40: https://thepihut.com/products/adafruit-sensirion-sht40-temperature-humidity-sensor-stemma-qt-qwiic
3. Accelerometer -> LIS2DW12: https://thepihut.com/products/fermion-lis2dw12-triple-axis-accelerometer-sensor-breakout-16g
4. Micro SD card ->  DFR0229: https://thepihut.com/products/microsd-card-module-for-arduino

## Sources

Several of the footprints and models of the development modules have been drawn from online resources. These are listed below.

**Most of these modules have been superceded on the latest sensor design, but these resources have been left here for reference.**

1. ESP32-C3 Supermini:
- Schematic component, PCB footprint, 3D model: https://www.snapeda.com/parts/ESP32-C3%20SuperMini_TH/Espressif+Systems/view-part/?ref=snap

2. Water sensor:
- 3D model: https://grabcad.com/library/water-level-sensor-5
- 3D model: https://grabcad.com/library/capacitive-soil-moisture-sensor-v1-2-2

3. Temperature and humidity sensor:
- 3D model: https://grabcad.com/library/ky-015-dht11-1

4. Accelerometer:
- 3D model: https://grabcad.com/library/mpu6050-accelerometer-module-2

# Next Steps

The next version of the PCB will include the LoRa radio module, and associated support circuitry.