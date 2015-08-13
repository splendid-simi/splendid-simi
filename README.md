# Park Assist
####A web application to quickly help you find the closest metered parking spot in Santa Monica, CA.

##Problem
Parking is near impossible to find in Santa Monica. In addition, the information in the Santa Monica Parking API is not being fully utilized.

##Abstract
Web app to take the decision making away from parking at meters in Santa Monica, CA. It will choose the closest confirmed open parking meter for you.

##Strategy
The Santa Monica Parking API provides information on over 6000 meters. At times over 100 events are added in less than a minute. We took this wealth of data and processed it behind the scenes using two different servers, and a client app to create a streamlined presentation of the most useful information for someone looking for a parking spot.

## Developer Documentation

####Tools Used:
* [AngularJS](https://angularjs.org/)
* [Firebase](https://www.firebase.com/)
* [Node.js](https://nodejs.org/)
* [Express](http://expressjs.com/)
* [Google Maps APIs](https://developers.google.com/maps/?hl=en/)
* [City of Santa Monica Parking Data API](https://parking.api.smgov.net/)

####To start contributing to the Park Assist codebase:
  1. Fork the repo
  2. Clone your fork locally
  3. npm install - server dependencies
  4. bower install - client dependencies
  5. gulp - run the app on a local server
  6. Visit http://localhost:8000/ on your browser

##Front End:

###Client Application Information
We loosely modeled the directory structure from the information in this article:
https://scotch.io/tutorials/angularjs-best-practices-directory-structure

```
src
├── app.js
├── directions
│   ├── directionsDisplayService.js
│   ├── directionsService.js
│   └── index.js
├── geocoder
│   ├── geocoderService.js
│   └── index.js
├── loading
│   ├── index.js
│   └── loadingService.js
├── locator
│   ├── index.js
│   └── locatorService.js
├── map
│   ├── index.js
│   ├── mapDirective.js
│   ├── mapOptions.js
│   ├── mapService.js
│   └── mapTemplate.html
├── markers
│   ├── index.js
│   ├── meterMarkerService.js
│   └── userMarkerService.js
├── modal
│   ├── index.js
│   ├── modalDirective.js
│   ├── modalService.js
│   └── modalTemplate.html
├── traffic
│   ├── index.js
│   └── trafficService.js
└── user
    ├── index.js
    └── userService.js
```
  * The main is located at src/app.js
      * All app.js does is require all module dependencies
      * There are no controllers in this app.

  * Primary functionality is split up into custom directives
    * **Map** - You will probably be most concerned with this.
    * **Modal**

  * ng services are arranged into separate directories w/ an index.js that requires the service into a module of the same name
    * **Directions** - Calculates and renders path on Google Maps.
    * **Geocoder** - Parses to Latitude/Longitude coordinates into a LatLng object w/ useful location data. Parses street address strings into LatLng objects.
    * **Loading** - Helper functions for displaying information and views during intermission (loading, waiting for directions, etc). JQuery heavy for DOM selection.
    * **Locator** - Functionality for creating user based on browser user location. Each unique user location is posted into the database as a unique user on Geolocation resolution.
    * **Map** - Map initialization, logic for setting parking spot marker when meter is found and returning an instance of the map initialized on the DOM.
    * **Markers** - Map marker methods for User and Parking Meters.
    * **Modal** - Overlay functions for when the user clicks 'Change My Destination'. JQuery heavy for DOM selection.
    * **Traffic** - Traffic layer for Google Maps. Minimal ng service ideal for studying the code base file structure.
    * **User** - Watches user position through browser Geolocation data. Heavy dependency on Directions service.

###Buttons
* **Show Me Another Spot** - Shows the user another candidate spot taken from the queue of meters.
* **Enter Another Destination** - This will change the target destination of the user and repeat steps 1-6 above with that location information.

##Back End

###Server Information

This application has two servers:

* **"Parking Spot Analyzer" Server** - responsible for choosing the meters to be sent to the client. Its logic is stored in server/server.js in the main Parking Assist repository.

* **"Cloudify" Server** - used to scrape the events from the City of Santa Monica parking API to keep the database updated. The source code for this server is located in the [dbScrape](https://github.com/splendid-simi/dbScrape) repository.

**Steps to loading up the main page and routing a user to a parking space:**

  1. The client app finds user current location and sends a get request to the PSA server.
  2. The PSA server stores the current location as a unique user in the firebase database, so that personalized parking recommendations can be stored.
  3. The PSA server pings Firebase database and iterates through all 6000+ meters stored in the database to find all within 0.2 (default setting) miles away that currently show as empty (or SE) according to the data imported in the Cloudify. The information for the meters are added to an array of meter information objects.
  4. The PSA server sorts this array of meter information objects by the distance from the user and adds this array of objects under the user information in the database.
  5. The PSA server responds to the client with the closest spot.
  6. The client app maps the location on Google Maps.

####Cloudify Server
Cloudify automatically updates the Firebase database with event information for each meter (mostRecentEvent and timeStamp fields only). We split this into a separate server so that the speed of this application would not be affected by the constantly changing meter event information. [The repo can be found here.](https://github.com/splendid-simi/dbScrape/)

### Database Information

We use Firebase to store the data.

#### Schema

#####MeteredParkingSpots
* #####MeterID
  * active - Set up with PSA server
  * latitude - Set up with PSA server
  * longitude - Set up with PSA server
  *  mostRecentEvent- Continually updated with Cloudify Server
  *  timeStamp- Continually updated with Cloudify Server


* #####Users
  * Firebase unique identifier
    * latitude - from Google Maps API
    * logitude - from Google Maps API
    * range - auto set
    * Recomendations
    * array of objects with MeterID: {active, latitude, longitude, mostRecentEvent, timeStamp} same format as MeteredParkingSpots.

* ####Application Flow
![alt text](https://github.com/rodocite/splendid-simi/blob/dev/applicationflow.jpg)

