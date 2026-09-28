# MPPT_FOR_SOLAR
MPPT Solar Charge Controller — Arduino Nano-based 50W MPPT charger using a synchronous buck converter and Perturb &amp; Observe algorithm for efficient solar energy extraction and battery charging.
A low-cost 50 W solar charge controller based on an Arduino Nano, designed to extract maximum available power from a photovoltaic panel using the Perturb and Observe (P&O) Maximum Power Point Tracking algorithm and a synchronous buck converter.

The system combines real-time voltage/current sensing, closed-loop PWM control, battery charging-state management, protection circuitry and an LCD-based monitoring interface.

1. Project Overview

Solar photovoltaic panels have nonlinear voltage-current and power-voltage characteristics. Their operating point changes with factors such as irradiance and temperature.

Therefore, simply connecting a solar panel directly to a battery does not guarantee operation at the panel's maximum power point.

This project implements an embedded MPPT controller that continuously measures the PV operating conditions and adjusts the converter duty cycle to keep the panel operating near its Maximum Power Point (MPP).

The developed system is designed around:

A 50 W solar panel
12 V lead-acid battery
Arduino Nano
Synchronous buck converter
Voltage and current sensing
Perturb and Observe MPPT
Battery charging state machine
Real-time LCD monitoring
Protection circuitry
2. Problem Statement

The power available from a solar panel varies with environmental conditions.

Changes in:

Solar irradiance
Temperature
Panel operating voltage
Load conditions

can shift the maximum power point.

A controller is therefore required to continuously adjust the operating point of the PV panel.

The objective of this project is to develop a practical, low-cost embedded controller that:

Measures PV voltage and current.
Calculates instantaneous PV power.
Tracks the maximum power point.
Controls a buck converter through PWM.
Charges a 12 V battery.
Monitors system parameters in real time.
Provides basic protection against abnormal conditions.
3. System Architecture

The overall system is divided into four major sections:

                 ┌─────────────────────┐
                 │    Solar Panel      │
                 │       50 W          │
                 └──────────┬──────────┘
                            │
                     PV Voltage/Current
                            │
                            ▼
                 ┌─────────────────────┐
                 │  Sensing Network    │
                 │ Voltage + ACS712    │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │    Arduino Nano     │
                 │                     │
                 │  P = V × I          │
                 │  P&O MPPT           │
                 │  PWM Control        │
                 └──────────┬──────────┘
                            │
                       PWM Duty Cycle
                            │
                            ▼
                 ┌─────────────────────┐
                 │ Synchronous Buck    │
                 │    Converter        │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │   12 V Battery      │
                 └─────────────────────┘

A monitoring and protection layer operates alongside the main power-control path.

4. Hardware
4.1 Solar Panel

The system is designed for a 50 W photovoltaic panel.

The panel provides the variable DC input power that is processed by the MPPT controller.

The documented design considers approximately 24 V open-circuit panel voltage for the selected configuration.

4.2 Arduino Nano

The Arduino Nano acts as the main MPPT controller.

Its responsibilities include:

Reading PV voltage.
Reading PV current.
Calculating instantaneous power.
Executing the P&O algorithm.
Updating PWM duty cycle.
Managing battery charging states.
Monitoring system parameters.
Updating the LCD interface.
4.3 Voltage Sensing

Voltage-divider circuits are used to measure:

PV voltage
Battery voltage

The measured voltages are scaled to levels suitable for the Arduino's analog inputs.

The controller uses these measurements to determine the current operating condition of the PV system and battery.

4.4 Current Sensing

An ACS712 Hall-effect current sensor is used to measure PV current.

The current measurement is required because MPPT depends on determining the instantaneous power:

P = V × I

The controller therefore obtains:

PV Voltage
     +
PV Current
     ↓
Instantaneous Power

The paper reports stable current measurements with low noise during prototype operation.

5. Synchronous Buck Converter

The power-conversion stage uses a synchronous buck converter.

The documented design operates at approximately:

Switching frequency: 50 kHz
Inductor: 33 µH
Output capacitor: 220 µF
MOSFETs: IRFZ44N
Gate driver: IR2104

The buck converter reduces and regulates the PV-side voltage to provide an appropriate charging output for the 12 V battery.

5.1 Converter Control

The Arduino controls the converter through PWM.

Conceptually:

Arduino Nano
     │
     │ PWM
     ▼
  IR2104 Driver
     │
     ▼
 IRFZ44N MOSFETs
     │
     ▼
 Buck Converter
     │
     ▼
 Battery

Changing the PWM duty cycle changes the converter operating point and therefore changes the electrical conditions seen by the solar panel.

6. Perturb and Observe MPPT

The project uses the Perturb and Observe (P&O) algorithm.

The algorithm repeatedly changes the converter operating point and observes the resulting change in PV power.

The basic process is:

Measure PV Voltage
        ↓
Measure PV Current
        ↓
