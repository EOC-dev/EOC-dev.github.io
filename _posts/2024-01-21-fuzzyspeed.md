---
layout: post
title: Fuzzy-Logic speed controller using an ultrasonic sensor
date: 2024-01-21 16:40:16
description: It's Fuzzy
tags: Fuzzy-Logic, RaspberryPi, Python
categories: portfolio
thumbnail: assets/img/Fuzzy-Logic speed controller/FuzzyMechanumStateFlow.png
github_repo: EOC-dev/LabviewArduinoSoilMonitor
---

For my Fuzzy Logic undergrad class we were required to do a project. This was a fairly open ended design project which included both a MATLAB simulation section as well as a hardware implentation section, and was a collaborative effort between me and my classmate **Cooper Burns** (insert LinkedIn link here). I mainly dealt with the hardware section, which I will describe in this article. 

## Hardware Setup ##

The hardware for this project was based on [this](https://osoyoo.com/2020/03/01/use-raspberry-pi-to-control-mecanum-omni-wheel-robot-car/) osoyoo robot kit, but generally to do this project one would need:

| Hardware        |      We Used      |
| ------------- | :-----------: |
| Single Board Computer     | RaspberryPi 4 |
| Motor Controller      |   Osoyoo L298n based module    |
| Geared DC Motor |   osoyoo Motors     |
| Ultrasonic Sensor |   HC-SR04    |
| Robot Chassis |   Osoyoo chassis    |

The robot was setup according to the instructions [here](https://osoyoo.com/2020/03/01/use-raspberry-pi-to-control-mecanum-omni-wheel-robot-car/), but fundamentally the hardware translates to the following control diagram:

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/Fuzzy-Logic speed controller/ControlFlowDiagram.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

## Software Setup and code walk through ##

After the hardware was setup, the software was created to implement the control scheme -- a setup guide for which is provided on the [github]() page for this project.

Below is the state flow diagram for the main python program:

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/Fuzzy-Logic speed controller/FuzzyMechanumStateFlow.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

Finally, here is a video of the code working in action:

Pretty cool!

## What could be improved? ##

1. An astute observer will note a disturbing lack of hardware interrupts in the source code. This was mainly because I was still going through my Embedded Systems class and was not comfortable using hardware interrupts extensively yet. Micropython supports interrupts [natively](https://docs.micropython.org/en/latest/reference/isr_rules.html), and Circuitpython supports them [indirectly](https://learn.adafruit.com/cooperative-multitasking-in-circuitpython-with-asyncio) using the asyncio module. Both Micropython and Circuitpython are supported on the raspberry pi (see [here](https://www.raspberrypi.com/documentation/microcontrollers/micropython.html) and [here](https://learn.adafruit.com/circuitpython-on-raspberrypi-linux/overview)).
2. One may also notice from the video the crude breadboard voltage divider, this was made out of necessity as there was a mismatch between the output signal from the Ultrasonic sensor and the maximum input voltage of the Rpi's GPIO pins. While our solution worked, A more elegant solution would be to use a logic level shifter.
3. Since doing this project I have since learned about the [Embedded Fuzzy Logic Library](https://github.com/alvesoaj/eFLL), which allows emplementing fuzzy logic controllers on microcontrollers. I think it would be neat to design a fuzzy controller using the skfuzzy library, then implement it on hardware using the eFLL.


{% assign repo_name = page.github_repo %}
{% if repo_name %}
  {% assign repos = site.data.repositories.github_repos | where: "name", repo_name %}
  {% for repo in repos %}
    {% include repository/repo.liquid repository=repo %}
  {% endfor %}
{% endif %}

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/9.jpg" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/7.jpg" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    A simple, elegant caption looks good between image rows, after each row, or doesn't have to be there at all.
</div>

Images can be made zoomable.
Simply add `data-zoomable` to `<img>` tags that you want to make zoomable.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/8.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/10.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

The rest of the images in this post are all zoomable, arranged into different mini-galleries.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/11.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/12.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/7.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>