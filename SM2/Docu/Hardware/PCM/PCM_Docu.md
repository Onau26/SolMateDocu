# 4 POWER CONTROL MODULE 
The Power Control Module is the main component in the new SolMate system and replaces the netD PCB used in previous versions.
The PCM is responsible for controlling all components present in the system including MPPT, UI, Ongrid, Offgrid and the Battery as well as communication with the Raspberry Pi. The netD technology is now also embedded onto this PCB. 

![[PCM PCB.png|508]]
## 4.1 ELECTRICAL CHARACTERISTICS 

| Parameter                  | Symbol               | Min | Typ | Max  | Unit |
| -------------------------- | -------------------- | --- | --- | ---- | ---- |
| Input Voltage DC           | V<sub>in(DC)</sub>   | 5   | 48  | 60   | V    |
| Input Voltage AC           | V<sub>in(AC)</sub>   | 85  | 230 | 264  | V    |
| Battery Current Internal   | I<sub>Bat(int)</sub> | 0   |     | 20   | A    |
| Battery Current External   | I<sub>Bat(ext)</sub> | 0   |     | 20   | A    |
| Current of Common Terminal | I<sub>Com</sub>      | 0   |     | 40   | A    |
| Ongrid Power Sense         | P<sub>ongrid</sub>   | 0   |     | 1600 | W    |
| FAN Voltage                | V<sub>FAN</sub>      |     | 5   |      | V    |
<div class="page-break" style="page-break-before: always;"></div>

## 4.2 COMMUNICATION
For the physical layer RS485 is used. On top of this Layer Modbus RTU is implemented which is a protocol which has been standardized for industrial applications.
Communication speed is 9600 kbit/s, the PCM acts as the master and is responsible for starting transmissions.
The following parameters can be written/read using this communication. 

| Parameter      | Unit | Description                            |
| -------------- | ---- | -------------------------------------- |
| Status         |      | Indicates the status                   |
| Error          |      | Indicates errors                       |
| Injection Mode |      | Current injection mode selected by CAM |
| Fan 1          | RPM  | rpm Speed of Fan 1                     |
| Fan 2          | RPM  | rpm Speed of Fan 2                     |

The Status and Error fields are containing bits which give more information about the system. 
<div class="page-break" style="page-break-before: always;"></div>

### 4.2.1 Status

| Bit | Short Description | Description                                                                                                                                       |
| --- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0   | Running           | System is active and running                                                                                                                      |
| 1   | Shutting Down     | The system is performing the shutdown sequence                                                                                                    |
| 2   | Starting Up       | The system is performing the starting sequence                                                                                                    |
| 3   | FW Update         | Ready The Raspberry Pi has issued a Firmware Update request (All data will be saved to the EEPROM and all batteries/inverter will get turned off) |
| 4   | Battery Pairing   | Battery paring of internal and external battery pack is enabled                                                                                   |
| 5   | Not implemented   | Not implemented                                                                                                                                   |
| 6   | Not implemented   | Not implemented                                                                                                                                   |
| 7   | Not implemented   | Not implemented                                                                                                                                   |
| 8   | Not implemented   | Not implemented                                                                                                                                   |
| 9   | Battery Low       | Battery SOC is low → Inverters have been disabled                                                                                                 |
| 10  | Min Charge Limit  | The minimum SOC has been reached → Inverters have been disabled                                                                                   |
### 4.2.2 Error

| Bit | Short Description      | Description                                                                 |
| --- | ---------------------- | --------------------------------------------------------------------------- |
| 0   | Communication          | A communication error occurred                                              |
| 1   | FAN 1 Malfunction      | FAN 1 is turned on but not spinning                                         |
| 2   | FAN 2 Malfunction      | FAN 2 is turned on but not spinning                                         |
| 3   | Current Limit Internal | The maximum allowed current for the internal battery pack has been exceeded |
| 4   | ADC Malfunction        | The 16bit Sigma-Delta ADC is not sampling correctly                         |
| 5   | EEPROM                 | Communication with EEPROM can’t be established                              |
<div class="page-break" style="page-break-before: always;"></div>