Calculate Power
        ↓
Compare with Previous Power
        ↓
Determine Power Change
        ↓
Adjust PWM Duty Cycle
        ↓
Measure Again
        ↓
Repeat

The objective is to operate close to:

dP/dV ≈ 0

which corresponds to the region around the maximum power point.

7. P&O Control Logic

The controller continuously compares the current power with the previous measured power.

Conceptually:

             Measure V and I
                    │
                    ▼
                P = V × I
                    │
                    ▼
          Compare with Previous P
                    │
          ┌─────────┴─────────┐
          │                   │
       Power ↑             Power ↓
          │                   │
          ▼                   ▼
 Continue in              Reverse
 same direction           perturbation
          │                   │
          └─────────┬─────────┘
                    ▼
              Update PWM
                    │
                    ▼
                 Repeat

The algorithm therefore creates a feedback loop between:

PV measurements → power calculation → control decision → PWM → converter → PV operating point

8. PWM Generation

The controller generates PWM for the buck converter.

The documented implementation uses the TimerOne library to generate approximately 50 kHz PWM.

The duty cycle is constrained below 99% to maintain appropriate operation of the IR2104 bootstrap arrangement.

9. Battery Charging State Machine

The controller also considers the battery charging state.

The documented charging sequence includes:

          ┌──────────────┐
          │     BULK     │
          └──────┬───────┘
                 │
          Voltage Approaches
             Threshold
                 │
                 ▼
          ┌──────────────┐
          │    FLOAT     │
          └──────┬───────┘
                 │
       Abnormal / Low Input
                 │
                 ▼
          ┌──────────────┐
          │   CUT-OFF    │
          └──────────────┘
Bulk Mode

The controller allows high charging power while the battery is in the bulk charging stage.

Float Mode

The charging voltage is stabilized as the battery approaches the desired voltage level.

Cut-Off

The system can move to a cut-off condition when an over-voltage or insufficient-input condition is detected according to the implemented charging logic.

10. Monitoring Interface

A 20×4 I²C LCD provides real-time system information.

The documented interface displays parameters including:

PV voltage
PV current
PV power
PWM duty cycle
State of charge / charging information
Load state

LED indicators are also used for battery-status indication.

The monitoring architecture can be represented as:

Sensors
   │
   ▼
Arduino Nano
   │
   ├──► MPPT Control
   │
   ├──► Battery Control
   │
   └──► LCD Display
11. Protection

The system includes protection elements to improve reliability.

The documented protection architecture includes:

TVS diodes for voltage-spike protection.
MOSFET-based load switching.
Reverse-current prevention during low-irradiance/night conditions.
Controlled charging states.

These features supplement the MPPT control loop rather than replacing proper external electrical protection.

12. Complete Working Methodology

The complete signal and power flow is:

                SUNLIGHT
                   │
                   ▼
             SOLAR PANEL
                   │
                   ▼
           PV Voltage & Current
                   │
          ┌────────┴────────┐
          ▼                 ▼
   Voltage Divider       ACS712
          │                 │
          └────────┬────────┘
                   ▼
             ARDUINO NANO
                   │
                   ▼
              P = V × I
                   │
                   ▼
              P&O MPPT
                   │
                   ▼
             PWM Duty Cycle
                   │
                   ▼
             IR2104 Driver
                   │
                   ▼
              MOSFET Stage
                   │
                   ▼
          Synchronous Buck
             Converter
                   │
                   ▼
             12 V Battery

At the same time:

Arduino
   │
   ├──► LCD Monitoring
   ├──► Battery State Logic
   ├──► Protection Logic
   └──► Load Control
13. Feedback Control Loop

The MPPT system forms a closed-loop control system.

        ┌──────────────────────────────┐
        │                              │
        ▼                              │
   PV Operating Point                 │
        │                              │
        ▼                              │
  Voltage + Current                   │
        │                              │
        ▼                              │
    Power Calculation                 │
        │                              │
        ▼                              │
     P&O Algorithm                    │
        │                              │
        ▼                              │
    PWM Duty Cycle                    │
        │                              │
        ▼                              │
    Buck Converter                    │
        │                              │
        └──────────► PV Operating Point

The controller therefore continuously reacts to changes in the electrical operating point rather than using a fixed converter duty cycle.

14. Simulation and Development

The project development involved embedded programming and circuit-level analysis.

Software / Tools
Arduino IDE
Proteus
Embedded C/C++
Serial monitoring / LCD-based data visualization

The project architecture can be simulated and tested before hardware implementation to verify:

Sensor readings
PWM generation
Converter behavior
Control logic
Display operation
15. Experimental Results

The prototype demonstrated stable MPPT operation and successfully tracked the maximum power point through iterative duty-cycle perturbation.

The documented observations include:

