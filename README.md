# Rain-Aware Automated Garden Irrigation System

## Overview

This project is a simple embedded system designed to automate the irrigation process for a small home garden. The system monitors soil humidity levels in two separate plant basins and detects rainfall to decide whether watering should be enabled or disabled.

The main goal of the project is to prevent unnecessary watering, especially during rainfall, while still ensuring that dry soil receives enough water. The project was implemented using the **PIC16F877A microcontroller**, programmed in **PIC Assembly**, and simulated using **Proteus**.

## Project Description

The garden contains two plant basins:

* **B1**
* **B2**

Each basin has a soil humidity sensor that measures the moisture level of the soil. A rainfall sensor is placed between the basins to detect the amount of fallen rain. Based on these sensor readings, the system controls two automatic irrigation valves.

Since the project is simulated in Proteus, the valves are represented using **green LEDs**:

* LED ON means the valve is open and water is flowing.
* LED OFF means the valve is closed and water is stopped.

The system supports both:

* **Automatic irrigation mode**
* **Manual irrigation mode**

## Main Features

* Reads soil humidity from two different plant basins.
* Detects rainfall level using a water level sensor.
* Automatically opens or closes irrigation valves based on sensor readings.
* Stops irrigation during rainfall to prevent soil saturation.
* Displays rainfall level using an LED barograph.
* Displays soil humidity percentage using multiplexed 7-segment displays.
* Uses ADC to read analog sensors.
* Uses timer interrupt to generate a 1-second timing sequence.
* Uses ADC interrupt to detect conversion completion.
* Supports automatic and manual irrigation modes.
* Fully simulated in Proteus.

## System Components

### Microcontroller

The system is based on the:

* **PIC16F877A**

This microcontroller was used because it provides:

* Multiple I/O ports
* Built-in ADC module
* Timer modules
* Interrupt support
* Suitable architecture for low-level embedded control

### Sensors

#### 1. Soil Humidity Sensors

Two soil humidity sensors are used:

* One sensor for basin B1
* One sensor for basin B2

Each sensor provides an analog output between 0V and 5V:

| Sensor Output | Meaning                     |
| ------------- | --------------------------- |
| 0V            | Soil is completely dry      |
| 2.5V          | Soil humidity is around 50% |
| 5V            | Soil has maximum moisture   |

The analog output is connected to the ADC input of the PIC microcontroller.

#### 2. Rainfall / Water Level Sensor

A water level sensor is used to detect rainfall. The sensor output increases as the detected water level increases.

The system assumes that:

* 0V means no rain.
* Higher voltage means higher rainfall.
* Maximum measurable water level is 50 mm.

The rainfall value is used to decide whether the irrigation system should stop watering.

### Outputs

The system uses multiple output devices:

* Green LEDs to simulate irrigation valves.
* LED barograph to display rainfall level.
* Multiplexed 7-segment displays to show soil humidity percentage.

## Control Logic

The irrigation decision is based on rainfall level and soil humidity.

| Rainfall Level | Soil Humidity | B1 Valve | B2 Valve |
| -------------- | ------------- | -------- | -------- |
| ≥ 20 mm        | Any           | Closed   | Closed   |
| Any            | ≥ 60%         | Closed   | Closed   |
| < 20 mm        | ≥ 40%         | Closed   | Open     |
| < 20 mm        | < 40%         | Open     | Open     |

### Explanation

* If rainfall is greater than or equal to 20 mm, both valves are closed.
* If the soil humidity is high enough, irrigation is stopped.
* If the soil is moderately humid, only one basin may need watering.
* If the soil is dry and there is no significant rainfall, both valves are opened.

## Operating Modes

### Automatic Mode

In automatic mode, the system decides whether to open or close the irrigation valves based on:

* Rainfall sensor reading
* B1 soil humidity reading
* B2 soil humidity reading

The control rules are applied automatically without user intervention.

### Manual Mode

A switch is used to select between automatic and manual mode.

When the switch is set to manual mode, the system allows direct manual control of the irrigation process instead of relying only on automatic sensor-based decisions.

## ADC Sampling Strategy

The system has three analog inputs:

1. Rainfall sensor
2. B1 soil humidity sensor
3. B2 soil humidity sensor

These sensors are sampled in sequence.

The sampling order is:

1. Read rainfall sensor
2. Wait 1 second
3. Read B1 soil humidity sensor
4. Wait 1 second
5. Read B2 soil humidity sensor
6. Wait 1 second
7. Repeat the cycle

This means each sensor is sampled once every 3 seconds, giving each channel a sampling frequency of:

```text
1/3 Hz
```

## Interrupt Usage

### Timer Interrupt

A hardware timer interrupt is used to generate the 1-second delay required between ADC readings.

Instead of using only software delay loops, the system uses timer-based timing to make the sampling process more reliable and structured.

### ADC Interrupt

The ADC interrupt is used to detect when an analog-to-digital conversion is complete.

This allows the system to know when the ADC result is ready to be processed.

## Display System

### LED Barograph

The rainfall level is displayed using an LED barograph.