## 4.3 FEATURE DESCRIPTION
### 4.3.1 Compute Module 4
Instead of the previously used Raspberry Pi 3B+, the main compute unit is now the Raspberry Pi Compute Module 4 (CM4). This module includes onboard eMMC memory which has a much longer lifetime than standard SD cards which avoids data loss or corruption and therefore increases the longevity of the SolMate. Like the Raspberry Pi 3B+, the Compute Module 4 features onboard WiFi including an onboard antenna. Additionally the CM4 also has the ability to use an external antenna which is connected through a standard U.FL connector which will be used in the SolMate 2.0 to increase the communication range. 
The CM4 has less overall power consumption than the 3B+ however, because of the smaller form factor there is less area to dissipate heat which leads to the CM4 becoming hotter than the 3B+.
For more detailed information about the CM4, please refer to the datasheet: Raspberry Pi Compute Module 4 Datasheet 
There is also a USB connection present which is directly connected to the USB controller of the CM4. Additional to the required EMI protection at the input of the USB connector, there is also a current monitoring IC which protects the 5V supply from overcurrent or reverse current  which could damage the system or result in undefined behaviour. Therefore, the PCM PCB cannot be powered from an external power supply via the USB Port! However, external USB devices like Smartphones can be charged using this USB port.

### 4.3.2 Real Time Clock (RTC)
For logging purpose, having the correct time of day is important. To ensure the system time is valid even when the Raspberry Pi is not connected to the internet, a RTC has been added which is backed up by a CR2032 coin cell battery. This backup supply should be able to power the RTC in case the main supply is removed for at least 10 years (limiting factor is the selfdischarge rate). 
The 32.768 kHz crystal is a standard clock crystal with an accuracy of +/- 20ppm which results in a time error of 1.73 seconds per day or 51 minutes per month. 

### 4.3.3 EEPROM 
An onboard 16k EEPROM is used to store data like operating status, battery charge level, inverter total power and verakey. Data will be written from the flash memory of the PIC MCU to the EPPROM when the 5V supply rail drops below 4.5V and will be read back during the starting sequence of the MCU. 
<div class="page-break" style="page-break-before: always;"></div>

### 4.3.4 FAN Control 
The PCM is able to control two fans. While both fans will have the same target speed, the actual speed can be read separately for both fans and is used to detect malfunctions.
Fan speed is controlled as a function of the hotspot temperature found on PCM, MPP or Inverter. Speed is increased linearly from 0% (50°C) to 100% (70°C). 

### ~~4.3.5 Battery Pairing~~
~~The PCM supports up to two different Battery where battery 1 is the internal battery and battery 2 is the external battery. When connecting two batteries with different capacities and different charging levels, it is important to balance the voltage of both batteries before connecting to avoid excessive balancing currents which can lead to potential damage to the battery. To avoid this problem, the PCM is able to switch between batteries and balance each battery to the same voltage before connecting both in parallel. When the external battery is connected, the inrush current limiting resistor is enabled which allows the external battery pack to wake from sleep. Afterwards, the voltage of both packs will be compared and the PCM switches to the battery with the lower voltage. During this time, the Ongrid and Offgrid inverter will be turned off and the MPP current will be limited to 0.5A to avoid voltage jumps. As soon as both battery voltages are within +/- 200mV, both batteries will be connected in parallel.~~
### 4.3.6 Temperature Sensors
The PCM PCB has two onboard temperature sensors which sense the temperatures of the MOSFETs responsible for switching the batteries. 

### 4.3.7 Battery Current/Voltage/Charge
Readings The PCM has an onboard 16bit Sigma-Delta ADC which constantly monitors the batteries currents and voltages. The batteries charge is also calculated internally and will be reset to 0 when the inverters will get turned off by the PCM in case of a low battery event. 

