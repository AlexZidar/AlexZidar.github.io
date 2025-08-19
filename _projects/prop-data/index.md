---
layout: post
title: Custom Data Acquisition System
description: As another contribution to Swamp Launch, I independently developed a custom data aquisition system (DAQ) for propulsion development. This DAQ is designed for use in testing and verifying prototype rocket motors and propellant formulation. The system records casing temperature, thrust, and combustion chamber pressure. This data is then immediately stored as a CSV to the on-board micro-SD card. Supplemental software was developed in Python for rapid report making and classification of rocket motors.

skills: 
  - C++
  - Python
  - Embedded Systems
  - Data Analytics
  - Propulsion
  - SolidWorks
  - 3D Printing


main-image: /daqlaunch.jpg
---

---


## Electronics:
{% include image-gallery.html images="daqelectronics.jpg" height="400" %}  

## Enclosure:
{% include image-gallery.html images="daqenclosure.jpg" height="400" %}  

## Analysis Tool:
{% include image-gallery.html images="daqanalysis.png" height="400" %}

## Example Output Report:
{% include image-gallery.html images="daqresults1.png" height="400" %}
{% include image-gallery.html images="daqresults2.png" height="400" %}

## Components:
- Teensy 4.1 Microcontroller
- Adafruit MCP9601 Thermocouple Amplifier
- LM085IT3.3 Linear Dropout Regulator
- SparkFun Qwiic Scale - NAU7802
- 1000 kG Load Cell
- 1000 psi Pressure Transducer
-	Generic K-Type Thermocouple
- Indicator LED
- Record Switch
- JST 1x2 Header
- JST 1X3 Header

## Capabilities:
- Temperature at 200hz
- Thrust at 320hz
- Pressure at 320hz
- Automatic Motor Classification and Report Generation



## Report Maker Code Snippet:
### Highlights calculation of a variety of important data.  
```Python
 # Calculate time differences between data points for integration
    df['delta_t'] = df['time_seconds'].diff().fillna(0)
    
    # Calculate impulse at each time step
    df['impulse_step'] = df['thrust_newtons'] * df['delta_t']
    
    # Calculate total impulse
    total_impulse = df['impulse_step'].sum()
    
    # Calculate average thrust
    avg_thrust = df['thrust_newtons'].mean()
    max_thrust = df['thrust_newtons'].max()
    
    # Calculate burn time (time from first non-zero thrust to last non-zero thrust)
    thrust_threshold = max_thrust * 0.05  # 5% of max thrust as threshold
    thrust_data = df[df['thrust_newtons'] > thrust_threshold]
    if not thrust_data.empty:
        burn_time = thrust_data['time_seconds'].max() - thrust_data['time_seconds'].min()
    else:
        burn_time = 0
    
    # Motor classification based on total impulse
    motor_classes = {
        0: "N/A", 1.26: "A", 2.51: "B", 5.01: "C", 10.01: "D", 20.01: "E",
        40.01: "F", 80.01: "G", 160.01: "H", 320.01: "I", 640.01: "J",
        1280.01: "K", 2560.01: "L", 5120.01: "M", 10240.01: "N", 20480.01: "O"
    }
    
    motor_class = "N/A"
    for impulse_threshold, class_letter in sorted(motor_classes.items()):
        if total_impulse >= impulse_threshold:
            motor_class = class_letter
        else:
            break
```
