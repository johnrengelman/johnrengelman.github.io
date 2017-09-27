---
layout: page
title: ETLS 509 - V&V (Fall 2017) Drone Collision & Avoidance System RFP
---

## Summary
The system is an add-on system to commercial drone units to detect dangerous collision situations and alert the drone operator with sufficient time to affect flight path changes. The systems consists of 2 components that should be capable of being retrofitted to a significant portion of commercial drone systems and control units.

## Scope
The collision detection and avoidance system (CDAS) consists of 2 primary components: the drone mounted detection system, and the operator console mounted alert add-on. The system consists of both components in a single sale unit. The installation, operation, and maintenance of the entire system shall be conducted by the purchasing user of the system.

The detection system shall be mounted to existing drone setups by a standard bracket and clamp system. No modifications to the drone (e.g. drilling, sawing, etc.) shall be made for installation purposes. The detection module shall interface with the drone through the exposed expansion ports of the onboard microcomputer. The expansion ports consist of the general purpose input/output (GPIO), the inter-integrated circuit (I2C), and the universal asynchronous receiver-transmitter (UART). Any or all of the expansion ports may be used by the the detection system. Power for the detection system shall be obtained via the GPIO port, which can typically supply 5V power. The detection system may use either the drone’s or its own onboard transmitter to send alerts to the operator.

The detection system shall be capable of detecting a collision situation and notify the operator of the situation. The minimum amount of time between detection and collision shall be 6 second. The system shall be capable of detecting collision situations with static, non moving objects in the forward flight path of the drone. Additionally, the system shall be capable of detecting collision scenarios with other dynamic (e.g. moving) objects in the vicinity of the drone. The system shall be capable of tracking at minimum 5 potential collision objects at a time. The detection system may operate in either a directional or omnidirectional mode. The omnidirectional mode may decrease the the detection time by 50%. The system shall be capable of detecting objects from on-plane (0 degree) to 45 degree beneath plane of the drone. The detection system shall be a maximum of 1 kg in total weight including the mounting kit.

The detection system shall transmitter potential collision detections to the operator via the console add-on panel. The panel shall notify the operator via visual and audible alerts of the potential collision and the relative direction of the collision in relation to the current flight path. The alert and collision direction indicator shall provide the operator with sufficient information to affect a course correction to avoid collision.

The system shall adhere to all applicable FAA and FCC regulations.