### 4.3.8 CAM Sensing
Unlike in previous versions, the CAM switch is now only responsible for switching the external ac plug between the Offgrid inverter and the Wieland connector at the back. 
The position of the CAM switch will be sensed by the three wires which are only signal wires and don’t commutate any significant current. The PCM detects the current switch position and enables or disables the inverters accordingly. 
**Please Note:** The position of the CAM switch only selects the injection mode and doesn’t switch the inverters directly. The power management implemented on the PCM decides if the inverter can be switched on according to the charging state of the battery. 
<div class="page-break" style="page-break-before: always;"></div>

### 4.3.9 Offgrid Inverter Switch
The offgrid inverter will be turned on/off by shorting the two wires which are normally connected to a DIP switch. This essentially connects the 48V battery voltage to the logic supply of the inverter which results in the inverter powering on. 

### 4.3.10 Power Supply
The Power Control Module features a redundant power supply which means the PCB can be powered either by the Meanwell AC/DC power supply or by the 48V DC supply. Switching between these two supplies is handled by a dedicated power management IC from Texas Instruments. 
The supply of higher priority is the 15-20W AC/DC Meanwell (primary) power supply. When this supply is not present (e.g. in case of a blackout) the power management IC automatically switches to the 5V buck converter (secondary supply). This buck converter can be supplied from either the internal battery, the external battery or by the output of the MPPT. Out of these three voltage sources, the supply with the higher voltage will be used. 
The switchover between then main supply and the secondary supply is seamless which means the system does not notice that the power source has been switched and no brownout reset / restart will be triggered. 
<div class="page-break" style="page-break-before: always;"></div>

### 4.3.11 Power Management
Power management of the components connect to the battery is the most critical part handled by the PCM because of its direct impact on battery health and lifetime. To ensure the longest battery lifetime possible, the power management currently uses only 85% of the usable capacity of the battery to leave some safety margin for the degradation the battery and to avoid operating the battery at low capacity. The power management has been implemented in 3 steps: 
1. Inverter Power Reduction The first step when the battery voltage reaches 47 V or lower is to reduce the output power of the inverter to 0W (see 6.3). Even the output power is reduced to 0W, the inverter is still synchronized to the grid and needs a significant amount of standby power which can still drain the battery. The reason why the inverter is not turned off immediately is because the produced power of the PV can jump up at any time (especially on a cloudy day where the sun might only peak for several minutes) which would otherwise cause the inverter to turn on and off repeatedly which would put unnecessary strain on the power MOSFETs. However, with the used approach, the output power of the inverter can be increased again when the PV power causes to batteries voltage to exceed the 47 V threshold. 
2. Battery low indication The step is responsible for putting the system into a battery low state when the battery is reaching certain limits. 
   This state can be entered when the batteries BMS reports an SOC lower than 15%, the cell low voltage warning of the BMS is triggered or the pack voltage (measured by the PCM) drops below 45V. This battery low state will be reverted when the SOC exceeds 25% and the BMS cell low voltage flag is not issued. 
   Entering the battery low state causes both inverters to enter their software specific low power modes where power consumption is reduced significantly. 
   In case the communication to the battery is failing and the SOC is not available, a backup system is used where only the pack voltage is used to determine whether this state should be entered or not. 
3. Entering sleep mode The last power saving mechanism activates when the battery low state has been activated and the pack voltage (which in this case is very close to the OCV voltage) is dropping below 46.5V. In this case, the DC power supply for both inverters will be cut. Additionally, the Raspberry Pi will be turned off, all communication between the components will be stopped by the MCU and the PCM enters power saving mode.

**Please note:** When the power management is operating correctly, the battery never hits a SOC value of 0% which also means that the SOC is only re-calibrated when the battery reaches the floating charge voltage.