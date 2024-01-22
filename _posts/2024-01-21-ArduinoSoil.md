---
layout: distill
title: Arduino soil monitor dashboard and email watering reminder in Labview
date: 2024-01-20 19:40:15
description: It's Labview
tags: Arduino, Labview, IOT
categories: portfolio
thumbnail: assets/video/Fuzzy-Logic speed controller/FuzzyLogic_FinalProject_Test2-(720p30).mp4
github_repo: EOC-dev/FuzzyLogicSpeedController_V1
---

For my Electronic Instrumentation undergrad class we were required to do a project. We were given free reign on what the project could be,
as long as it was labview related. I had long wanted to do some kind of sensor project using an Arduino, and I wondered: "Could 
the arduino be interfaced with and controlled by Labview?"
The answer may shock you.

## The Plan ##

The plan for this project was to use labview to create a dashboard that would show a visual indicator when the plant needed watering,
as well as send some kind of notification to the user reminding them to water their plant.

## Hardware Setup ##

The hardware for this project was pretty simple

| Hardware        |      We Used      |
| ----------------- | :--------------: |
| Microcontroller     | Arduino UNO R3 |
| Soil moisture sensor      |   Capacitative based module    |
| Plants |   Mint (1x)     |

The soil sensor was connected to the arduino, and the arduino to a laptop via a USB cable.

## Software Setup and code walk through ##

After the hardware was setup, labview was setup to communicate with arduino and take in its sensor readings -- a setup guide for which is provided on the [github](https://github.com/EOC-dev/LabviewArduinoSoilMonitor) page for this project.

Below is the Dashboard of the completed project:

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/ArduinoSoil/Dash1.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/ArduinoSoil/Dash2.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>


Finally, here is a video of the code working in action:

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include video.liquid path="assets/video/Fuzzy-Logic speed controller/FuzzyLogic_FinalProject_Test2-(720p30).mp4" class="img-fluid rounded z-depth-1" controls=true autoplay=false %}
    </div>
</div>
<div class="caption">
    Pretty cool, huh?
</div>

<!-- Something to note is that one need not have any of the hardware to test out the code. Included in the [github]() are two files, one for using the code with hardware and one for using -->

## What could be improved? ##

1. An astute observer will note a disturbing lack of hardware interrupts in the source code. This was mainly because I was still going through my Embedded Systems class and was not comfortable using hardware interrupts extensively yet. Micropython supports interrupts [natively](https://docs.micropython.org/en/latest/reference/isr_rules.html), and Circuitpython supports them [indirectly](https://learn.adafruit.com/cooperative-multitasking-in-circuitpython-with-asyncio) using the asyncio module. Both Micropython and Circuitpython are supported on the raspberry pi (see [here](https://www.raspberrypi.com/documentation/microcontrollers/micropython.html) and [here](https://learn.adafruit.com/circuitpython-on-raspberrypi-linux/overview)). If I were to return to this project I would absolutely use interrupts.
2. One may also notice from the video the crude breadboard voltage divider, this was made out of necessity as there was a mismatch between the output signal from the Ultrasonic sensor and the maximum input voltage of the Rpi's GPIO pins. While our solution worked, A more elegant solution would be to use a logic level shifter.
3. Since doing this project I have since learned about the [Embedded Fuzzy Logic Library](https://github.com/alvesoaj/eFLL), which allows emplementing fuzzy logic controllers on microcontrollers. I think it would be neat to design a fuzzy controller using the skfuzzy library, then implement it on hardware using the eFLL.


{% assign repo_name = page.github_repo %}
{% if repo_name %}
  {% assign repos = site.data.repositories.github_repos | where: "name", repo_name %}
  {% for repo in repos %}
    {% include repository/repo.liquid repository=repo %}
  {% endfor %}
{% endif %}

