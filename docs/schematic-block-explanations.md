# Incubator Temperature, Airflow, Humidity, and Protection Control Design

## 1. Purpose of the Design

This document explains the final initial control design for a small egg incubator with approximately 30–60 egg capacity. The design focuses on temperature control, heater current monitoring, airflow control, humidity control, and basic protection.

The system is based on a 12V supply because the incubator is intended to operate from a solar/battery power source. The control section is managed by a microcontroller such as an STM32 or Renesas MCU.

The final design includes:

- Two heating elements controlled by a MOSFET
- Total heater current sensing
- Temperature and humidity sensing
- Extra NTC temperature sensing points
- Circulation fans controlled by PWM
- A small exhaust fan controlled by PWM
- A solenoid pinch valve controlled by ON/OFF switching
- Flyback diode protection for inductive loads
- Common ground connection for all control and power sections
- Fuse/overcurrent protection marked as TBD after discussion with the power-supply team

---

## 2. Main Design Philosophy

The design was made using the following priorities:

1. **Cost efficiency**  
   The system avoids unnecessary expensive components such as motorized valves and motorized air flaps.

2. **Power efficiency**  
   Since the system is solar/battery-powered, loads are only activated when needed. Fans are PWM controlled, and the pinch valve is pulsed ON/OFF briefly.

3. **Reliability**  
   The design separates normal control from protection. The MCU controls normal operation, while hardware-level components such as flyback diodes and thermal cut-off/protection improve reliability.

4. **Scalability**  
   The system starts with one circulation fan and two heaters, but allows extra fans or NTC temperature points to be added later.

---

## 3. Heater Control and Current Sensing

### 3.1 Heater Arrangement

The incubator uses two 12V heating elements connected in parallel. The heaters are switched using a low-side N-channel MOSFET.

Basic heater path:

```text
12V → Fuse/TBD protection → Current Sensor → Heating Elements → MOSFET → GND
```

The MOSFET is controlled by a PWM signal from the MCU.

### 3.2 Why a Low-Side N-Channel MOSFET Was Used

A low-side N-channel MOSFET was selected because it is:

- Cheap and widely available
- Easy to drive directly from a 3.3V MCU pin if a logic-level MOSFET is used
- Efficient for PWM switching
- Simpler than high-side switching

The connection is:

```text
Heating element negative side → MOSFET drain
MOSFET source → GND
MCU PWM pin → gate resistor → MOSFET gate
Gate pulldown resistor → GND
```

The gate resistor limits switching spikes from the MCU pin, while the pulldown resistor keeps the MOSFET OFF during startup or when the MCU pin is floating.

### 3.3 Current Sensing Decision

A generic current sensor module is placed before the heaters split into two branches. This allows the system to measure the **total current** drawn by both heating elements.

This decision was made because:

- There are only two heating elements
- Total current measurement is cheaper than one sensor per heater
- It is enough to detect major heater faults
- It reduces wiring and MCU resource usage

The current sensor output goes to the MCU through either:

```text
Analog output → MCU ADC pin
```

or:

```text
Digital output/I2C → MCU communication pins
```

The exact current sensor module will be selected based on:

- Maximum heater current
- Voltage rating
- Output type
- Accuracy requirement
- Cost

### 3.4 Faults Detected by Current Sensing

The MCU can use the current reading to detect:

- Heater disconnected
- Open-circuit heater
- Abnormally low heater current
- Abnormally high heater current
- Possible short circuit
- Current flowing when heater PWM is OFF
- Possible MOSFET stuck-ON fault

Example firmware logic:

```c
if (heater_pwm > 0 && heater_current < minimum_current) {
    fault = HEATER_OPEN_OR_DISCONNECTED;
}

if (heater_current > maximum_current) {
    fault = HEATER_OVERLOAD_OR_SHORT;
    heater_pwm = 0;
}

if (heater_pwm == 0 && heater_current > leakage_limit) {
    fault = MOSFET_STUCK_ON;
}
```

---

## 4. Temperature and Humidity Sensing

### 4.1 Main Temperature and Humidity Sensor

The system uses one main temperature and humidity sensor for chamber feedback.

This sensor is used for:

- Main temperature control
- Humidity monitoring
- Display readings
- Alarm conditions
- Fan and humidity-control decisions

Depending on the selected sensor, it may connect through:

```text
VCC → 3.3V
GND → GND
DATA/SDA/SCL → MCU digital or I2C pins
```

### 4.2 Extra NTC Temperature Points

Additional NTC thermistors are used for temperature-only monitoring at different points inside the incubator.

Possible NTC locations include:

- Near the egg tray
- Near the heater area
- Near a corner/cold spot
- Near the airflow path
- Near the water tray area