Stable MPPT operation.
Minimal oscillation around the MPP.
Stable sensor readings.
Low sensing noise.
Predictable PWM response to irradiance changes.
Smooth battery-voltage progression from bulk toward float charging.
PV power measurements closely matching expected theoretical values for the 50 W panel.

The results indicate that the classical P&O algorithm is suitable for the project's small-scale, cost-sensitive application.

16. Key Engineering Decisions
Why P&O?

P&O was selected because it offers:

Simple implementation
Low computational requirements
Low hardware complexity
Straightforward integration with an Arduino
Suitability for small-scale MPPT systems

The project documentation also recognizes that more advanced adaptive and AI-based algorithms can offer improvements under rapidly changing conditions, but they increase complexity and computational requirements.

Why a Buck Converter?

A buck topology allows the higher PV-side voltage to be converted into a lower regulated charging voltage for the battery.

It also provides a controllable operating point through PWM.

Why Voltage and Current Sensing?

MPPT requires knowledge of instantaneous PV power.

Therefore:

Voltage × Current = Power

Accurate sensing directly affects the quality of the MPPT control decision.

17. Main Components
Component	Function
50 W Solar Panel	PV energy source
Arduino Nano	MPPT controller
ACS712	PV current sensing
Voltage Dividers	PV/battery voltage sensing
IRFZ44N MOSFETs	Converter switching
IR2104	MOSFET gate driver
33 µH Inductor	Buck converter energy storage
220 µF Capacitor	Output filtering
12 V Lead-Acid Battery	Energy storage
20×4 I²C LCD	System monitoring
TVS Diodes	Voltage-spike protection
MOSFET Load Switch	Load/reverse-current control

The listed values and components are taken from the documented project architecture.

18. Skills Demonstrated
Embedded Systems
Arduino Nano
Embedded C/C++
PWM generation
ADC-based sensing
Real-time control
Power Electronics
DC-DC conversion
Synchronous buck converter
MOSFET switching
Gate-driver interfacing
Inductor/capacitor selection
Battery charging
Control Systems
Maximum Power Point Tracking
Perturb and Observe algorithm
Closed-loop control
Duty-cycle control
Battery charging state machine
Hardware
Voltage sensing
Current sensing
Power-stage integration
Protection circuitry
Hardware testing
Tools
Arduino IDE
Proteus
Embedded C/C++
LCD/I²C interface
Serial monitoring
19. Development Workflow
Literature Review
       ↓
MPPT Algorithm Selection
       ↓
System Architecture
       ↓
Buck Converter Design
       ↓
Voltage/Current Sensing
       ↓
Arduino Control
       ↓
P&O Implementation
       ↓
Battery Charging Logic
       ↓
Protection & Monitoring
       ↓
Simulation
       ↓
Hardware Integration
       ↓
Experimental Testing
       ↓
Performance Evaluation
20. Future Scope

The project can be extended in several directions.

Adaptive MPPT

Possible future approaches include:

Adaptive-step P&O
MRAC
ANFIS
AI-assisted MPPT
PSO-based optimization

These approaches could improve convergence under rapidly changing irradiance conditions.

Higher-Power Operation

The system could be scaled to larger PV arrays by upgrading:

MOSFETs
Inductor
Protection circuitry
Power-stage components
IoT Monitoring

Connectivity could be added for:

Remote monitoring
Data logging
Performance analysis
Remote diagnostics
Multiple Battery Chemistries

The controller could be adapted for:

Li-ion
LFP
Gel batteries

through programmable charging profiles.

Partial-Shading Handling

Additional algorithms could be introduced to handle multiple local maxima caused by partial shading.

These features are future extensions, not part of the current documented prototype.

21. Project Status

Status: Functional Prototype

The documented system successfully demonstrates MPPT-based solar charging using a 50 W PV source, synchronous buck conversion, real-time sensing and Arduino-based P&O control. Experimental observations reported stable tracking and charging behavior under varying operating conditions.

22. Repository Structure

A suitable repository structure is:

MPPT-Solar-Charge-Controller/
│
├── README.md
│
├── firmware/
│   ├── mppt/
│   ├── pwm/
│   └── battery_control/
│
├── hardware/
│   ├── schematic/
│   ├── pcb/
│   └── components/
│
├── simulation/
│   └── proteus/
│
├── testing/
│   ├── measurements/
│   └── results/
│
├── documentation/
│
└── images/

Only create/include directories that correspond to the actual contents of the repository.

23. Key Takeaway

This project demonstrates the complete interaction between:

PV Generation → Electrical Sensing → Embedded Computation → MPPT Algorithm → PWM Control → Power Conversion → Battery Charging → Monitoring & Protection

The main engineering concept is the closed-loop adjustment of the converter operating point based on real-time electrical measurements.

Technologies

Arduino Nano Embedded C/C++ MPPT Perturb & Observe Power Electronics Synchronous Buck Converter PWM IR2104 IRFZ44N ACS712 Voltage Sensing Battery Charging Proteus I2C LCD
