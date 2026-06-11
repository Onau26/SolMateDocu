
# MAXIMUM POWER POINT TRACKER

The MPPT in the SolMate system is used for power conversion and steps up the DC-input voltage of the PV panels to a higher voltage needed to charge a battery. The new MPPT consists of two individual inputs with two phase each, leading to a total of 4 phases. Each phase consists of one synchronous boost converter capable of an output power of around 300W. For switching, Gallium Nitride High Electron Mobility Transistors (GaN HEMTs) are used which allow to increase the power density while maintaining industry leading efficiency.

![MPPT](Docu/Images/MPPT_PCB.png) 
## 2M.1 ELECTRICAL CHARACTERISTICS 

| Parameter                 | Symbol                 | Min | Typ | Max       | Unit |     |
| :------------------------ | :--------------------- | :-- | :-- | :-------- | :--- | --- |
| Input Voltage             | V<sub>in</sub>         | 6   | 32  | 65 <sup>1 | V    |     |
| Output Voltage            | V<sub>out</sub>        |     | 48  | 60        | V    |     |
| Input Current (per Phase) | I<sub>in</sub> (Phase) | 0   |     | 20        | A    |     |
| Input Current (total)     | I<sub>ln</sub> max     | 0   |     | 40 <sup>2 | A    |     |
| Output Current            | I<sub>out</sub>        | 0   |     | 28        | A    |     |
| Operating Temperature     | T<sub>op</sub>         | -20 |     | 100       | °C   |     |
| Efficiency                | 𝜂                     |     |     | 98        | %    |     |
| Switching Frequency       | fsw                    |     | 500 |           | kHz  |     |
| FAN Voltage               | V<sub>FAN</sub>        |     | 5   |           | V    |     |
<sup>1</sup> Maximum voltage which can be handled by Hardware, the overall input voltage must not exceed battery voltage.  
<sup>2</sup> May be limited due to thermal limitations.

<div class="page-break" style="page-break-before: always;"></div>

## 2.2 COMMUNICATION 

For the physical layer RS485 is used. On top of this Layer Modbus RTU is implemented which is a protocol which has been standardized for industrial applications. Communication speed is 9600 kbit/s, the default address of the MPPT is 0x05. The following parameters can be written/read using this communication.

| Parameter          | Unit | Description                                |
| ------------------ | ---- | ------------------------------------------ |
| Status             |      | Indicates the status                       |
| Error              |      | Indicates errors                           |
| Temperature (x4)   | °C   | emperature of each individual power stage  |
| Output Voltage     | V    | Voltage measured at the output of the MPPT |
| Input Voltage (x2) | V    | Voltage measured at each input             |
| Input Current (x2) | A    | Current measured at each input             |
| Output Current     | A    | Current measured at the output             |
| Fan Speed          | rpm  | Speed of the FAN                           |
 
The Status and Error fields are containing bits which give more information about the system. 

### 2.2.1 Status Byte Bit Description

| Bit |                          | Description                                                                      |
| --- | ------------------------ | -------------------------------------------------------------------------------- |
| 0   | Enabled                  | The MPPT has been disabled by Software (Active Low)                              |
| 1   | Tracking                 | Maximum Power Point Tracking is enabled (Active High)                            |
| 2   |                          | Not implemented                                                                  |
| 3   | Battery Full             | Charging has been stopped since the battery has reached its charging end voltage |
| 4   | Smart Charging           | The MPPT has reduced charge current since the battery is reaching 100%           |
| 5   | Low Battery Derating     | he stack voltage of the battery is very low → charging current is reduced        |
| 6   | Low Temperature Derating | The temperature of the battery is very low → charging current is reduced         |
<div class="page-break" style="page-break-before: always;"></div>

### 2.2.2 Error Byte Bit Description

| Bit |                   | Description                                              |
| --- | ----------------- | -------------------------------------------------------- |
| 0   | Communication     | A communication error occurred                           |
| 1   | Temperature Limit | The maximum allowed temperature rating has been exceeded |
| 2   | Overvoltage       | The maximum allowed input voltage has been exceeded      |
| 3   | Overcurrent       | The maximum allowed input current has been exceeded      |
| 4   | Reverse Polarity  | The input polarity is in reverse                         |

**Note:** Every error condition leads to the MPP shutting down and halting operation until error condition is removed and the system has been restarted. 

## 2.3 FEATURE DESCRIPTION
For correct operation, several features have been implemented. 

