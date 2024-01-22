---
layout: distill
title: Fuzzy-Logic speed controller using an ultrasonic sensor
date: 2024-01-21 16:40:16
description: It's Fuzzy
tags: Fuzzy-Logic, RaspberryPi, Python
categories: portfolio
thumbnail: assets/video/Fuzzy-Logic speed controller/FuzzyLogic_FinalProject_Test2-(720p30).mp4
github_repo: EOC-dev/FuzzyLogicSpeedController_V1
---

For my Fuzzy Logic undergrad class we were required to do a project. This was a fairly open ended design project which included both a MATLAB simulation section as well as a hardware implentation section, and was a collaborative effort between me and my classmate **Cooper Burns** (insert LinkedIn link here). I mainly dealt with the hardware section, which I will describe in this article. 

## The Plan ##

The plan for this project was to design a fuzzy logic controller such that, while approaching an object (a wall, box, or any object detectable by an ultrasonic sensor), a robot’s speed would decrease based on the distance to that object, getting slower and slower as it gets closer until it stops a set distance away.

## Hardware Setup ##

The hardware for this project was based on [this](https://osoyoo.com/2020/03/01/use-raspberry-pi-to-control-mecanum-omni-wheel-robot-car/) osoyoo robot kit, but generally to do this project one would need:

| Hardware        |      We Used      |
| ----------------- | :--------------: |
| Single Board Computer     | RaspberryPi 4 |
| Motor Controller      |   Osoyoo L298n based module    |
| Geared DC Motors (x2) |   osoyoo Motors     |
| Ultrasonic Sensor |   HC-SR04    |
| Robot Chassis |   Osoyoo chassis    |

The robot was setup according to the instructions [here](https://osoyoo.com/2020/03/01/use-raspberry-pi-to-control-mecanum-omni-wheel-robot-car/), but fundamentally the hardware translates to the following control diagram:

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/Fuzzy-Logic speed controller/ControlFlowDiagram.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

## Software Setup and code walk through ##

After the hardware was setup, the software was created to implement the control scheme using python -- a setup guide for which is provided on the [github](https://github.com/EOC-dev/FuzzyLogicSpeedController_V1) page for this project.

Below is the state flow diagram for the main python program:

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/Fuzzy-Logic speed controller/FuzzyMechanumStateFlow.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

First, the GPIO pins are setup using the [RPI.GPIO](https://sourceforge.net/p/raspberry-gpio-python/wiki/Examples/) library

```python
# GPIO pin assignments for motor control

#Osoyoo used an l298n based motor driver which requires 3 pins per motor: a PWM(analog) signal pin to control the speed of the motor (the EN pins), and two digital logic pins to control the direction of the motor using an H-bridge circuit.
IN1Rear, IN2Rear = 16, 18
IN3Rear, IN4Rear = 13, 15 
ENA, ENB = 12, 33

#front Wheels were not used do to a hardware problem with the osoyoo motor x driver
IN1Front, IN2Front = 40, 38
IN3Front, IN4Front = 36, 32

#Setting up pins for the ultrasonic sensor
TRIGGER_PIN, ECHO_PIN = 31, 37

# Initialization - setting up GPIO pins as input/output. For more info, please read the GPIO docs: https://sourceforge.net/p/raspberry-gpio-python/wiki/BasicUsage/
GPIO.setmode(GPIO.BOARD)
GPIO.setup([IN1Rear, IN2Rear, IN3Rear, IN4Rear, ENA, ENB, IN1Front, IN2Front, IN3Front, IN4Front, TRIGGER_PIN, ECHO_PIN], GPIO.OUT)
GPIO.output([ENA, ENB], True)
GPIO.setup(ECHO_PIN, GPIO.IN)

# Motor control functions (e.g., go_ahead, turn_left, etc.)
# Each function corresponds to a motor action, these defs were mainly taken from the provided osoyoo tutorial series for the robot kit used in this project: https://osoyoo.com/driver/mecanum/mecanum-pi.py

#make rear right motor moving forward
def rr_ahead(speed):
    GPIO.output(IN1Rear,True)
    GPIO.output(IN2Rear,False)

    #ChangeDutyCycle(speed) function can change the motor rotation speed
    #rightSpeed.ChangeDutyCycle(speed)

#make rear left motor moving forward    
def rl_ahead(speed):  
    GPIO.output(IN3Rear,True)
    GPIO.output(IN4Rear,False)
    #leftSpeed.ChangeDutyCycle(speed)

def go_ahead(speed):
    rl_ahead(speed)
    rr_ahead(speed)
#     fl_ahead(speed)
#     fr_ahead(speed)
    GPIO.output(IN1Front,False) #Since we are just using motors that use motor pi controller, the rear wheels are disabled
    GPIO.output(IN2Front,False) #Sadly, only motors controlled using motor pi can have the pwm signal modified
    GPIO.output(IN3Front,False)
    GPIO.output(IN4Front,False)

# Setup for PWM speed control
#following code only works when using Model-Pi instead of Model X motor driver board which can give raspberry Pi USB 5V power
#Initialize Rear model Pi board ENA and ENB pins, tell OS that ENA,ENB will output analog PWM signal with 1000 frequency
rightSpeed = GPIO.PWM(ENA, 1000)
leftSpeed = GPIO.PWM(ENB, 1000)
rightSpeed.start(0)
leftSpeed.start(0)
```
Next, the code defines the function for the Ultrasonic Sensor function:

```python
# Ultrasonic distance measurement function. I go into more detail on the concepts behind this code on the corresponding blog entry for this project:
def DistMeasure():
    # set Trigger pin to high to send out pulse
    GPIO.output(TRIGGER_PIN, True)
 
    # set Trigger pin to low after 0.01ms
    time.sleep(0.00001)
    GPIO.output(TRIGGER_PIN, False)
 
    TimeSent = time.time()
    TimeRecieved = time.time()
 
    # save time the pulse was sent
    while GPIO.input(ECHO_PIN) == 0:
        TimeSent = time.time()
 
    # save time the reflected pulse was recieved
    while GPIO.input(ECHO_PIN) == 1:
        TimeRecieved = time.time()
 
    # difference between timestamp sending the pulse and timestamp recieving the reflected wave back
    ElapsedTIme = TimeRecieved - TimeSent

    # Solving for distance using the classic speed = dist/time equation. Speed is speed of sound (343 m/s so 34300 cm/s), and we divide by two here to account for our time calculation being the time for the pulse to hit an object and then return -- whereas we just want the time to travel to the object.
    distance = (ElapsedTIme * 34300) / 2
 
    return distance
```

Essentially whats happening here is an extrapolation of the classic physics formula:

$$
\text{speed} = \frac{\text{distance}}{\text{time}}
$$

Whereas in our case we use the ultrasonic sensor to send out a sound pulse when the TRIGGER pin is set high, then record the time when it recieves it and the ECHO pin is high. The speed of sound is a constant, and thus our equation becomes:

$$
\text{distance} = \frac{\text{(TimeSignalRecieved - TimeSignalSent)} \times 34300}{2}
$$

The code then defines the function for the Fuzzy Logic controller function:

```python
# Fuzzy inferencing system that takes in the distance calculated from the ultrasonic sensor, and uses it to determine what speed to set motors to.
def get_speed_value(dist):
    # Define universe variables
    distance = ctrl.Antecedent(np.arange(0, 40, 1), 'distance')
    speed = ctrl.Consequent(np.arange(0, 100, 1), 'speed')

    # Define fuzzy membership functions
    distance['close'] = fuzz.trimf(distance.universe, [0, 0, 20])
    distance['medium'] = fuzz.trimf(distance.universe, [10, 20, 30])
    distance['far'] = fuzz.trimf(distance.universe, [20, 40, 40])

    speed['slow'] = fuzz.trimf(speed.universe, [0, 0, 40])
    speed['medium'] = fuzz.trimf(speed.universe, [30, 50, 70])
    speed['fast'] = fuzz.trimf(speed.universe, [60, 100, 100])

    # Define fuzzy rules
    rule1 = ctrl.Rule(distance['close'], speed['slow'])
    rule2 = ctrl.Rule(distance['medium'], speed['medium'])
    rule3 = ctrl.Rule(distance['far'], speed['fast'])

    # Create control system
    speed_ctrl = ctrl.ControlSystem([rule1, rule2, rule3])

    # Create a control system simulator
    speedL = ctrl.ControlSystemSimulation(speed_ctrl)

    # Set inputs
    speedL.input['distance'] = dist

    # Calculate results
    speedL.compute()

    FuzzySpeed = int(speedL.output['speed'])
    
    return FuzzySpeed
```
A detailed explanation of the principles behind how fuzzy inferencing works was beyond the scope of this short portfolio article,<d-footnote> If the reader is unfamiliar with how fuzzy logic inferencing works, here is a great series of videos by the great Brian Douglas on the topic: </d-footnote> but essentially the function starts by establishing two fuzzy inference sets (FIS), one for distance and the other for speed. These sets interpret the sensor's output levels, translating them into more fuzzy terms like "close" or "far." Likewise for the concept of speed (expressed via pwm values) -- mapping these values to fuzzy concepts like "slow or "fast."


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/Fuzzy-Logic speed controller/DistFuzzySet" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/Fuzzy-Logic speed controller/SpeedFuzzySet" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

Once the two inference sets are defined, the function uses triangular membership functions to map these qualitative descriptors to the sensor's numerical data. In this context, 'distance' and 'speed' are treated as fuzzy variables, each associated with sets like 'close', 'medium', and 'far' for distance, and 'slow', 'medium', and 'fast' for speed. These fuzzy sets are then used to formulate rules that govern the system's behavior. For instance, one rule states that IF the distance is 'close', THEN the speed should be 'slow'. This rule-based approach enables the system to map the distance we recieve from the ultrasonic sensor to a PWM signal to control the motors.

The last part of the code defines the main control loop, which calls the MeasureDist function and pipes the distance recieved into our fuzzy logic controller function to obtain the corresponding PWM which will be used to drive our system. This code runs in a loop until a user interrupts the program with a keyboard interrupt.

```python
# Continuous loop for processing and control
try:
    while True:
        UltraDist = DistMeasure()  # Measure distance
        print("Object distance = " + str(UltraDist) + " cm")
        FuzzSpeedOut = get_speed_value(UltraDist)  # Process distance through fuzzy logic
        print("\n Fuzzy Speed is: " + str(FuzzSpeedOut))
        go_ahead(10)  # initialize a speed for the motor
        rightSpeed.ChangeDutyCycle(FuzzSpeedOut) #USe the speed obtained 
        leftSpeed.ChangeDutyCycle(FuzzSpeedOut)
        time.sleep(0.1)

# Shutdown - cleanup on interrupt (CTRL+C)
except KeyboardInterrupt:
    print("User ended program")
    GPIO.cleanup()
```

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

