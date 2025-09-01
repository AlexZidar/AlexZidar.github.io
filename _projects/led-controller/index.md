---
layout: post
title: BlackBeard LED Controller
description: This universal LED controller was developed as part of my mentorship of First Robotics Competition (FRC) Team-79 “Krunch”. Throughout my 6 years on the team I went from member to president, and now mentor. In that time,I’ve found a distinct lack of suitable, robotics-focused LED controllers on the market. These controllers, notably REV Blinkin, are some combination of far too expensive or functionally limited. Therefore, I sought to challenge myself in developing a custom PCB and thus superior LED controller. Overall, this controller was designed to be flexible, inexpensive, and extremely easy to assemble. 
skills: 
  - C++
  - Embedded Systems
  - PCB Design
  - KiCad
  - Product Development
  - 3D Printing

main-image: /hardaf.jpg
---

---
## Results:
### Excluding optional pin headers, only 4 unique components are required. The board can receive commands via WiFi, Bluetooth, GPIO, I2C, or SPI. From there, the microcontroller interprets the commands to control two individually addressable strips and three power transistors, one for each color channel of non-addressable LED strips.

---

## Demonstration Video: Team-79
### Celebratory lighting on successful action, handled by the BlackBeard
{% include youtube-video.html id="jPneyFgghBk" autoplay= "false"%}

---

## Bare PCB:
{% include image-gallery.html images="BarePCB.jpg" height="400" %}  


## Fully Assembled:
{% include image-gallery.html images="FullPCB.jpg" height="400" %}

---

## Components:
- Custom PCB
- 3x TIP41C Power Transistor
- LM085IT3.3 Linear Dropout Regulator
- JST 1x2 Header
- 2x JST 1x6 Header
- 3x JST 1x3 Header
-	3x JST 1x4 Header

---

## Features:
- Comms over WiFi, Bluetooth, GPIO, I2C, or SPI
- Both Addressable and Non-Addressable LEDs
- Sub-$10 Total Component Cost
- Easy Assembly for Students
- Easy Programming for Students


---

## PWM Control Code Snippet:
### Shows how the ESP32 accepts PWM inputs for color, and optionally for brightness.   
```C++
void loop() {
  // Read the pulse width from the color control PWM input
  unsigned long colorPulseWidth = pulseIn(PWM_COLOR_PIN, HIGH, 30000);
  
  // Read the pulse width from the brightness control PWM input
  unsigned long brightnessPulseWidth = pulseIn(PWM_BRIGHTNESS_PIN, HIGH, 30000);
  
  // If no pulse is detected on the color input, skip this loop iteration
  if (colorPulseWidth == 0) {
    return;
  }
  
  // For debugging: print the measured pulse widths
  Serial.print("Color Pulse width: ");
  Serial.print(colorPulseWidth);
  Serial.print(" us, Brightness Pulse width: ");
  Serial.print(brightnessPulseWidth);
  Serial.println(" us");
   // Map the color pulse width to an RGB color.
  uint8_t r = 0, g = 0, b = 0;
  if (colorPulseWidth < 1100) {
    r = 255; g = 0;   b = 0;    // Red
  } else if (colorPulseWidth < 1200) {
    r = 255; g = 128; b = 0;    // Orange
  } else if (colorPulseWidth < 1300) {
    r = 255; g = 255; b = 0;    // Yellow
  } else if (colorPulseWidth < 1400) {
    r = 0;   g = 255; b = 0;    // Green
  } else if (colorPulseWidth < 1500) {
    r = 0;   g = 255; b = 255;  // Cyan
  } else if (colorPulseWidth < 1600) {
    r = 0;   g = 0;   b = 255;  // Blue
  } else if (colorPulseWidth < 1700) {
    r = 75;  g = 0;   b = 130;  // Indigo
  } else if (colorPulseWidth < 1800) {
    r = 148; g = 0;   b = 211;  // Violet
  } else {
    r = 255; g = 255; b = 255;  // White
  }
```
