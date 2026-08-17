# Project Proposal: Smart Cellular-Enabled Solar Charge Controller (Prototype)

## 1. Background and Problem Statement
In rural Tanzania and across East Africa, off-grid solar installations predominantly rely on low-cost, imported PWM charge controllers. While financially accessible, these "dumb" controllers offer zero remote visibility. For mini-grid operators, agricultural setups, and NGOs deploying hundreds of distributed systems, this lack of telemetry is a massive operational blind spot. Battery degradation, system overloads, or component failures are only discovered when the end-user complains, requiring expensive and time-consuming physical maintenance trips. 

While premium IoT-enabled controllers and proprietary Pay-As-You-Go (PayGo) ecosystems exist, they are either prohibitively expensive for small-scale deployments or locked into closed hardware loops. There is a gap for an open, highly configurable charge controller that marries low-cost PWM power switching with modern MQTT-based cellular telemetry.

## 2. Project Objectives

### General Objective
To design, fabricate, and program a standalone, smart PWM solar charge controller capable of remote telemetry and configurable battery chemistry management, optimized for rural off-grid applications.

### Specific Objectives
1.  **Develop multi-chemistry charging profiles:** Implement precise voltage regulation algorithms to safely charge both standard Lead-Acid and modern LiFePO₄ battery banks.
2.  **Engineer automated environment responses:** Implement dusk-to-dawn load switching logic by utilizing the solar panel as an ambient light sensor.
3.  **Implement safe voltage auto-detection:** Design the system to automatically configure itself for 12V or 24V operation based strictly on initial battery connection parameters (ignoring PV open-circuit voltage).
4.  **Integrate cellular IoT telemetry:** Deploy a cellular modem to transmit daily health metrics (voltage, current, yield) via MQTT over low-bandwidth GPRS/LTE networks, with SMS fallback for critical failure alerts.
5.  **Enable local interface and debugging:** Incorporate a segmented LCD for on-site visual feedback and a dedicated hardware port for firmware flashing and local serial data logging.

## 3. Scope and Methodology
This project focuses on the prototyping phase. The hardware architecture will decouple the low-voltage logic (MCU and communications) from the high-current power paths (MOSFETs). The firmware will be developed in C++ using the PlatformIO environment, utilizing a strictly non-blocking state machine architecture to ensure cellular network negotiations do not interrupt the critical real-time PWM switching required for safe battery charging. 

## 4. Expected Deliverables and Results
Upon completion, the project will yield:
*   **Hardware Prototype:** A functional, custom-designed Printed Circuit Board (PCB) populated with an MCU, current/voltage sensing circuitry, a cellular modem, and power MOSFETs.
*   **Production Firmware:** A stable, non-blocking C++ codebase capable of executing charging algorithms while simultaneously handling MQTT/SMS communication.
*   **Live Telemetry Dashboard:** A working cloud endpoint (or local server setup) demonstrating the successful reception and parsing of the controller's MQTT JSON payloads.
*   **Technical Documentation:** Complete schematic diagrams, Bill of Materials (BOM), and pin-mapping documentation.

## 5. System Acceptance Criteria (Feature Lock)
The prototype will be considered successful if it demonstrates the following features under load:
1.  **Segmented LCD:** Accurately displays real-time battery voltage and load status.
2.  **Chemistry Selection:** Successfully toggles between Lead-Acid (Bulk/Absorption/Float) and LiFePO₄ (CC/CV) profiles.
3.  **Load Control:** Automatically switches the DC load on when PV voltage drops below 5V for a sustained period, and off when sunlight returns.
4.  **Voltage Sizing:** Correctly locks into 12V or 24V mode upon battery connection without user input.
5.  **Data/Programming Port:** Successfully flashes new firmware and outputs raw serial logs over a physical header.
6.  **Telemetry Transmission:** Successfully connects to a local cellular network (Vodacom/Airtel/Tigo) and publishes at least one MQTT payload without halting the PWM charging loop.
