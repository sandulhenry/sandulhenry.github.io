---
layout: page
title: Yonder Dynamics
description: Undergraduate Student Org
img: assets/img/rover_cover.JPG
importance: 1
category: work
related_publications: false
---

## What is Yonder Dynamics?

[Yonder Dynamics](https://yonderdynamics.org/) is an undergraduate student engineering organization at UC San Diego. We compete in the University Rover Challenge, an event organized by The Mars Society. The process in this event consists of a Preliminary Design Report at the start of the year, a System Acceptance Review in February, and if accepted, the event in late-May. Yonder Dynamics consistently passes the SAR and attends the competition.

## My Role

In my first year on the team, I was a part of the Embedded Systems subteam. On my second year, I was a Software Lead, with my roles expanding beyond just embedded systems, including providing leadership and direction to the Autonomous and Frontend team. In my third year, I continued my leadership role.

## My Projects

- Joint-by-Joint Control of 6-DOF Arm
- Planar Inverse Kinematics
- Modded CAN Bus Hardware Abstraction Layer
  - ODrive, a BLDC motor controller, provides an abstraction layer in the form of a [ROS2 node](https://docs.odriverobotics.com/v/latest/guides/ros-package.html). Its utility is in how it provides abstraction in the form of ROS2 messages (state of the motor, state of the controller, input to the system), allowing easy monitoring and integration, without directly interfacing with the CAN Bus.
  - A shortcoming in the ODrive controller is in accessing arbitrary parameters. These can include more niche things that wouldn't be needed in implementation, like raw encoder values. These are easy to access through the Web GUI, but not trivial to access when your interfacing method is over a CAN bus. The method for accessing these parameters is by sending a specifically loaded CAN frame, and watching and decoding the response. The issue leading to this is that the means to do this don't exist in ODrive's ROS2 abstraction node.
  - Our need was from the desire to do arbitrary parameter access (APA) for raw encoder values from a RS485 encoder for how we were doing complete inverse kinematics. My solution was to mod the code, using a RCLCPP wall clock timer to trigger releasing a CAN frame for the 6 controllers and catching and decoding it. I added the necessary field to the `controller_status.msg` message.
  - As we continued, it became clear that we wanted to APA more than just this value. The underlying question is how can you efficiently communication the data you get, for any number of parameters you access? We created a `vector` of `rclcpp::GenericPublisher`, who's main purpose is to contain publishers whose type is _not known at compile time_. A separate function checks if a given topic info (from a generate list of IDs for a given firmware) exists, and creates and appends it if not. A ROS2 service is used to request APA. At reception of the known response ID (all CAN frames on this bus follow the ODrive docs), a publisher is got or created, and the information is published.
  - Future improvements including bringing all the controllers up to the same firmware, so that you don't need to keep track of the endpoint IDs that vary by firmware and board model. Neil made a JSON-to-cpp code generator that converts the endpoint IDs to a very long cpp enum class.
- Sandy's Super Useful Utilities
  - _An ongoing project, these are some standalone quality of life upgrades meant to make development and use easier._
  - Local Access Point:
    - For moving the rover very short distances, such as from our workspace to the soccer field to test, or even to the loading dock to put in a car, it is a little bit of a pain to control. You would need to carry a spool of ethernet and carry a laptop, or you need to carry the base station and connect over a point-to-point AP on the base station's network.
    - This project makes use of a software AP of a usb wifi card, to create a local hotspot and connect to the rover that way.
    - Due to the nature of this AP having low bandwidth, I added the ability to remove camera streams to avoid wasting it, as the intention is that you are using it accompany it within arm's range.
    - I started a cronjob that checks for connected clients 2 minutes after boot, and disables the AP if nothing is found.
- Drill Apparatus Control
