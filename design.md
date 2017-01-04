#CPP Parking Status Indicator (PSI)

##Abstract
Currently, Cal Poly Pomona parking has no live centralized online service showing the availability of parking spaces in each parking lot. We propose to create this service through a 3-component software application. 

##Architecture
The three primary components are:

1. [Ingress/Egress Monitors](https://github.com/cpp-css/cpp-parking-computer-vision)
	
	*  These are the Raspberry Pis running our computer vision python scripts. 

	*  They will monitor incoming and outgoing cars, and post updates to the PSI Backend.


2. [PSI Backend](https://github.com/cpp-css/cpp-parking-backend)

	*  This is our server, holding all of the centralized state (i.e. number of cars per lot).

	*  The backend should expose http endpoints that allow the Ingress/Egress Monitors to update state (with authentication - so random people can't alter our traffic data), as well endpoints that allow clients to query current state.

	*  In the future we should support websocket endpoints so that updates can be streamed to clients, without clients polling the server. 

	*  The backend should be independent of the implementation of the Ingress/Egress Monitors and the frontend clients. This way, in the future, we can potentially swap out the Ingress/Egress Monitors with other sensors (i.e. magnetic bars on the entrances) or the frontend without having to change any code in the backend.

	*  Horizontal scaling can be achieved with multiple backends multiplexed under a load balancer. All backends can depend on a common single redis instance, with perhaps a caching layer.

	*  Furthermore, the backends can be hosted on Cal Poly servers, or in AWS. 

3. [PSI Frontend](https://github.com/cpp-css/cpp-parking-frontend)

	*  This is the GUI that the user sees. At first we plan on building a webapp, since it should work on all platforms/devices. In the future, we may also create Android/iOS apps. 

##Architecture Diagram

Note: diagrams were created with [draw.io](https://draw.io). Our diagrams are stored [here](https://drive.google.com/file/d/0B5urvZjIEkRkOFJhd2ZyYWFwc2c/view?usp=sharing).

###Single Backend:

![Single Backend](SingleBackendArchitecture.svg)


###Multiple Backend:

![Multiple Backend](MultipleBackendArchitecture.svg)


