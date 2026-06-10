# 7 OFFGRID INVERTER 
The offgrid inverter manufactured by Yuco provides a local 230Vac voltage source with a rated output power of 1000 W. The output waveform is pure sinewave. 

## 7.1 ELECTRICAL CHARACTERISTICS 

### 7.1.1 YK-1000-48

| Parameter             | Symbol            | Min | Typ  | Max  | Unit |
| --------------------- | ----------------- | --- | ---- | ---- | ---- |
| Input Voltage         | V<sub>in</sub>    | 40  | 48   | 60   | V    |
| Output Power          | P<sub>rated</sub> | 0   | 1000 | 2000 | W    |
| Output Voltage        | V<sub>out</sub>   | 209 | 220  | 231  | V    |
| Frequency             | f<sub>AC</sub>    |     | 50   |      | Hz   |
| Idle Current          | I<sub>Idle</sub>  |     | 0.6  |      | A    |
| Operating Temperature | T<sub>op</sub>    |     | 80   |      | °C   |
| Efficiency            | 𝜂                |     | 90   |      | %    |


## 7.2 COMMUNICATION
Because of the missing communication channel of the Offgrid, there is no communication between the Inverter and the PCM. 

## 7.3 FEATURES 
### 7.3.1 On/Off Control
The inverter turns on or off when the two wires, which are normally connected to manual switch, are shorted together. This essentially connects the 48V from the input to the supply of the inverter logic which in turn start to operate. 
The PCM is responsible for turning the inverter on or off in dependence on the current state of the CAM switch and the current operation status. 
<div class="page-break" style="page-break-before: always;"></div>

### 7.3.2 Current Protection
Additional to the inbuild protection mechanism of the inverter, the PCM also checks for malfunctions or power limits. 
When the input current of the inverter exceeds the 30A limit per battery, the inverter will get turned off and the UI shows a corresponding warning code.
To reset this warning, the CAM switch must be changed to Standby which clears the warning on the UI and allows the Offgrid inverter to be turned back on again.

### 7.3.3 FAN Control 
The FANs of the Inverter are controlled by the inverter itself and are not controlled otherwise in any way. 

### 7.3.4 Inductive Loads
Special care must be taken when using the inverter with inductive loads such as motors or fridges. The high inrush current at start-up can trip the inverters current protection or prevent the inverter from starting up properly even though the continuous power rating of the device is well below the continuous output power of the inverter.
In general, always start up the inverter first and connect the load afterwards