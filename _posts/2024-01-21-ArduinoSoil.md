---
layout: distill
title: Arduino soil monitor dashboard and email watering reminder in Labview
date: 2024-01-20 19:40:15
description: It's Labview
tags: Arduino, Labview, IOT
categories: portfolio
thumbnail: assets/IMG/ArduinoSOil/Dash1.png
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

## Software Setup and code walkthrough ##

After the hardware was setup, labview was setup to communicate with arduino and take in its sensor readings -- a setup guide for which is provided on the [github](https://github.com/EOC-dev/LabviewArduinoSoilMonitor) page for this project. This project made
use fo the [Labview LINX addon](), which essentially turned the Arduino into a DAQ.

Below is the Dashboard of the completed project -- split into two parts: The first contains all the main controls of the program, and the second shows the current state machine the program is in as well as sensor output data for troubleshooting purposes:

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

The code utilizes [state machines](https://www.ni.com/en/support/documentation/supplemental/16/simple-state-machine-template-documentation.html) to control the flow of the program depending on the detected soil moisture level as well as user input. The state machine has four cases:

## Read Value (Default)
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/ArduinoSoil/State1.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
This is the state the program will start in, and will use the obtained sensor values to determine whether to loop within this case or progress to the "Send Reminder" case.

## Send Reminder
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/ArduinoSoil/State2.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
This state uses the "Send Email" expressVi to send an email to the designated user telling them their plant needs to be watered. Depending on whether the user pressed the "Send additional reminders" button, it will either enter the "Reminder Timer" case, or the "Button fallback" case.

* Reminder Timer
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/ArduinoSoil/State3.png" class="img-fluid rounded z-depth-1" style="max-width: 458px;" zoomable=true %}
    </div>
</div>
This state uses the "Elapsed time" expressVI, and subtracts from it the user defined Duration to determine the remaining time before moving forward. The Duration is entered in hours and Minutes, and needs to first be converted to seconds before being subtracted. The "remaining time" value is then used to determine whether to go to the "Send Reminder" case or stay in the present one. Theres also a failsafe to ensure that, if the moisture level falls below the threshold (i.e if the user waters the plant in this state), it will return to the "Read Value" case.

## Button Fallback
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/ArduinoSoil/State4.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
This existed purely to deal with the scenario where the user does not select "Send additional email reminders." It will stay in this case until either the moisture level falls below the threshold or the user presses the button while in this case.

Besides the 4 main cases of the state machine, the outer shell mainly consists of routing the relevant signals to the labview i/o 
and making sure the signals were reserved through successive loops.

Finally, here is a picture of the code successfully sending an email to a user:

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

1. Labview sadly has many limitations that make doing an IOT project with it very difficult. This was mainly a proof of concept, but if it seriously wanted to be taken further there would need to be some way to communicate wirelessly with the Arduino/sensor.


{% assign repo_name = page.github_repo %}
{% if repo_name %}
  {% assign repos = site.data.repositories.github_repos | where: "name", repo_name %}
  {% for repo in repos %}
    {% include repository/repo.liquid repository=repo %}
  {% endfor %}
{% endif %}

