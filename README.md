# Smart Greenhouse Environment Control System

• Jiadong Song; Jiayong Hu; Jingwei Li; Yiqi Zhou

<iframe src="https://player.vimeo.com/video/367126470" width="600" height="300" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe>
<p><a href="https://vimeo.com/367126470">12740Final_Environmental_Contril</a> from <a href="https://vimeo.com/user90437655">jingwei</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

### [Progress Report4](progress_report_4_0.md)

### [Progress Report3](progress_report_3_0.md)

### [Progress Report2](progress_report_2.md)

### [Progress Report1](progress_report_1.md)

### [Progress Report0](progress_report.html)

## 1. Introduction

Our project is to use temperature and light intensity sensors to monitor and control the environmental factors in a greenhouse to make sure that they are in an appropriate range so the plants can grow fast with little human work.

### 1.1 Motivation

A greenhouse is a structure in which plants requiring regulated climatic conditions are grown. Many commercial greenhouses are high tech production facilities for plants. The greenhouses are filled with equipment including screening installation, heating, cooling, lighting, and may be controlled by a computer to optimize conditions for plant growth. Different techniques are then used to evaluate optimality-degrees and comfort ratio of greenhouse micro-climate (i.e., air temperature, relative humidity and vapor pressure deficit) in order to reduce production risk prior to cultivation of a specific crop.

![Greenhouse](greenhouse.jpg)

The greenhouse is one of the most important innovations to improve crop yield. Temperature and light intensity are two important environmental factors that affect the plants’ growth, which are also the factors we measure and control in this project. 

It is hard and time-consuming for people to monitor and control these factors manually. A lot of companies provide modern solutions for control system in greenhouse involving advanced equipment and machine learning methods. Our motivation behind this Smart Greenhouse project is to provide a simple and cheap solution for the greenhouse control system.


### 1.2 Goals

The goal of our system is to monitor the greenhouse environment and adjust the light and temperature automatically so that the plant is constantly absorbing light and in a convenient temperature range. Thus the crop yield can be maximized by ensuring the plant is always in a conducive growing environment while lowering the human effort required.

## 2. Methodology

### 2.1 Phenomena of Interest

The growth of the plant is mainly dependent on the temperature and light.

Light intensity is measured through illuminance, where lux are the units (equivalent to lumens per squared-meter). For reference, an office will normally have lux values between 320-500 lux.

Temperature is measured in unit of Celsius, where 0°C is the freezing point of water and 100°C is the boiling point of water.

### 2.2 Sensor(s) Used

We use DHT11 Temperature & Humidity Sensor Module and Photosensitive Light Sensor Module.

#### Temperature & Humidity Sensor Module

DHT11 Temperature-Humidity Sensor Module is an accurate and inexpensive sensor that includes an ADC to convert analog values of humidity and temperature. It comes with an 8-bit microcontroller and provides reliable output results.

Parameters:
* Power supply: 3.3 - 5 V
* Current: 2.5 mA max use of current during conversion (when data request)
* Humidity: 20 - 90 % ± 5 %
* Temperature: 0 - 50º ± 2 %
* Sampling rate: ≤ 1 Hz
![Temperature sensor](temperature.png)

#### Photosensitive Light Sensor Module

The Light sensor module is based on photodetector GL5528 to detect the light intensity of the environment. As the resistance of the sensor varies depending on the amount of light it is exposed to, the output voltage changes with the light intensity. This module outputs both analog and digital signals, which can be used to trigger other modules. Also, the potentiometer can be used to adjust the sensitivity of the digital output. The digital terminal output HIGH when the light intensity exceeds the value set by a potentiometer and vice versa. The outputs of analog terminal increase with the light intensity.

Parameters:
* Working Voltage: 5V
* Output Signal: Analog or Digital(0 or 1)
* PCB Size: 3.2*1.4cm
![Photoresistor sensor](light.png)

## 2.3 Signal Conditioning and Processing

* For the temperature part, we use two LEDs (Red/ Blue) as indicators of whether the temperature is above 28C or not. The data acquired by temperature sensor is sent to Raspberry Pi. When the red LED lights up, it means that the environment temperature in the greenhouse is ideal. If the blue LED lights up, it means that we need to warm up the greenhouse.

* For the light part, we use 5 LEDs as our controller. These LEDs will help to generate extra light if the environment is not bright enough. The control method behind is PID. We collect the light intensity every iteration and based on the target value, we can calculate the error of light intensity. Then, we sum up all errors in history as the integral of error and subtract this error with last error to get the differential of error. We multiply them (error, integral, differential) with three parameters (P, I, D) to get the control variable. When we have the control variable, we adjust the duty ratio of the PWM wave to the LED, and the LEDs will respond with different intensity of light. After carefully tuning these parameters, we found the most suitable values for them, and we recorded the video with the best parameters.

## 3. Experiments and Results

### 3.1 Temperature indicator system

### 3.2 Light control system

For the light compensation system, the light sensor measures light intensity every 0.1s. PID is used to control LEDs to provide extra light for the greenhouse system and keep the light intensity of the greenhouse at a stable value.

On sunny days, the external light is sufficient for the greenhouse and meets its needs. Therefore, the LEDs would not turn on.

On cloudy days, the external light is weakened and cannot meet the needs of the greenhouse. Hence, the LEDs would automatically turn on and gradually turn up to make up the required light intensity.

The LEDs would further enhance the brightness at night, that is, they would gradually turn up until the greenhouse has enough light intensity even though there is no external light at all. They would turn down again when the sun rises and turn off if light intensity meets the needs.

We integrated our system to the OpenChirp so we can visualize the light intensity and the lightness of the LEDs. As the figure shows below:

[OpenChip]()

## 4. Discussion

## 5. Code

## 6. Reference
1. [https://en.m.wikipedia.org/wiki/Greenhouse](https://en.m.wikipedia.org/wiki/Greenhouse)
2. [https://www.robotshop.com/en/dht11-temperature-humidity-sensor-module-dyn.html](https://www.robotshop.com/en/dht11-temperature-humidity-sensor-module-dyn.html)
3. [https://www.makerfabs.com/light-sensor-module.html](https://www.makerfabs.com/light-sensor-module.html)