### 4.3 NTC Voltage Divider Connection

Each NTC is connected as a voltage divider because the MCU cannot measure resistance directly.

Typical connection:

```text
3.3V
 │
Fixed resistor
 │
 ├── MCU ADC input
 │
NTC thermistor
 │
GND
```

A small capacitor may be added from the ADC node to ground to reduce noise:

```text
ADC node → 100nF capacitor → GND
```

### 4.4 Scalability of NTC Sensors

The design can be scaled in two ways.

#### Option 1: Direct ADC Inputs

Each NTC divider output connects to a separate MCU ADC pin.

This is simple and cheap if enough ADC pins are available.

#### Option 2: Analog Multiplexer

For more NTC sensors, the divider outputs can be connected to an analog multiplexer. The multiplexer output connects to one MCU ADC pin.

This allows extra NTC sensors to be added later by connecting another divider to an unused multiplexer channel and updating the firmware.

---

## 5. Hardware Overtemperature Cut-Off

A resettable thermal switch is included as a hardware cut-off option for heater protection.

The purpose of the resettable thermal switch is to provide independent protection if the MCU, firmware, sensor reading, or MOSFET control fails.

Recommended placement:

```text
12V heater supply → resettable thermal switch → current sensor → heaters → MOSFET → GND
```

The thermal switch should be a normally closed type. During normal temperature, it remains closed. When the chamber exceeds the selected safety limit, it opens and cuts heater power.

The exact fuse/protection arrangement is marked **TBD** because the power-supply and protection responsibility will be finalized after discussion with the relevant team member.

---

## 6. Air Circulation Control

### 6.1 Circulation Fan Decision

The final initial design uses two circulation fans in parallel, controlled by one MOSFET. The fans are PWM controlled by the MCU.

The purpose of the circulation fans is to:

- Mix air inside the incubator
- Reduce hot and cold spots
- Distribute humidity evenly
- Improve sensor reading stability
- Improve uniform heating around the eggs

### 6.2 Why PWM Control Was Used

PWM control was selected because it allows the MCU to control fan speed without wasting much power.

The fan speed can be adjusted based on:

- Temperature stability
- Humidity distribution
- Incubator operating stage
- Power-saving requirements

Basic connection:

```text
12V → Fans in parallel → MOSFET → GND
MCU PWM pin → gate resistor → MOSFET gate
Gate pulldown resistor → GND
```

### 6.3 Scalability

Although the design currently supports the selected circulation fans, it allows expansion by adding another fan header or another MOSFET channel if needed.

This is useful if testing shows:

- Uneven temperature
- Weak airflow
- Hot or cold corners
- Poor humidity distribution

---

## 7. Exhaust Fan and Ventilation

### 7.1 Final Ventilation Decision

The design includes a small exhaust fan for rapid short exhaust periods.

This decision was made instead of relying only on passive vents because the client expects a more automated machine.

The exhaust fan provides controlled air exchange by forcing stale/hot/humid air out of the incubator. Fresh air enters through inlet holes or vents.

### 7.2 Why a Separate Exhaust Fan Was Used

A separate exhaust fan was selected because:

- It gives the MCU actual control over air exchange
- It is cheaper than motorized vent flaps
- It uses power only for short periods
- It improves removal of excess heat and humidity
- It helps bring in fresh oxygen and remove carbon dioxide

The exhaust fan does not run continuously. It operates only when needed or during scheduled short ventilation periods.

### 7.3 Exhaust Fan Connection

The exhaust fan is also controlled by a low-side N-channel MOSFET.

Connection:

```text
12V → Exhaust fan → MOSFET → GND
MCU PWM pin → gate resistor → MOSFET gate
Gate pulldown resistor → GND
```

The exhaust fan is PWM controlled so that the MCU can adjust the strength of ventilation.

---

## 8. Humidity Control Using a Solenoid Pinch Valve

### 8.1 Final Humidity Control Decision

Humidity is controlled using a solenoid pinch valve connected to a hose from an external water reservoir.

The water path is:

```text
External water reservoir → silicone tube → solenoid pinch valve → water tray
```

The water tray is placed at the bottom of the incubator. Water evaporates naturally into the chamber, assisted by internal airflow.

### 8.2 Why a Pinch Valve Was Chosen

A solenoid pinch valve was selected instead of a motorized valve because it is:

- Cheaper
- Simpler to control
- Lower complexity
- Easier to maintain
- Less likely to clog internally because water only touches the tube
- Suitable for brief ON/OFF water dripping

### 8.3 Pinch Valve Control

The pinch valve is controlled using ON/OFF pulses, not PWM.

Connection:

