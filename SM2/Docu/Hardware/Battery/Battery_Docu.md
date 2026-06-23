# 5 BATTERY

The battery is the main energy storage component in the SolMate system. The current supported battery family is the Topband 48 V LiFePO4 rack battery. The available SolMate variants use 25 Ah or 30 Ah packs. The attached `TB4825F-T110D-3201-ESS Specification V00.pdf` describes the 25 Ah pack; the 30 Ah pack is electrically equivalent for this documentation except for its higher capacity and energy.

The PCM supervises the battery through two paths:

- The PCM measures battery voltage, current and MOSFET temperature locally with its ADCs.
- The PCM reads BMS data from the Topband RS485 Modbus interface.

![Topband Battery](../Images/Topband_Battery.png)

## 5.1 ELECTRICAL CHARACTERISTICS

| Parameter | 25 Ah Battery | 30 Ah Battery | Unit / Notes |
| --------- | ------------- | ------------- | ------------ |
| Cell chemistry | LiFePO4 | LiFePO4 | |
| Rated capacity | 25 | 30 | Ah |
| Energy | 1.2 | 1.44 | kWh |
| Nominal voltage | 48 | 48 | V |
| Outgoing voltage | >= 40.5 | >= 40.5 | V |
| Limited charge voltage | 52.5 | 52.5 | V |
| Floating charge voltage | 51.0 | 51.0 | V |
| Discharge cut-off voltage | 40.5 | 40.5 | V |
| Maximum charge current | 25 | 25 | A |
| Maximum discharge current | 25 | 25 | A |
| Internal resistance | <= 40 | <= 40 | mOhm |
| Communication | CAN / RS485 | CAN / RS485 | PCM firmware uses RS485 |
| IP rating | IP20 | IP20 | |
| Dimensions | 442 x 88 x 243 | 442 x 88 x 243 | mm, 19 inch rack format |
| Net weight | approx. 14.8 | higher than 25 Ah pack | kg |
| Charging temperature | 0..50 | 0..50 | deg C |
| Discharging temperature | -20..60 | -20..60 | deg C |
| Recommended operating temperature | 15..35 | 15..35 | deg C |
| Storage temperature, less than 1 month | -20..45 | -20..45 | deg C |
| Recommended storage | 15..35, 45..80 %RH | 15..35, 45..80 %RH | |
| Self-discharge | <= 2 %/month, <= 12 %/year | <= 2 %/month, <= 12 %/year | |
| Cycle life | >= 4000 | >= 4000 | at 80 % DOD, 0.33 C |
| Parallel connection | up to 14 packs | up to 14 packs | no series connection |

For long-term storage, charge the battery to about 50 % SOC, store it in the recommended environment, and perform a full charge/discharge/charge-to-50 % cycle at least every 6 months.

## 5.2 BMS PROTECTION THRESHOLDS

The BMS monitors cells, pack voltage, current, temperature and MOSFET state. The values below are from the Topband specification.

| Protection | Alarm | Protection | Release |
| ---------- | ----- | ---------- | ------- |
| Cell overvoltage | 3.65 +/-0.03 V | 3.75 +/-0.03 V | 3.40 +/-0.03 V |
| Pack overvoltage | 54.75 +/-0.5 V | 56.25 +/-0.5 V | 54.0 +/-0.5 V |
| Cell undervoltage | 2.90 +/-0.03 V | 2.70 +/-0.03 V | 3.00 +/-0.03 V |
| Pack undervoltage | 43.5 +/-0.5 V | 40.5 +/-0.5 V | 45.0 +/-0.5 V, charge to recover |
| Charge overcurrent | 28 +/-5 A | 30 +/-5 A, 5 +/-1 s delay | auto release after 1 min |
| Discharge overcurrent | 25 +/-5 A | 30 +/-10 A, 5 +/-1 s delay | auto release after 1 min |
| Short circuit | | 280 A, 200..800 us delay | |
| Charge overtemperature | 50 +/-3 deg C | 55 +/-3 deg C | 45 +/-3 deg C |
| Discharge overtemperature | 60 +/-3 deg C | 65 +/-3 deg C | 55 +/-3 deg C |
| Charge low temperature | 3 +/-3 deg C | 0 +/-3 deg C | 5 +/-3 deg C |
| Low SOC | 5 % | | |

