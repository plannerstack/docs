Plannerstack is a travel planning platform that combines data from (almost) all Dutch public transport (trains, buses, trams, etc) providers, to provide the most complete public transport data services.

# Plannerstack documentation

Welcome to the Plannerstack documentation! Good to have you here!

To get you kickstarted, first a quick introduction to the world of Plannerstack:

0. In the Netherlands we have a lot of Busses, trains, trams driving around continuously, just check out [ovradar](http://ovradar.nl) for how many there are currently on the move.
0. In order to give any developer the ability to create new apps we provide a set of coherent real-time API's. For this we first need to combine different sources of information:
	0. All the roads and walk areas of the Netherlands based on [OpenStreetMap](http://openstreetmap.org) Data so we know where vehicles can drive and where you can walk or bike
	0. All the static different schedules of the different agencies, integrated into one open standard: [GTFS](https://developers.google.com/transit/gtfs/)
	0. All the dynamic real-time schedules (based on those real-time positions) of the different agencies, all integrated into another open standard: [GTFS-RT](https://developers.google.com/transit/gtfs-realtime/)
0. Next to enabling developers to plan multi-modal and in real time, Plannerstack also offers all of this raw, integrated data separately, enabling others to also build on top of that.

### The Dutch OpenTripPlanner

The most easy step to enter our world is first to start with the highest layer: the trip planner that offers all the different data in a bite-sized and integrated way. After that, it's a matter of pealing of the layers to find out what data is behind it all!

The main trip planner API offered by Plannerstack is based on OpenTripPlanner (OTP), an open source multi-modal trip planner, which runs on Linux, Mac, Windows, or potentially any platform with a Java virtual machine. More background information can be found here: http://opentripplanner.readthedocs.org/en/latest/ 

It’s good to know that OpenTripplanner consists of a Java backend for routing that is accessible through a REST API and a javascript demo client. You can see the latter in action on: http://demo.planner.plannerstack.org

Open up your javascript console there and you'll see the actual REST-calls going out to the OTP server. The REST API endpoint for the planner is served on /otp/routers/default/plan [API documentation](http://dev.opentripplanner.org/apidoc/0.15.0/resource_PlannerResource.html).

An example URL towards this endpoint is:     

* http://demo.planner.plannerstack.com/otp/routers/default/plan?fromPlace=51.923943445544715%2C4.4659423828125&toPlace=52.38901106223458%2C4.9658203125&mode=TRANSIT%2CWALK&maxWalkDistance=3000&arriveBy=false&wheelchair=false

First things first: don't worry. While visting this URL will give you back XML, it's just possible to get the result back in JSON by setting the correct request headers.

Second: a quick explanation about the url and its parameters:

```
http://demo.planner.plannerstack.com                // the demo planner root URL. Production: planner.plannerstack.com
  /otp/routers/default/plan?                        // the plan resource, OTP has many more to discover (lists of routes, etc)
  &fromPlace=51.923943445544715%2C4.4659423828125   // the from location, as a geo point, Plannerstack also offers a geocoder! (BAG)
  &toPlace=52.38901106223458%2C4.9658203125         // the to location, also a geo point
  &mode=TRANSIT%2CWALK                              // the modes to travel on, a lot of possibilities :)
  &maxWalkDistance=3000                             // How far is the traveller willing to walk, best result when not too low
  &arriveBy=false                                   // Whether to arrive by the to-location, or depart from the from-location
  &wheelchair=false                                 // Accesibility, all the way!
```

Missing here are the date and time parameters, on purpose, otherwise this example would become old pretty soon. Sending along the date and the time can be done in multiple ways, easiest:
```
    &date=10-30-2015&time=22%3A16
```

Again, check out the documentation of the plan resource in the [API documentation](http://dev.opentripplanner.org/apidoc/0.15.0/resource_PlannerResource.html) of OTP for more options. Or (it's open source anyway), check out the [actual code](https://github.com/opentripplanner/OpenTripPlanner/blob/master/src/main/java/org/opentripplanner/api/common/RoutingResource.java) behind the endpoint.

Again, this planner is just a top of the iceberg, go through this documentation for all the other possibilities or join us on our slack channel by visiting: http://slack.plannerstack.org

