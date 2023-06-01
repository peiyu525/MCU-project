---
layout: post
title: ESP32 ADC
author: [Richard Kuo]
category: [Lecture]
tags: [jekyll, ai]
---

Introduction to ADC functions, and Joystick, Thermistor, Photoresistor, Gas sensors (MQ-2, MQ-3, MQ-135), eNose.
---
## ADC (Analog Digital Converter)
![]()

**Vin** is input voltage for ADC to sample<br>

**Vref** is reference voltage for ADC to compare with Vin<br>

**3-bit ADC example:**
<img width="50%" height="50%" src="https://www.edn.com/wp-content/uploads/media-1050787-c0306-figure1.gif">

**12-bit ADC** : output is a 12-bit binary code, so its value = 0 ~ 4095<br>
![](https://github.com/rkuo2000/MCU-course/blob/main/images/ADC_12bit_table.png?raw=true)

---
### ADC architecture
**Direct-Conversion ADC**<br>
![](https://www.maximintegrated.com/content/dam/images/design/tech-docs/634/E33Fig01.gif)

**Successive-Approximation ADC**<br>
![](https://github.com/rkuo2000/MCU-course/blob/main/images/SA_ADC_block_diagram.png?raw=true)

**Sigma-Delta ADC**<br>
![](https://github.com/rkuo2000/MCU-course/blob/main/images/Sigma-Delta_ADC_block_diagram.png?raw=true)

---
## ESP32 ADC
[ESP32 ADC documentation](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/adc.html)<br>

### [Analog Input Tutorial](https://www.arduino.cc/en/Tutorial/BuiltInExamples/AnalogInput)
**Vout = Vin * R2 / (R1+R2)**

With a **potentiometer**
<table>
<tr>
<td><img src="https://www.arduino.cc/wiki/static/7dbfb4b4c090ba1bc52c2a779822b8f9/5a190/analoginoutserial1_bb.png"></td>
<td><img src="https://www.arduino.cc/wiki/static/39a5da6e06c51305fa0bb902f3cab1e3/5a190/analoginoutserial_sch.png"></td>
</tr>
</table>

With a **photoresistor**
<table>
<tr>
<td><img src="https://www.arduino.cc/wiki/static/bb8d0c184836ed4f8cabf71c3dc07ce9/5a190/PhotoCellA0.png"></td>
<td><img src="https://www.arduino.cc/wiki/static/794469252e6fbe8ab19959f4fe067fca/5a190/PhotoResistorA0_schem.png"></td>
</tr>
</table>

### NodeMCU-32S pinout
![](https://github.com/rkuo2000/MCU-course/blob/main/images/NodeMCU-32S_pinout.jpg?raw=true)

---
### Sketch> ESP32> ESP32_ADC
* Connect NodeMCU-32S with a NTC-Thermistor and a 10K ohm to GND
![](https://github.com/rkuo2000/MCU-course/blob/main/images/Example_Thermistor.jpg?raw=true)
* Download [ESP32_ADC.ino](https://github.com/rkuo2000/arduino/blob/master/examples/ESP32_ADC/ESP32_ADC.ino)
to ~/Documents/Arduino/examples/
![](https://github.com/rkuo2000/MCU-course/blob/main/images/Sketch_ESP32_ADC.png?raw=true)
* Verify, then Tools>Serial Monitor>
![](https://github.com/rkuo2000/MCU-course/blob/main/images/Sketch_ESP32_ADC_monitor.png?raw=true)

### Sketch> ESP32 > ESP32_ADC_Joystick
![](https://github.com/rkuo2000/MCU-course/blob/main/images/Example_ESP32_ADC_Joystick.jpg?raw=true)

[ESP32_ADC_Joystick.ino](https://github.com/rkuo2000/Arduino/blob/master/examples/ESP32/ESP32_ADC_Joystick/ESP32_ADC_Joystick.ino)<br>
![](https://github.com/rkuo2000/MCU-course/blob/main/images/Sketch_ESP32_ADC_Joystick.png?raw=true)

---
## Examples> 03.Analog> 
* **AnalogInOutSerial** - Read an analog input pin, map the result, and then use that data to dim or brighten an LED.

* **AnalogWriteMega** - Fade 12 LEDs on and off, one by one, using an Arduino Mega board.

* **Calibration** - Define a maximum and minimum for expected analog sensor values.

* **Fading** - Use an analog output (PWM pin) to fade an LED.

* **Smoothing** - Smooth multiple readings of an analog input.

---
### Examples> 03.Analog> AnalogInput
* Connect NodeMCU-32S with a NTC-Thermistor and a 10K ohm pulldown.
![](https://github.com/rkuo2000/MCU-course/blob/main/images/Example_Thermistor.jpg?raw=true)
* AnalogInput.ino read sensor value as delay to turn LED on & off
* To show sensorValue
  - setup() to add `Serial.begin(9600);
  - loop() to add `Serial.println(sensorValue);`
![](https://github.com/rkuo2000/MCU-course/blob/main/images/Examples_AnalogInput.png?raw=true)
* Verify, then Tools>Serial Monitor>
![](https://github.com/rkuo2000/MCU-course/blob/main/images/Examples_AnalogInput_monitor.png?raw=true)

---
### Examples> 03.Analog> AnalogInOutSerial
* Connect NodeMCU-32S with a NTC Thermistor and a 10K ohm pulldown.
![](https://github.com/rkuo2000/MCU-course/blob/main/images/Example_Thermistor.png?raw=true)

* Modify AnalogInOutSerial.ino
  - analogOutPin = 2
  - replace analogWrite to ledcWrite (for ESP32 PWM)
  
```
const byte ledPin = 2;  // for ESP32 built-in LED

// setting PWM properties
const int freq = 5000;
const int ledChannel = 0;
const int resolution = 10; //Resolution 8, 10, 12, 15

ledcSetup(ledChannel, freq, resolution); // configure LED PWM functionalitites
ledcAttachPin(analogOutPin, ledChannel); // attach the channel to the GPIO2 to be controlled

ledcWrite(ledChannel, 102); //PWM Value varries from 0 to 1023 
```
* Compile
![](https://github.com/rkuo2000/MCU-course/blob/main/images/Examples_AnalogInOutSerial.png?raw=true)

* Verify, then Tools>Serial Monitor>
![](https://github.com/rkuo2000/MCU-course/blob/main/images/Examples_AnalogInOutSerial_monitor.png?raw=true)

---
## NTC Thermistor
**N**egative **T**emperature **C**oefficient = Rising Temperature will decrease Resistance
NTC thermistors can also be characterised with the B (or β) parameter equation,
![](https://wikimedia.org/api/rest_v1/media/math/render/svg/da9073f8178893fd670e3019816a07f594d0151f)

where the temperatures and the B parameter are in kelvins, and R0 is the resistance at temperature T0 (25 °C = 298.15 K).<br>
Solving for R yields<br>
![](https://wikimedia.org/api/rest_v1/media/math/render/svg/4592cdcd28a2cfc87bc70ef976024cfbc6a0415d)

**取log**<br>

log(R) = log(R0) + B * (1/T – 1/T0)

1/T = 1/T0 + (log(R) – log(R0)) / B

**T = 1 / (1/T0 + (log(R) – log(R0)) /B)**

---
### [Homework] Thermistor.ino
* write above formula in C++
* create Thermistor.ino with the following codes

**Read ADC** (get ADCvalue)<br>
```
int16_t ADCvalue;

void setup(){
  Serial.begin(9600);
}

void loop() {
  ADCvalue = analogRead(A0);
  Thermistor(ADCvalue);
}
```

**Read Thermistor** (use ADCvalue to calculate R & T)
```
void Thermistor(int16_t ADCvalue)
{
  double T, Temp;
  double T0 = 301.15;  // 273.15 + 28 (room temperature) 室溫換成絕對溫度
  double lnR;
  int16_t R;          // Thermistor resistence 
  int16_t R0 = 8805;  // calibrated by reading R at room temperature (=28 degree Celsius )
  int16_t B  = 3950;
  int16_t Pullup = 9930; // 10K ohm
	
  // R / (Pullup + R)   = adc / 4096
  R = (Pullup * ADCvalue) / (4096 - ADCvalue);
	
  // B = (log(R) - log(R0)) / (1/T - 1/T0) 
  T = 1 / (1/T0 + (log(R)-log(R0)) / B );
  Temp = T - 273.15;	
		
  Serial.println(Temp);
}
```

---
### [Homework] Thermistor_accuracy.ino
* Create Thermistor_accuracy.ino
* Verify on NodeMCU-32S and, compare it to Thermistor.ino output

**From Sketch/ESP32_ADC.ino**

`V = ReadVoltage(A0); //accuracy improved`

```
double ReadVoltage(byte pin){
  double reading = analogRead(pin); // Reference voltage is 3v3 so maximum reading is 3v3 = 4095 in range 0 to 4095
  if(reading < 1 || reading > 4095) return 0;
  // return -0.000000000009824 * pow(reading,3) + 0.000000016557283 * pow(reading,2) + 0.000854596860691 * reading + 0.065440348345433;
  return -0.000000000000016 * pow(reading,4) + 0.000000000118171 * pow(reading,3)- 0.000000301211691 * pow(reading,2)+ 0.001109019271794 * reading + 0.034143524634089;
} // Added an improved polynomial, use either, comment out as required
```

Then,<br>

V / 5 = R / (10K + R)<br>

V = 5 * R / (10000+R)<br>

V * (10000 + R) = 5 * R<br>

10000 * V = 5 * R - V * R<br>

10000 * V = (5 - V) * R<br>

R = 10000 * V / (5-V) <br>

```
V = ReadVoltage(A0); // ADC accuracy improved for ESP32
R = 9990 * V / (5 - V); // assuming 9990 is the measured resistance of 10K resistor by a multi-meter.
T = 1 / (1/T0 + (log(R)-log(R0)) / B ); // R0=8805 measured in room tempature at 28 celsius degree.
Temp = T - 273.15;

Serial.println(Temp);
```

---
## Photoresistor
![](https://github.com/rkuo2000/MCU-course/blob/main/images/Photoresistor_GL55_series_spec.png?raw=true)

### LDR arduino library
[Photocell (Light Dependent Resistor) Library for Arduino](https://github.com/QuentinCG/Arduino-Light-Dependent-Resistor-Library)

---
### Sketchbook> GL5516>
![](https://github.com/rkuo2000/MCU-course/blob/main/images/Example_GL5516.jpg?raw=true)

* Download files from github.com/rkuo2000/arduino/examples/GL5516
  - GL5516.ino
  - LightDependentResistor.cpp
  - LightDependentResistor.h
  
![](https://github.com/rkuo2000/MCU-course/blob/main/images/Sketch_GL5516.png?raw=true)

* Verify with NodeMCU-32S, and open Serial Monitor on ArduinoIDE
![](https://github.com/rkuo2000/MCU-course/blob/main/images/Sketch_GL5516_monitor.png?raw=true)

---
## [MQ Gas sensors](https://playground.arduino.cc/Main/MQGasSensors/)
![](https://github.com/rkuo2000/MCU-course/blob/main/images/MQ_gas_sensors.png?raw=true)
<br>
![](https://playground.arduino.cc/uploads/Main/alchoolau5/index.jpg)

---
### MQ2 300~10000ppm 煙幕感應器 (液化氧、丁烷、丙烷、甲烷、酒精、氫氣)<br>
![](https://www.taiwaniot.com.tw/wp-content/uploads/2016/05/1521002-1-600x402.jpg)
接線方式：VCC:接電源正極（5V）、GND:接電源負極、DO:TTL開關信號輸出、AO:模擬信號輸出<br>
注意：感測器通電後，需要預熱20秒左右，測量的數據才穩定，感測器發熱屬於正常現象，因為內部有電熱絲，如果燙手就不正常了<br>

[MQ2 datasheet](https://www.mouser.com/datasheet/2/321/605-00008-MQ-2-Datasheet-370464.pdf)<br>
![](https://github.com/rkuo2000/MCU-course/blob/main/images/MQ2_chart.png?raw=true)

---
### MQ3 酒精蒸氣,乙醇,氣體檢測模組
![](https://www.taiwaniot.com.tw/wp-content/uploads/2016/05/51Ic6UAJ2JL.jpg)
[MQ3 datasheet](https://www.sparkfun.com/datasheets/Sensors/MQ-3.pdf)<br>
![](https://github.com/rkuo2000/MCU-course/blob/main/images/MQ3_chart.png?raw=true)
* The typical sensitivity characteristics of the MQ-3 for several gases. in their: Temp: 20°C、 Humidity: 65%、 O2 concentration 21% RL=200kΩ
  - Ro: sensor resistance at 0.4mg/L of Alcohol in the clean air. 
  - Rs:sensor resistance at various concentrations of gases.

---
### MQ-135 空氣品質檢測 有害氣體感測器模組
用於建築物/辦公室的空氣質量控制設備，適用於檢測NH3，NOx，酒精，苯，煙，CO2等。
![](https://www.taiwansensor.com.tw/wp-content/uploads/2018/03/SNA-002110-600x600.jpg)

---
### Sketch> MQ3>
* NodeMCU-32 GPIO36(A0) is connected to MQ3 sensor AO pin
<table>
<tr>
<td><img src="https://github.com/rkuo2000/MCU-course/blob/main/images/Example_MQ2.jpg?raw=true"></td>
<td><img src="https://github.com/rkuo2000/MCU-course/blob/main/images/Example_MQ3.jpg?raw=true"></td>
</tr>
</table>

* MQ3 Vcc connected to 5V, required 15 minutes burn-in
![](https://github.com/rkuo2000/MCU-course/blob/main/images/Sketch_MQ3.png?raw=true)

* Open Tools>Serial-Monitor>
![](https://github.com/rkuo2000/MCU-course/blob/main/images/Sketch_MQ3_monitor.png?raw=true)

* Open Tools>Serial-Plotter> to see waveform 
![](https://github.com/rkuo2000/MCU-course/blob/main/images/Sketch_MQ3_plotter.png?raw=true)

* 3 test cases: Fresh Air, Your Breathe, Marker are applied to MQ3 sensor
![](https://github.com/rkuo2000/MCU-course/blob/main/images/Sketch_MQ3_waveforms.png?raw=true)
  
---
## Electronic Nose

### Detection of Fruits in Warehouse
**Paper:** [Detection of fruits in warehouse using Electronic nose](https://www.matec-conferences.org/articles/matecconf/pdf/2018/91/matecconf_eitce2018_04035.pdf)
![](https://github.com/rkuo2000/MCU-course/blob/main/images/Detection_of_fruits%20in_warehouse_using_Electronic_nose.png?raw=true)

---
### Estimating Gas Concentration
**Paper:** [Estimating Gas Concentration using Artificial Neural Network for Electronic Nose](http://repository.ubaya.ac.id/37946/1/1-s2.0-S1877050917329137-main.pdf)
![](https://github.com/rkuo2000/MCU-course/blob/main/images/Respon_of_Alcohol_Gas_Content_on_Sensor_MQ3.png?raw=true)


<br>
<br>

*This site was last updated {{ site.time | date: "%B %d, %Y" }}.*