The PCM firmware additionally writes BMS parameter updates at startup. It lowers the BMS charge/discharge low-temperature alarm/protection settings to allow lower-temperature operation and writes the BMS function setting that disables the heater. The firmware records whether this BMS parameter update succeeded in the internal status flags.

## 5.3 COMMUNICATION

The battery BMS uses RS485 with Modbus-style framing.

| Parameter | Value |
| --------- | ----- |
| Baud rate | 9600 bit/s |
| Parity | none |
| Data bits | 8 |
| Stop bits | 1 |
| CRC | Modbus CRC16, transmitted LSB first then MSB |
| Base BMS address | 38 |

The battery address is `38 + DIP switch value`. In the SolMate firmware, the internal battery is addressed as `38` and the external battery as `39`.

Supported BMS function codes used by the PCM:

| Function Code | Function |
| ------------- | -------- |
| `0x03` | Read collected BMS information |
| `0x11` | Read product/vendor information |
| `0x12` | Control command: sleep, reset, wake |

Control subcommands:

| Command | Subcommand |
| ------- | ---------- |
| Sleep | `0xF0` |
| Reset | `0xF1` |
| Wake | `0xF2` |

The PCM sends a wake command to the internal battery at startup. A shutdown request sends the BMS sleep command.

### 5.3.1 Address Switches and Connector

The Topband front panel has four address switches. The switch value is binary-coded from 0 to 15. Pack 0 therefore uses BMS address 38, Pack 1 uses address 39, and so on.

RS485 connector pinout:

| Pin | Function |
| --- | -------- |
| 1 | B |
| 2 | A |
| 3 | GND |
| 4 | NC |
| 5 | NC |
| 6 | GND |
| 7 | A |
| 8 | B |

Front-panel indicators:

| Item | Description |
| ---- | ----------- |
| RUN | Green LED flashing indicates running state. |
| ALM | Red LED flashing indicates alarm; steady red indicates protection state. |
| SOC | Four green LEDs indicate battery capacity. |
| RESET | Press for more than 3 s to activate or reset the battery pack. |

### 5.3.2 BMS Register Map Used by PCM

The PCM reads 39 registers starting at `0x1000` every 5 s when the measured battery voltage is valid. Communication is attempted only after the local PCM voltage measurement has been above 43 V for three polling cycles.

| Register | Parameter | Unit / Scaling | PCM Use |
| -------- | --------- | -------------- | ------- |
| `0x1000` | Pack voltage | raw * 0.01 V | BMS pack voltage |
| `0x1001` | Pack current | raw * 0.1 A - 1000 A | BMS pack current |
| `0x1002` | Remaining capacity | raw * 0.1 Ah | BMS capacity |
| `0x1003` | Average cell temperature | (raw - 400) * 0.1 deg C | Cell temperature |
| `0x1004` | Environment temperature | (raw - 400) * 0.1 deg C | Environment temperature |
| `0x1005` | Warning flags | bit field | Battery warnings |
| `0x1006` | Protection flags | bit field | Battery protections |
| `0x1007` | Fault/status flags | bit field | Battery fault/status |
| `0x1008` | SOC | raw * 0.01 % | BMS SOC |
| `0x1009` | Average charge voltage | raw * 0.01 V | Read but not currently stored by PCM |
| `0x100A` | Float charge voltage | raw * 0.01 V | Read but not currently stored by PCM |
| `0x100B` | Maximum charging current | raw * 0.1 A - 1000 A | Read but not currently stored by PCM |
| `0x100C` | Maximum discharging current | raw * 0.1 A - 1000 A | Read but not currently stored by PCM |
| `0x100D` | SOH | raw * 0.01 % | State of health |
| `0x100E` | Full charge capacity | raw * 0.1 Ah | PCM stores as mAh |
| `0x100F` | Pack address | raw | Read but not currently stored by PCM |
| `0x1010` | Cycle count | raw | Battery cycle count |
| `0x1011..0x1020` | Cell voltage 1..16 | mV | Protocol-defined cell voltage block |
| `0x1021..0x1024` | Cell temperatures 1..4 | (raw - 400) * 0.1 deg C | Protocol-defined cell temperature block |
| `0x1025` | Balance temperature | (raw - 400) * 0.1 deg C | Read but not currently stored by PCM |
| `0x1026` | MOS temperature | (raw - 400) * 0.1 deg C | PCM uses its local MOSFET temperature measurement instead |

