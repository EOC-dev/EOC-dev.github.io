---
layout: distill
title: OpenCV image color extractor and 
date: 2024-01-21 17:40:15
description: It's color
tags: python, OpenCV
categories: portfolio
thumbnail: assets/video/Fuzzy-Logic speed controller/FuzzyLogic_FinalProject_Test2-(720p30).mp4
github_repo: EOC-dev/FuzzyLogicSpeedController_V1
---

One of the projects I did in my undergrad years was to make a framebuffer on an FPGA using VHDL. This was challanging enough on 
its own, but one of the ancillary challenges I faced was how to load images on to the memory arrays that I had defined. I needed 
some way to take images, downsample them (so theyd fit on the limited memory FPGA), then extract that color palette and the colors
themselves into a format that could be easily loaded on to the FPGA. 

Initially I turned my attention towards [MATLABS Image Processing Toolbox](https://www.mathworks.com/products/image.html), but I
couldn't get it to work right. Essentially, the matlab script I had created kept thinking there were more colors then there were
in the image, which *really* messed things up.

Eventually, I figured out a solution in python using OpenCV. One of the [first things you do](https://docs.opencv.org/3.4/db/deb/tutorial_display_image.html) when you make an opencv script is
use the imread function. This basically takes the input image (or frames from a video feed), and converts it into a numpy array.
If that numpy array contains the rgb values of each of the pixels in the image, couldnt you loop through that array and extract 
those values into a list? Thats precisely what the following python script does:

```python

```