Each LED bar represents approximately:

```text
5 mm of rainfall
```

For example:

| Rainfall Level | Barograph Display |
| -------------- | ----------------- |
| 10 mm          | 2 LEDs ON         |
| 25 mm          | 5 LEDs ON         |
| 45 mm          | 9 LEDs ON         |

This gives a simple visual indication of the detected rain level.

### Multiplexed 7-Segment Displays

Two multiplexed 7-segment displays are used to show the soil humidity percentage.

Multiplexing reduces the number of required microcontroller pins by rapidly switching between the displays while giving the appearance that both are ON at the same time.

## Tools Used

* **MPLAB IDE**
* **Proteus**
* **PIC16F877A**
* **PIC Assembly**
* **Soil Moisture Sensor Library for Proteus**
* **Water Sensor Library for Proteus**

## Hardware Simulation

The project was simulated in Proteus because the required sensors are not available by default in the simulator.

External sensor libraries were added for:

* Soil moisture sensor
* Water level sensor

Since there is no real soil or rainfall in simulation, potentiometers were used to simulate analog sensor values.

The generated `.hex` file from MPLAB was loaded into the PIC16F877A component inside Proteus to run the system simulation.

## Project Flow

```text
Start
  |
  v
Initialize ports, ADC, timers, interrupts, and displays
  |
  v
Check selected mode
  |
  +--> Manual Mode
  |       |
  |       v
  |   Control valves manually
  |
  +--> Automatic Mode
          |
          v
      Read rainfall sensor
          |
          v
      Display rainfall on LED barograph
          |
          v
      Read B1 soil humidity
          |
          v
      Read B2 soil humidity
          |
          v
      Display soil humidity on 7-segment displays
          |
          v
      Apply irrigation control rules
          |
          v
      Open or close valves
          |
          v
      Repeat
```

## Main Implementation Details

### ADC Configuration

The ADC module is configured to read analog values from the sensors. Since the system has multiple analog inputs, the ADC channel is switched during runtime.

The ADC is responsible for converting the analog voltage from each sensor into a digital value that can be processed by the program.

### Timer Configuration

A hardware timer is configured to generate a 1-second timing event. This timing is used to organize the sensor sampling sequence.

### Port Usage

The system uses the PIC ports for:

* Reading sensor values through ADC pins
* Reading the mode selection switch
* Controlling valve LEDs
* Driving the LED barograph
* Controlling multiplexed 7-segment displays

### Valve Simulation

Proteus does not include real irrigation valves, so LEDs are used instead.

* LED ON = valve open
* LED OFF = valve closed

This makes it easy to verify the control behavior visually during simulation.

## Challenges

Some of the main challenges in the project were:

* Configuring ADC channel switching correctly.
* Using ADC interrupt to handle conversion completion.
* Generating accurate timing using hardware timer interrupts.
* Reducing port pin usage as much as possible.
* Displaying values using multiplexed 7-segment displays.
* Correctly mapping rainfall level to the LED barograph.
* Implementing the control rules in PIC assembly.
* Testing different sensor values in Proteus using potentiometers.

## What I Learned

Through this project, I gained practical experience in:

* PIC16F877A microcontroller programming
* Embedded systems design
* Assembly language programming
* ADC configuration and analog sensor reading
* Timer interrupt handling
* ADC interrupt handling
* Sensor interfacing
* Display interfacing
* Proteus simulation
* Rule-based embedded control systems
* Manual and automatic control modes

## Possible Future Improvements

The project can be improved by adding:

* Real hardware implementation instead of only simulation.
* LCD display for clearer sensor readings and system status.
* More plant basins.
* Adjustable humidity thresholds.
* Water pump control instead of only valve simulation.
* Data logging for soil moisture and rainfall history.
* Low-power sleep mode for battery operation.
* Wireless monitoring using Bluetooth or Wi-Fi.

## Repository Structure

```text
Rain-Aware-Automated-Garden-Irrigation-System/
│
├── AquaWatch.c        # Embedded C source code for the irrigation control system
├── AquaWatch.mcp      # MPLAB project file
├── AquaWatch.mcs      # MPLAB workspace/session file
├── AquaWatch.pdsprj   # Proteus simulation project
└── README.md          # Project documentation
```

## How to Run the Project

1. Open `AquaWatch.mcp` in MPLAB.
2. Build the project to generate the output HEX file.
3. Open `AquaWatch.pdsprj` in Proteus.
4. Double-click the PIC16F877A microcontroller in the Proteus schematic.
5. Load the generated HEX file into the microcontroller.
6. Run the simulation.
7. Adjust the simulated sensor inputs to test soil moisture and rainfall conditions.
8. Observe the valve LEDs, rainfall barograph, and display outputs.


## Notes

* The project was implemented using PIC Assembly.
* Proteus sensor libraries must be added before running the simulation.
* Potentiometers are used to simulate analog sensor readings.
* The system uses LEDs to represent irrigation valves.

## Author

**Izzeldeen Akloub**
Computer Engineering
University of Jordan

## License

This project is intended for educational purposes.