### 2.3.1 Multiple Inputs
The MPPT has two separate inputs which allows to connect two separate strings of PV panels. Each input supports up to 20 A which leads to a total input power of 600 W per input or 1200 W in total. To use the MPPT as efficient as possible, both inputs should be balanced which means that an equal number of panels are connect to each input. Possible configurations are therefore 1x1, 2x2 or 3x3 panels. Since each input is limited to 20 A, overloading an input is not possible but efficiency will decrease if the input is not properly balanced.

### 2.3.2 MPP Tracking 
The MPP tries to find the maximum power point at the DC-input side by constantly varying the input current and observing the resulting input power. The algorithm is shown below. 

![[MPPT_Tracking.png|518]]


For most of the PV panels, the maximum power point is around 31V.
When a power supply is connected at the input instead of a PV module, the MPP will steadily increase the input power until the maximum input current of 20A per phase is reached. 

**Please Note:** 
This algorithm only searches the local maximum power point. In case of partial shadowing, the global maximum power point may be different from the local maximum power point.

### 2.3.3 Temperature Limit
To protect the MPPT against thermal overload, input current foldback has been implemented. If the temperature of the MPPT reaches 90°C, the input power will be reduced to stay under this limit. When there is no malfunction (like a shorted FET), the MPP will never reach a temperature above 100°C since the input current would be reduced to 0A beforehand. If the temperature exceeds 100°C, the MPPT will shut down and an error warning will be displayed. 

### 2.3.4 Overcurrent Protection 
Each individual phase has its own overcurrent protection implemented in hardware. This turns off the MPPT in case of shorts at the input or the output and protects the MPPT from damage.
<div class="page-break" style="page-break-before: always;"></div>

### 2.3.5 Fan Control 
The MPPT has the control over its own fan, can set the speed via PWM as well as measure the speed. The fans will be controlled in dependence of the temperature of the device.

### 2.3.6 Charging Strategy 
When the output voltage of the MPPT (i.e. the battery voltage) reaches the charging end voltage (which by default is 52.5V), the converter enters into constant voltage charging mode where the output voltage is kept constant and the current will adjust accordingly<sup>3</sup> .
Additionally, the PCM can set the “smart charging” bit on the MPP which reduces the constant voltage charging threshold to 51.2V. This bit will be set when the battery is getting full i.e. the SOC is greater than 90% or the “cell overvoltage” bit is set by the BMS. This mode is maintained until the SOC is below 85% or the “cell overvoltage” bit has been de-asserted.
The battery is interpreted as full and the MPPT is disabled when the output current is smaller than 0.5A and the output voltage is greater than 52V. The full charging cycle is shown below. 

<sup>3</sup> The minimum output current is 0.2A 

![[Charging_Strategy.png|621]]

The battery full state is exited according to this state diagram. 
This charging strategy gives the cells more time to balance when the battery is nearly full and prevents the battery from being overcharged. Additionally, there is a small hysteresis when the battery is full and stops charging before the MPPT resumes normal operation. 
In case the battery voltage is low, charging current will be reduced until the battery voltage has risen to a save level again. This is done by linearly increasing the output current from 0.1C (41 V) to 0.5C (46 V). In low temperature environments, charging current is reduced to 0.1C to avoid crystallization of lithium ions. 

### 2.3.7 Power Supply 
The MPPT will be supplied by the output voltage of the converter or the PCM in case the battery has shut down and mains is connected to the PCM. 
The source of higher priority is the converters output voltage, when this voltage is below 6V, the Gate drivers of the power stages will get turned off and the supply logic automatically switches to the 3.3V supply provided by the PCM. This allows to keep the communication between MPP and PCM active even when the battery and PV voltages are 0V.

### 2.3.8 Input Overvoltage 
Detection Input overvoltage can occur when PV panels are connected in series, exceeding the maximum allowed voltage of the system. In general, the input voltage must not exceed the battery voltage, otherwise current starts to flow through the body diodes of the transistors, charging the battery uncontrollable.
This can cause damage to the battery or, in case the battery shuts down, to the complete system. 
The MPPT detects input overvoltage events when the input voltage exceeds the output voltage and is greater than 50V while an output current to the battery can be measured. In this case, an error bit will be set and the UI displays an error code.

### 2.3.9 Input Reverse Polarity 
If PV panels are connected in reverse polarity, current starts to flow through the body diode of the low-side transistors. This causes excessive power loss and results in thermal damage to the system. 
Reverse polarity is detected by checking if the measured input voltage is exactly 0V<sup>4</sup> and the temperature rises above 70°C. In this case, an error bit will be set and the UI issues an error code.
Additionally, the low-side MOSFET is turned-on completely which shorts the input and reduces power loss, thereby protecting the MPPT from damage. 
The MPPT recovers from this state when the PV input connector is disconnected and connected again.

<sup>4</sup>Minimum voltage which can be measured by the system, real voltage has negative sign
