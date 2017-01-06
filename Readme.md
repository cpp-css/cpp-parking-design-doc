#CPP Parking Status Indicator (PSI)

The Parking Status Indicator (PSI) is a centralized service for live monitoring of parking availability in Cal Poly Pomona parking lots.


##Abstract
Currently, Cal Poly Pomona parking has no live centralized online service showing the availability of parking spaces in all parking lots. This presents a problem as students waste time driving around trying to find parking. We propose to solve this problem by creating the Parking Status Indicator, a 3-component webservice that will reduce congestion and average time taken to find parking.

##Background

One means of determining the number of parking spots available per lot would be to place a sensor on each parking spot. This would be ideal, as we could get a global view of all lots with accurate information. However, the cost of the sensors and the labor to install them is prohibitively high. For example, [even small sensors can cost between $200 and $400 each](http://www.popularmechanics.com/technology/gadgets/a6528/smart-parking-systems-steer-drivers-to-open-spaces/).

Instead we can count the number of cars entering and exiting each lot using a camera placed at every entrance/exit of each lot. Using computer vision, we can determine when cars pass the boundary of the lot. This solution only requires a camera, and a computer capable of running computer vision software mounted at **each entrance/exit** instead of each parking space. The Raspberry Pi fulfills this role cheaply, and Cal Poly Pomona's Computer Science Society can build the computer vision software. 


##Architecture
The three primary components are:

1. [Ingress/Egress Monitors](https://github.com/cpp-css/cpp-parking-computer-vision)
	
	*  These are the Raspberry Pis running our computer vision python scripts. 

	*  They will monitor incoming and outgoing cars, and post updates to the PSI Backend.


2. [PSI Backend](https://github.com/cpp-css/cpp-parking-backend)

	*  This is our server, holding all of the centralized state (i.e. number of cars per lot).

	*  The backend should expose http endpoints that:
		*  allow the Ingress/Egress Monitors to update state (with authentication - so random people can't alter our traffic data)
		*  allow clients to query current state.

	*  The backend should be independent of the implementation of the Ingress/Egress Monitors and the frontend clients. This way, in the future, we can potentially swap out the Ingress/Egress Monitors with other sensors (i.e. magnetic bars on the entrances) or the frontend without having to change any code in the backend.

	*  Horizontal scaling can be achieved with multiple backends multiplexed under a load balancer. If so, all backends can be refactored to depend on a single data store.

3. [PSI Frontend](https://github.com/cpp-css/cpp-parking-frontend)

	*  This is the GUI that the user sees. At first we plan on building a webapp, since it should work on all platforms/devices. In the future, we may also create Android/iOS apps. 

##Architecture Diagram

Note: diagrams were created with [draw.io](https://draw.io). Our diagrams are stored [here](https://drive.google.com/file/d/0B5urvZjIEkRkOFJhd2ZyYWFwc2c/view?usp=sharing).

###Single Backend:

![Single Backend](images/SingleBackendArchitecture.png)


###Multiple Backend:

![Multiple Backend](images/MultipleBackendArchitecture.png)


##Implementation

Every single parking lot has entrances and exits. An Ingress/Egress Monitor is a Python script running on a [Rasberry Pi](https://www.raspberrypi.org/) equipped with a camera stationed at the entrance and exit of each lot. We use [OpenCV](http://opencv.org/) [image subraction libraries](http://docs.opencv.org/trunk/db/d5c/tutorial_py_bg_subtraction.html) on an incoming video stream to determine the presence of cars in the video. The car objects can then be dilated into blobs, which are then processed by an [OpenCV blob-tracking library](https://www.learnopencv.com/blob-detection-using-opencv-python-c/). As their [centroid passes through a line marking the entrance/exit](https://github.com/andrewssobral/simple_vehicle_counting), we increment/decrement a local counter within the script. The counter measures the net number of cars that have entered the lot through the currently monitored entrance/exit. Every few minutes, the counter's value is sent as the JSON payload of an HTTP Post to the appropriate lot endpoint of the PSI Backend, and the local counter's value is reset to 0. 

*  An eventual goal would be to make these update requests require authentication, such that only the Ingress/Egress Monitors can successfully affect the state of our backend.

*  Although OpenCV has Java bindings, [Python](http://docs.opencv.org/3.0-beta/doc/py_tutorials/py_tutorials.html) makes the OpenCV development much easier and more concise.


The PSI Backend takes the value sent in the HTTP Post and updates a counter associated with the lot in a threadsafe manner. If we have multiple backends, we can fall back to [Redis](https://redis.io/) as the data store for its threadsafe synchronized counters. 

The Backend also exposes endpoints for clients to request the state of any lot. We return a JSON object conveying the current number of cars __x__ in the lot, as well as its maximum capacity. We do not guarantee that __x__ is less than or equal to the maximum capacity, as it is possible for cars to enter the lot while it is full (as they search the lot in vain). 

*  An eventual goal would be to support websocket endpoints so that updates can be streamed to clients, so that clients don't have to constantly poll the server.  

*  Since we have a stretch goal of supporting potentially thousands of websocket connections and want as many CPP students to contribute as possible, we are considering the [Play!](https://www.playframework.com/) framework, since it supports Java, while also providing a robust, scalable [Actor](http://akka.io/)-[websocket model](https://www.playframework.com/documentation/2.5.x/JavaWebSockets).


By having the backend only return JSON (not html), we can easily divide the responsibilty of frontend and backend. The frontend can be an [Angular](https://angularjs.org/) webapp, an Android app, or an iOS app. As long as our backend provides a consistent interface, we should be able to build any sort of frontend on top of it.



##Open Issues/Assumptions

*  The projected number of cars in each lot may vary from the real number by a small percentage. We aim to determine that percentage over the course of our testing. 
*  Each Raspberry Pi must have access to power and wifi (hopefully with the help of the transportation committee?). 
*  Initially, we assume clear weather and daylight conditions.
*  We assume no network outages or Pi failures. 
*  We may have to reset the state of the backend, if there is a strong accumulated differential between projected and actual number of cars in lots. Need testing to see.
*  We assume we can take a simple ratio of width to height to classify car object blobs. If this proves to be inaccurate, we can also look into [Haar-like features](https://en.wikipedia.org/wiki/Haar-like_features).
*  We plan to eventually record statistics on number of cars in parking lots through the day, week, month, and quarter of Cal Poly Pomona to discover long term parking trends. 


##Cost


The major costs for this project are the Raspberry Pis, cameras, and associated accessories (such as cases).

###Pilot Program

For testing a single entrance on a single lot in our pilot program we need:

*  [Raspberry Pi 3 Kit](https://www.amazon.com/CanaKit-Raspberry-Clear-Power-Supply/dp/B01C6EQNNK/ref=sr_1_3?ie=UTF8&qid=1483666861&sr=8-3&keywords=raspberry+pi+3): $50


*  [Rasberry Pi Camera](https://www.amazon.com/Arducam-Megapixels-Sensor-OV5647-Raspberry/dp/B012V1HEP4/ref=sr_1_2?ie=UTF8&qid=1483666978&sr=8-2&keywords=raspberry+pi+3+camera): $15
	

*  [Raspberry Pi Camera Case](https://www.amazon.com/TEK-CAM3-Raspberry-Camera-Module-Black/dp/B01H1BIE2Q/ref=sr_1_1?ie=UTF8&qid=1483696074&sr=8-1&keywords=raspberry+pi+camera+case&refinements=p_72%3A2661618011): $22 OR [Raspberry Pi Camera Case](https://www.amazon.com/Latest-Raspberry-Camera-Case-Megapixel/dp/B00IJZJKK4/ref=pd_sbs_147_2?_encoding=UTF8&pd_rd_i=B00IJZJKK4&pd_rd_r=XHJ7AJT5H7DX8DB0HDR9&pd_rd_w=ZhddD&pd_rd_wg=oGNhW&psc=1&refRID=XHJ7AJT5H7DX8DB0HDR9): $8.50


*  [Raspberry Pi Battery](https://www.amazon.com/Intocircuit-11200mAh-Portable-Charger-External/dp/B00BB5GR0A/ref=sr_1_1?ie=UTF8&qid=1483667604&sr=8-1&keywords=intocircuit+11200): $22.99


**Our pilot program requested budget is (50 + 15 + 22 + 8.5 + 23) * 1.10 ~= $135**


###Final Estimated Cost

For the finished service, we propose one raspberry pi 3 with a camera and case per parking lot entrance and exit. 

**This is up to (50 + 15 + 22) * 1.10 ~= $100 per parking lot entrance/exit.** 

**We also assume that the Transportation Committee can arrange for the construction of poles as necessary** (to place the Pis on, overlooking the entrances/exits), **the availability of power source** (for each Pi to run on), and **the availability of ethernet or wifi** (so the Pis can send their data). 

We would also need a computer for us to run the PSI Backend Server on. Fortunately, the Computer Science Department should have one available.


##Milestones (to be completed)

*  Design and Plan Finalized

*  Demo of Image Subraction to Transportation Committee

*  Testing with Pi and Camera on Cal Poly property

*  Observing the data obtained on a single entrance/exit parking lot

*  Presentation of PSI results

*  Scaling to a parking lot with multiple entrances and exits

*  Scaling to all parking lots in Cal Poly Pomona