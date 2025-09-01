---
layout: post
title: Motorized Digital Microscope
description: A custom built, sub-$150, digital microscope for use as an interactive display or in general microscopy. I was motivated to create this device as part of my continued and longstanding volunteer work for the museum of archeology, paleontology, and science (MAPS) in New Port Richey. In addition to being a museum display, this microscope serves as a prototyping and development platform for future work planned in microscopy. 
skills: 
  - SolidWorks
  - Python
  - 3D Printing
  - PCB Design
  - Product Development

main-image: /micro1.jpg
---

---
## Overview:
### Using SolidWorks, this microscope was designed to use cheap and easily accessible components, lowering costs and increasing repairability. Python was used to operate the device via Raspberry Pi Zero 2W. For the optics module, an Arducam IMX219 camera module was used and required only small focal-length adjustments to achieve sub-millimeter resolution. A small 'Hat' for the Pi Zero 2W was created to condense control system electronics.


## Finished Product On Display:
{% include image-gallery.html images="micro1.jpg" height="400" %} 

## Control System Electronics - Top:
{% include image-gallery.html images="controlTop.jpg" height="400" %}

## Control System Electronics - Underside:
{% include image-gallery.html images="controlBot.jpg" height="400" %} 

## Features:
- Gaussian diffused ‘spotlight’, following the gantry and illuminating the objective 
- Intuitive, single-input, interface 
- One-time manual calibration for autofocus 
- Compact, all-in-one, control system

## Components:
- Raspberry Pi Zero 2W 
- Arducam IMX219 Camera Module 
- 2x A4988 Stepper Drivers 
- 2x NEMA17 Stepper Motors 
- Addressable LED Strip 
- Various 3D Printed Components 
- 20x20 T-Slot Aluminum Framing 
- 2x Linear Rails 
- 2x Linear Bearings 
- Assorted Hardware
- 2x Optical Endstops
- Rotary Encoder
- 12v to 5v Buck Converter
- Mode Switch

## Code Snippet:
### Shows how leds are diffused to create a 'spotlight' look about the gantry position
```python
def update_leds_aliased(gantry_x_pos=None):
    """Update LEDs with white light and full brightness."""
    if gantry_x_pos is None:
        with position_lock:
            gantry_x = current_x_position
    else:
        gantry_x = gantry_x_pos

    for i in range(LED_COUNT):
        distance = abs(gantry_x - LED_POSITIONS[i])
        falloff = math.exp(-(distance ** 2) / (2 * LIGHT_SPREAD ** 2))
        brightness = LED_BRIGHTNESS * falloff
        strip.setPixelColor(
            i,
            Color(
                int(LED_R * brightness),
                int(LED_G * brightness),
                int(LED_B * brightness)
            )
        )
    strip.show()
```