```text
12V → Solenoid pinch valve → MOSFET → GND
MCU GPIO pin → gate resistor → MOSFET gate
Gate pulldown resistor → GND
```

Control logic:

```c
if (humidity < humidity_low_threshold) {
    valve_on();
    delay(short_pulse_time);
    valve_off();

    wait_for_evaporation();
}
```

This prevents continuous dripping and reduces the risk of overfilling the tray.

---

## 9. Flyback Diode Protection

### 9.1 Why Flyback Diodes Are Needed

Fans and solenoid valves are inductive loads. When an inductive load is switched OFF, it can produce a voltage spike. This spike can damage the MOSFET or disturb the MCU.

Flyback diodes are used to provide a safe path for this voltage spike.

### 9.2 Flyback Diode Connection

The flyback diode is connected across the inductive load.

For low-side MOSFET switching:

```text
12V → Load → MOSFET → GND
```

The diode is connected as:

```text
Diode cathode/stripe side → 12V side of load
Diode anode/non-stripe side → MOSFET drain side
```

### 9.3 Loads Protected by Flyback Diodes

Flyback diodes are used for:

- Circulation fan group
- Exhaust fan
- Solenoid pinch valve

Heating pads do not need flyback diodes because they are mainly resistive loads.

---

## 10. Capacitors for Noise Reduction

Small capacitors may be placed across fan terminals to reduce electrical noise from the fan motors.

Typical value:

```text
100nF ceramic capacitor across fan terminals
```

Purpose:

- Reduce high-frequency motor noise
- Improve sensor reading stability
- Reduce disturbance on MCU ADC and communication lines
- Improve general system reliability

A larger capacitor such as 47µF–100µF near the fan group supply is optional. It may be added if testing shows voltage dips, MCU resets, or unstable sensor readings when fans start.

---

## 11. Common Ground Requirement

All control and power sections must share a common ground reference.

This means:

```text
MCU GND = 12V supply GND = sensor GND = MOSFET source GND
```

This is necessary because the MCU gate signals and sensor readings need a stable reference point.

Without common ground, the MOSFETs may not switch correctly and the MCU may read incorrect sensor values.

---

## 12. MCU Pin Summary

The MCU pins are assigned by function as follows:

| Function                         | MCU Pin Type                          |
| -------------------------------- | ------------------------------------- |
| Heater MOSFET control            | PWM output                            |
| Circulation fan MOSFET control   | PWM output                            |
| Exhaust fan MOSFET control       | PWM output                            |
| Pinch valve control              | GPIO output, ON/OFF                   |
| Main temperature/humidity sensor | Digital/I2C input                     |
| NTC temperature points           | ADC input or analog multiplexer input |
| Current sensor                   | ADC input or digital/I2C input        |

---

## 13. Final Initial Design Summary

The final initial design contains the following sections:

### Heater Section

```text
12V → Fuse/TBD protection → Current Sensor → Heating Elements → MOSFET → GND
```

The MCU controls the heater through PWM and monitors total heater current for fault detection.

### Temperature and Humidity Section

```text
Main temperature/humidity sensor → MCU
NTC dividers → MCU ADC or analog multiplexer
```

The main sensor controls the chamber environment, while NTCs provide extra temperature points.

### Air Circulation Section

```text
12V → Circulation fans in parallel → MOSFET → GND
```

The MCU controls fan speed using PWM.

### Exhaust Section

```text
12V → Exhaust fan → MOSFET → GND
```

The exhaust fan provides short, rapid, automated air exchange.

### Humidity Water Control Section

```text
12V → Solenoid pinch valve → MOSFET → GND
```

The MCU opens the valve briefly when humidity is below the threshold.

---

## 14. Main Reasons for the Final Decisions

| Design Decision                        | Reason                                                                      |
| -------------------------------------- | --------------------------------------------------------------------------- |
| 12V system                             | Compatible with solar/battery power                                         |
| Low-side N-MOSFET switching            | Cheap, efficient, and easy to drive from MCU                                |
| Total heater current sensing           | Lower cost than per-heater sensing and enough for two heaters               |
| One main temperature/humidity sensor   | Provides main environmental feedback                                        |
| Extra NTC sensors                      | Cheap temperature-only monitoring at multiple points                        |
| PWM fan control                        | Saves power and allows speed regulation                                     |
| Separate exhaust fan                   | Enables automated ventilation without motorized flaps                       |
| Pinch valve instead of motorized valve | Cheaper, simpler, lower clogging risk                                       |
| Flyback diodes                         | Protect MOSFETs from inductive voltage spikes                               |
| Common ground                          | Required for correct MCU control and sensor readings                        |
| Fuse marked TBD                        | Final fuse selection belongs to the power-supply/protection team discussion |

---