The current PCM firmware reads through `0x1026`, but its data structure stores 15 cell voltages and 4 cell temperatures. This does not map one-to-one to the full 16-cell protocol table from Topband; when changing the BMS parser, check the register offsets against the supplier protocol.

### 5.3.3 Product/Vendor Information

The PCM reads BMS vendor information once after valid internal battery communication is available.

| Field | Size | Description |
| ----- | ---- | ----------- |
| Model | 10 bytes | ASCII model string |
| Software version | 2 bytes | Version and revision |
| Hardware version | 5 bytes in protocol, 6 bytes in PCM storage | Hardware version data |
| Serial number | 20 bytes | ASCII serial number |
| Full capacity | derived from BMS data register `0x100E` | Used as battery full capacity in mAh |

## 5.4 STATUS AND ERROR FLAGS

### 5.4.1 PCM Battery Status and Error

The PCM maintains local battery state in addition to BMS state.

PCM battery status:

| Bit | Name | Description |
| --- | ---- | ----------- |
| 0 | Enabled | PCM requests the battery MOSFET path to be enabled. |

PCM battery error:

| Bit | Name | Description |
| --- | ---- | ----------- |
| 0 | Communication | PCM cannot communicate with the BMS. |
| 1 | Pre-charge | PCM battery pre-charge check failed. |

The PCM-side battery data exposed to the CM4 contains status, error, voltage, current, charge and MOSFET temperature.

### 5.4.2 BMS Warning Flags

| Bit | Name |
| --- | ---- |
| 0 | Cell overvoltage alarm |
| 1 | Cell low-voltage alarm |
| 2 | Pack overvoltage alarm |
| 3 | Pack low-voltage alarm |
| 4 | Charge overcurrent alarm |
| 5 | Discharge overcurrent alarm |
| 6 | Cell high-temperature alarm |
| 7 | Cell low-temperature alarm |
| 8 | Environment high-temperature alarm |
| 9 | Environment low-temperature alarm |
| 10 | MOS high-temperature alarm |
| 11 | Low-capacity alarm |

### 5.4.3 BMS Protection Flags

| Bit | Name |
| --- | ---- |
| 0 | Cell overvoltage protection |
| 1 | Cell low-voltage protection |
| 2 | Pack overvoltage protection |
| 3 | Pack low-voltage protection |
| 4 | Short-circuit protection |
| 5 | Overcurrent protection |
| 6 | Charging high-temperature protection |
| 7 | Charging low-temperature protection |
| 8 | Discharging high-temperature protection |
| 9 | Discharging low-temperature protection |

### 5.4.4 BMS Fault and Status Flags

| Bit | Name | Description |
| --- | ---- | ----------- |
| 0 | Front-end sampling fault | BMS analog front-end communication/sampling fault. |
| 1 | Temperature sensor fault | BMS temperature sensor fault. |
| 8 | Charging state | BMS reports charging active. |
| 9 | Discharging state | BMS reports discharging active. |
| 10 | Charge MOS break | Charge MOS status flag. |
| 11 | Discharge MOS break | Discharge MOS status flag. |
| 12 | Charge current limit open | BMS charge current limit active/open flag. |
| 13 | Heating active | BMS heating film active. |
| 14 | Parameter update successful | PCM-injected flag showing whether startup BMS parameter update succeeded. |

## 5.5 PCM BATTERY MANAGEMENT

### 5.5.1 Local Voltage, Current and Charge Measurement

The PCM measures internal and external battery voltage/current with its 16-bit ADC and calibrates current offset at startup. Current is set to zero in firmware when the corresponding PCM battery path is disabled.

The PCM maintains its own charge counter in mAh by integrating measured current:

- Charge current uses a correction factor of 1.0.
- Discharge current uses a correction factor of 1.03 when a minimum SOC threshold is configured.
- The counter is clamped at 0.
- The internal counter is clamped at the full capacity reported by the BMS when available.
- If a persisted charge value is invalid, the PCM estimates it from 41 V to 53 V.
- If BMS heating is active, the PCM holds the previous charge value instead of integrating current.

The internal and external charge counters are stored to EEPROM during shutdown.

### 5.5.2 Battery Switching and Pre-Charge

The PCM controls separate MOSFET and pre-charge paths for the internal and external batteries.

