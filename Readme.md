#CPP Parking Status Indicator

The Status Indicator is an application for monitoring parking availability in each of the lots at Cal Poly Pomona. The motivation for this application is to provide up-to-date awareness of all parking lot statuses, so that students don't need to aimlessly drive between lots hoping to find an open spot. The hope is that this will reduce congestion and overall time needed to find parking.

CPP Parking Status Indicator provides a centralized backend for all live parking information, including the parking structures that already have LED counters. The application interface shall be specified in this repository's [design doc](design.md).


##Implementation Methodology

One means of determining the number of parking spots available per lot would be to place a sensor on each parking spot. This would be ideal, as we could get a global view of the lot with accurate information. However, the cost of the sensors and the labor to install them would be prohibitively high.

Instead we count the number of cars entering and exiting each lot using a camera placed at every entrance/exit of each lot. We can use computer vision software to determine when cars pass the boundary of the lot. This solution only requires a camera, and a computer capable of running computer vision software mounted at each entrance/exit. The Raspberry Pi fulfills this role cheaply, and Cal Poly Computer Science Society will build the computer vision software. 


##Milestones
A one-entrance lot.
Etc...


##Assumptions
Each Raspberry Pi must have access to power and wifi. Initially, we also assume clear weather and daylight conditions. 
