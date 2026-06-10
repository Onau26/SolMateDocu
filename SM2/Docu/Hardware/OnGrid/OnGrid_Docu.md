# 6 ONGRID INVERTER
The grid-tie inverter will be sourced from Envertech (EVT800) with a custom firmware so output power can be adjusted via a communication interface.
Since the communication interface of the EVT800 uses UART, a custom PCB will be added which handles protocol and physical layer translation. 

![[Ongrid PCB.png|535]]

## 6.1 ELECTRICAL CHARACTERISTICS

| Parameter             | Symbol    | Min                        | Typ                                       | Max                                      | Unit |
| --------------------- | --------- | -------------------------- | ----------------------------------------- | ---------------------------------------- | ---- |
| Input Voltage         | Vin       | 16                         | 48                                        | 60                                       | V    |
| Output Voltage        | Vout      |                            | 220/230                                   |                                          | V    |
| Input Current         | Iin(cont) | 0                          |                                           | 24                                       | A    |
| Output Current        | Iout      | 0                          |                                           | 3.27                                     | A    |
| Output Power          | Pout      | 0                          |                                           | 800                                      | W    |
| Operating Temperature | Top       | -40                        |                                           | 65                                       | °C   |
| Efficiency            | 𝜂        |                            |                                           | 95.6                                     | %    |
| Compliance            |           | VDE-AR-N 4105, IEC/EN61000 | IEC/EN62109-1/2, EN50549-1/2019 TOR 2019, | C10/11:2019 UTE C15-712-1:2013, VFR 2019 |      |

## 6.2 COMMUNICATION 
For the physical layer RS485 is used. On top of this Layer Modbus RTU is implemented which is a protocol which has been standardized for industrial applications.
The following parameters can be written/read using this interface. 

| Parameter      | Unit | Description                                              |
| -------------- | ---- | -------------------------------------------------------- |
| Status         |      | Indication the status                                    |
| Error          |      | Indicates errors                                         |
| Temperature    | °C   | Temperature of the Inverter (measured at the DC MOSFETs) |
| Voltage AC     | V    | Grid Voltage                                             |
| Current AC     | A    | Injected current into the grid                           |
| Active Power   | W    | Active power delivered to the grid                       |
| Apparent Power | W    | Apparent power delivered to the grid                     |
| Total Energy   | kWh  | Total injected energy since system production            |

The Status and Error fields are containing bits which give more information about the system. 

### 6.2.1 Status 

| Bit | Short Description | Description                                                 |
| --- | ----------------- | ----------------------------------------------------------- |
| 0   | Power Startup     | Pre-charge circuit is active – Inverter is powering up      |
| 1   | Not implemented   | Not implemented                                             |
| 2   | Enabled           | Low resistance MOSFETs are enabled – Inverter is powered up |
| 3   | Grid Sync         | Inverter is synced to the grid and can start injection      |

### 6.2.2 Error 

| Bit | Short Description | Description                                                |
| --- | ----------------- | ---------------------------------------------------------- |
| 0   | Communication     | A communication error occurred                             |
| 1   | Temperature Limit | The maximum allowed temperature rating has been exceeded   |
| 2   | Energy Meter      | Communication with the energy metering IC is not possible  |

Note: Every error condition leads to the Inverter shutting down and halting operation until error condition is removed and the system has been restarted. 

## 6.3 INJECTION STRATEGY 
To optimize battery life and use the available capacity as best as possible, a control algorithm for the on-grid inverter das been developed which assures that the battery is never used outside its rated specifications. This is done by artificially limiting the voltage window in which the inverter is allowed to operate.
When the battery reached its floating charge voltage and the on-grid inverter is enabled, the charging current into the battery will be limited to 0 A which is done by injecting the excess current produced by the MPPT into the grid.

![[Injection Control.png|540]]