Internal battery enable sequence:

| Step | Timing | Action |
| ---- | ------ | ------ |
| 1 | 0 s | Enable internal battery pre-charge. |
| 2 | 5 s | Check pre-charge voltage on hardware revision 150. |
| 3 | after check | Enable internal battery MOSFET and disable pre-charge. |

If the common battery voltage is more than 10 V below the internal battery voltage during the revision-150 pre-charge check, the PCM disables pre-charge and sets the internal battery pre-charge error.

External battery enable sequence:

| Step | Timing | Action |
| ---- | ------ | ------ |
| 1 | 0 s | Enable external battery pre-charge. |
| 2 | 5 s | Enable external battery MOSFET and disable pre-charge. |

### 5.5.3 External Battery Pairing

When battery pairing is enabled and an external pack is detected, the PCM balances connection of the internal and external packs. The pairing logic runs every 10 s.

- The external pre-charge path is enabled if the on-grid inrush path is not active.
- The MPPT current limit is temporarily reduced to 5 A.
- If pack voltages are within +/-0.4 V, both packs are enabled.
- If the internal pack voltage is more than 0.4 V higher, the internal pack is disconnected and the external pack is enabled.
- If the external pack voltage is more than 0.4 V higher, the external pack is disconnected and the internal pack is enabled.
- After both packs are connected, the GaN MPPT current limit is restored.

If the external pack is not detected or pairing is disabled, the PCM disables the external battery path.

### 5.5.4 Battery-Low and Minimum-Charge Handling

When BMS communication is valid, the PCM uses BMS voltage and warning flags for battery-low decisions. The base low-battery voltage threshold is 47.2 V. In off-grid mode, this threshold is reduced with discharge current by up to 1.5 V at 25 A discharge.

Battery-low state is entered when:

- the BMS cell-low-voltage warning is active, or
- the BMS pack voltage remains below the calculated low-battery threshold long enough.

Battery-low state is cleared when the internal charge counter is above 1000 mAh and the BMS cell-low-voltage warning is inactive.

If BMS communication is not valid, the PCM falls back to local voltage measurement:

| Condition | Action |
| --------- | ------ |
| Internal battery voltage < 46 V | Enter battery-low state and reset charge counters to 0. |
| Internal battery voltage > 49 V | Clear battery-low state. |

The CM4 can configure a minimum charge threshold. If the internal charge counter is below that threshold and the system is not in off-grid mode, the PCM sets the minimum-charge-limit bit and disables inverter operation. The limit is cleared when charge rises at least 1000 mAh above the threshold.

### 5.5.5 Charging Limits and Smart Charging

The PCM uses battery state to control the MPPT:

- Default MPPT current limit: 15 A.
- On-grid inverter active: MPPT current limit can be raised to 25 A.
- Smart charging is enabled when the internal charge counter is within 2 Ah of full capacity or the BMS reports cell overvoltage.
- Smart charging is disabled when the counter falls more than 4 Ah below full capacity and no BMS cell-overvoltage warning is present.
- Battery low-temperature charge derating starts below 5 deg C. The MPPT current limit is mapped from 5 A at 5 deg C to 1 A at 0 deg C. Below 0 deg C, the limit is 0.5 A.

The PCM sets the internal charge counter to full capacity when full-battery evidence is sustained: local internal battery voltage above 53 V, BMS cell-overvoltage warning, MPPT battery-full status, or MPPT output voltage above the smart-charge threshold while smart charging is active.

### 5.5.6 Waking Up and Sleep Mode

The battery can be activated or reset by pressing the front-panel RESET button for more than 3 s. The Topband protocol also supports BMS sleep, reset and wake commands through function code `0x12`.

The PCM sends:

- wake (`0xF2`) to the internal battery during startup,
- sleep (`0xF0`) when the system shutdown sequence requests battery shutdown.

Always shut down the battery before working on the system. Connecting power connectors while the battery is running can create sparks and may permanently damage or degrade the connector.

### 5.5.7 Heating System

The Topband pack includes heating pads controlled by the BMS. The original battery specification describes heater operation when charging at low cell temperature.

The current PCM firmware writes a BMS function setting during startup that disables the heater. Battery heating can still appear in the BMS status flags if a pack is configured differently or the startup parameter update fails; when heating is reported active, the PCM inhibits on-grid inverter startup and freezes its local charge integration.
