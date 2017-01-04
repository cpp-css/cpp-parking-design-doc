#CPP Parking Status Indicator (PSI)

##Abstract
Currently, Cal Poly Pomona parking has no live centralized online service showing the availability of parking spaces in all parking lots. This presents a problem as students waste time driving around trying to find parking. We propose to create this service through a 3-component software application, and in doing so, reduce congestion and average time taken to find parking.

##Architecture
The three primary components are:

1. [Ingress/Egress Monitors](https://github.com/cpp-css/cpp-parking-computer-vision)
	
	*  These are the Raspberry Pis running our computer vision python scripts. 

	*  They will monitor incoming and outgoing cars, and post updates to the PSI Backend.


2. [PSI Backend](https://github.com/cpp-css/cpp-parking-backend)

	*  This is our server, holding all of the centralized state (i.e. number of cars per lot).

	*  The backend should expose http endpoints that allow the Ingress/Egress Monitors to update state (with authentication - so random people can't alter our traffic data), as well endpoints that allow clients to query current state.

	*  The backend should be independent of the implementation of the Ingress/Egress Monitors and the frontend clients. This way, in the future, we can potentially swap out the Ingress/Egress Monitors with other sensors (i.e. magnetic bars on the entrances) or the frontend without having to change any code in the backend.

	*  Horizontal scaling can be achieved with multiple backends multiplexed under a load balancer. If so, all backends can be refactored to depend on a common single redis instance.

	*  The backends can be hosted on Cal Poly servers, or in AWS. 

3. [PSI Frontend](https://github.com/cpp-css/cpp-parking-frontend)

	*  This is the GUI that the user sees. At first we plan on building a webapp, since it should work on all platforms/devices. In the future, we may also create Android/iOS apps. 

##Architecture Diagram

Note: diagrams were created with [draw.io](https://draw.io). Our diagrams are stored [here](https://drive.google.com/file/d/0B5urvZjIEkRkOFJhd2ZyYWFwc2c/view?usp=sharing).

###Single Backend:

![Single Backend](SingleBackendArchitecture.png)


###Multiple Backend:

![Multiple Backend](MultipleBackendArchitecture.png)


##Implementation

Every single parking lot has entrances and exits. An Ingress/Egress Monitor is a Python script running on a [Rasberry Pi](https://www.raspberrypi.org/) equipped with a camera stationed at the entrance and exit of each lot. We use [OpenCV](http://opencv.org/) image subraction libraries on an incoming video stream to determine the presence of cars in the video. The car objects can then be dilated into blobs, which are then processed by an OpenCV blob-tracking library. As their centroid passes through a line marking the entrance/exit, we increment/decrement a local counter within the script. Every few minutes, the counter's value is sent as the JSON payload of an HTTP Post to the appropriate lot endpoint of the PSI Backend, and the local counter's value is reset to 0. 

An eventual goal would be to make these update requests require authentication, such that only the Ingress/Egress Monitors can successfully affect the state of our backend.

Although OpenCV has Java bindings, [Python](http://docs.opencv.org/3.0-beta/doc/py_tutorials/py_tutorials.html) makes the OpenCV development much easier and more concise.



The PSI Backend takes the value sent in the HTTP Post and updates a counter associated with the lot in a threadsafe manner. If we have multiple backends, we can fall back to [Redis](https://redis.io/) as the data store for its threadsafe synchronized counters. 

The Backend also exposes endpoints for clients to request the state of any lot. We return a JSON object conveying the current number of cars __x__ in the lot, as well as its maximum capacity. We do not guarantee that __x__ is less than or equal to the maximum capacity, as it is possible for cars to enter the lot while it is full (as they search the lot in vain). 

An eventual goal would be to support websocket endpoints so that updates can be streamed to clients, so that clients don't have to constantly poll the server.  

Since we have a stretch goal of supporting potentially thousands of websocket connections and want as many CPP students to contribute as possible, we are considering the [Play!](https://www.playframework.com/) framework, since it supports Java, while also providing a robust, scalable [Actor-websocket model](https://www.playframework.com/documentation/2.5.x/JavaWebSockets).

By having the backend only support http endpoints that return the status of lots in JSON (without any html server-side rendering) we can easily divide the responsibilty of frontend and backend. The frontend can be an [Angular](https://angularjs.org/) webapp, an Android app, or an iOS app. As long as our backend provides a consistent interface, we should be able to build any sort of frontend on top of it.



##Open Issues/Assumptions

*  The projected number of cars in each lot should vary from the real number by a small percentage. We aim to determine that percentage over the course of our testing. 
*  Each Raspberry Pi must have access to power and wifi (hopefully with the help of the transportation committee?). 
*  Initially, we assume clear weather and daylight conditions.
*  We assume no network outages or Pi failures. 
*  We may have to reset the state of the backend, if there is a strong accumulated differential between projected and actual number of cars in lots. Need testing to see.
*  We assume we can take a simple ratio of width to height to classify car object blobs. If this proves to be inaccurate, we can also look into [Haar-like features](https://en.wikipedia.org/wiki/Haar-like_features).
*  We plan to eventually record statistics on number of cars in parking lots through the day, week, month, and quarter of Cal Poly Pomona to discover long term parking trends. 