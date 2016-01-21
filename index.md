# Plannerstack Kickstart documentation

### Introduction

To get you kickstarted, first a quick introduction to the world of Plannerstack:

0. In the Netherlands we have a lot of Busses, trains, trams driving around continuously, just check out [ovradar](http://ovradar.nl) for how many there are currently on the move.
0. In order to give any developer the ability to create new apps we provide a set of coherent real-time API's. For this we first need to combine different sources of information:
	0. All the roads and walk areas of the Netherlands based on [OpenStreetMap](http://openstreetmap.org) Data so we know where vehicles can drive and where you can walk or bike
	0. All the static different schedules of the different agencies, integrated into one open standard: [GTFS](https://developers.google.com/transit/gtfs/)
	0. All the dynamic real-time schedules (based on those real-time positions) of the different agencies, all integrated into another open standard: [GTFS-RT](https://developers.google.com/transit/gtfs-realtime/)
0. Next to enabling developers to plan multi-modal and in real time, Plannerstack also offers all of this raw, integrated data separately, enabling others to also build on top of that.

Join us on our Slack channel to get in direct contact. Visit: [slack.plannerstack.org](http://slack.plannerstack.org)

### Planning multi-modal routes through the Netherlands

The most easy step to enter our world is first to start with the highest layer: the trip planner that offers all the different data in an integrated and visual way. After that, it's a matter of pealing of the layers to find out what data is behind it all!

The main trip planner API offered by Plannerstack is based on OpenTripPlanner (OTP), an open source multi-modal trip planner, which runs on Linux, Mac, Windows, or potentially any platform with a Java virtual machine. More background information can be found at the [OTP ReadTheDocs](http://opentripplanner.readthedocs.org/en/latest/).

It’s good to know that OpenTripplanner consists of a Java backend for routing that is accessible through a REST API and a [javascript demo client](http://demo.planner.plannerstack.org), as pictured below.

![The OTP Demo Planner](/images/demo-planner.png "The OTP Demo Planner")

[Visit the demo client](http://demo.planner.plannerstack.com/?module=planner&fromPlace=51.923943445544715%2C4.4659423828125&toPlace=52.38901106223458%2C4.9658203125&mode=TRANSIT%2CWALK&maxWalkDistance=3000&arriveBy=false&wheelchair=false) and [open up your Browser Dev Tools](https://developers.google.com/web/tools/chrome-devtools/) to see the actual REST-calls going out to the OTP server. The REST API endpoint the planner talks to is served on `/otp/routers/default/plan` ([API documentation](http://dev.opentripplanner.org/apidoc/0.15.0/resource_PlannerResource.html)).

An example call URL towards this endpoint is:

```
# Just an example url
http://demo.planner.plannerstack.com/otp/routers/default/plan?fromPlace=51.923943445544715%2C4.4659423828125&toPlace=52.38901106223458%2C4.9658203125&mode=TRANSIT%2CWALK&maxWalkDistance=3000&arriveBy=false&wheelchair=false
```

First things first: don't worry. While visting this URL will give you back XML, it's possible to get the result back in JSON by setting the correct request header, for instance, by retrieving this url through the command line with curl it will return a JSON object:

```
# In your command line
curl --request GET \
        --header "Accept: application/json" \
        --header "Content-type: application/json" \
        "http://demo.planner.plannerstack.com/otp/routers/default/plan?fromPlace=51.923943445544715%2C4.4659423828125&toPlace=52.38901106223458%2C4.9658203125&mode=TRANSIT%2CWALK&maxWalkDistance=3000&arriveBy=false&wheelchair=false"
```

To get somewhat up to speed, a quick explanation about the url and its parameters:

```
# The example url explained
http://demo.planner.plannerstack.com                // the demo planner root URL. Production: planner.plannerstack.com
  /otp/routers/default/plan?                        // the plan resource, OTP has many more resources to discover (lists of routes, etc)
  &fromPlace=51.923943445544715%2C4.4659423828125   // the from location, as a geo point, Plannerstack also offers a geocoder! (BAG)
  &toPlace=52.38901106223458%2C4.9658203125         // the to location, also a geo point
  &mode=TRANSIT%2CWALK                              // the modes to travel on, a lot of possibilities :)
  &maxWalkDistance=3000                             // How far is the traveller willing to walk, best result when not too low
  &arriveBy=false                                   // Whether to arrive by the to-location, or depart from the from-location
  &wheelchair=false                                 // Accesibility, all the way!
```

Missing here are the date and time parameters, on purpose, otherwise this example would become old pretty soon. Sending along the date and the time can be done in multiple ways, easiest:

```
# Extra parameters for date time
&date=10-30-2015&time=22%3A16
```

Again, check out the documentation of the plan resource in the [API documentation](http://dev.opentripplanner.org/apidoc/0.15.0/resource_PlannerResource.html) of OTP for more options. Or (it's open source anyway), check out the [actual code](https://github.com/opentripplanner/OpenTripPlanner/blob/master/src/main/java/org/opentripplanner/api/common/RoutingResource.java) behind the endpoint.

*****
** This demo service is free to use. If you want more than a fair-use policy, want nightly data updates and an SLA, consider signing up for [the Dutch Multi-Modal Planner API subscription plan of plannerstack.com](https://app.moonclerk.com/pay/y22td5kfa1).**
*****


### Getting the Dutch public transport schedules

The OpenTripPlanner above makes use of a unified data-format ([GTFS](https://developers.google.com/transit/gtfs/)) that combines the different Dutch schedules available. This data is constantly kept up-to-date based on data coming in from the multitude of public transport providers in the Netherlands. This GTFS data can for example be used in combination with other geographical data to create routing graphs as pictured below.

![Graph](/images/graph.png "Graph")

Currently there are two seperate feeds for the schedules offered:

0. a GTFS feed containing the schedules from the biggest Dutch rail road provider "NS" - [download](#coming-soon) **coming soon**
0. a GTFS feed containing all other available public transport schedules - [download](#coming-soon) **coming soon**


*****
** These feeds are free to download. If you would like more than a fair-use policy on the amount of times it can be downloaded, consider signing up for [the GTFS & GTFS-RT subscription plan of plannerstack.com](https://app.moonclerk.com/pay/1vmtqc1df9e).**
*****


### Hooking into the Dutch real-time public transport data streams

Real-time streams about last-minute updates to schedules are offered in the [GTFS-RT](https://developers.google.com/transit/gtfs-realtime/) standard and are related to the static [GTFS](https://developers.google.com/transit/gtfs/) schedule data above. An example application that uses one of these streams is pictured below: [ovradar](http://ovradar.nl).

![OV Radar](/images/ov-radar.png "Ov Radar")

There are three different types of streams offered: *Trip Updates*, *Alerts* & *Vehicle Positions*:

* The Trip updates stream contains information concerning delayed, cancelled, or new trips in a simple to parse format.
* The Alerts stream contains human readable (Dutch) alerts related to trips, providing context to delays or adding other extra information.
* The Vehicle Positions stream contains updates on the last known position of the different vehicles currently moving through the Netherlands.

These streams are again offered seperately (in line with the GTFS feeds):

0. GTFS-RT streams related to the data from the biggest Dutch rail road provider "NS"
    * Trip Updates - [download debug info](#coming-soon) **coming soon**
0. GTFS-RT streams related to all other available public transport providers
    * Trip Updates - [download debug info](#coming-soon) **coming soon**
    * Alerts - [download debug info](#coming-soon) **coming soon**
    * Vehicle Positions - [download debug info](#coming-soon) **coming soon**

#### Example GTFS-RT client

The links above will only present debug information showing the last set of incoming messages. Consuming the streams in real-time takes slightly more effort as they are communicated over [WebSockets](http://) in the [Protocol Buffer](https://) format.

In order to get you started with consuming these feeds you can download a simple example client ([GitHub](https://github.com/plannerstack/gtfsrt-example-client)) written in Python:

```
# Clone example client
git clone https://github.com/plannerstack/gtfsrt-example-client.git
```

As this is a Python client with several dependencies, you need to install its requirements with pip:

```
# Move into the cloned folder and install requirements
cd gtfsrt-example-client
pip install -r requirements.txt
```

Now your ready to start consuming one of the streams by defining the desired stream to connect to and running the example client that will log the incoming messages:

```
# Run the example client
stream="ws://demo-server-coming-soon.com:port/tripUpdates" python gtfsrt-example-client.py
```


*****
** These streams are free to use for experimentation purposes. If you would like more than a fair-use policy on its usage and an actual SLA consider signing up for [the GTFS & GTFS-RT subscription plan of plannerstack.com](https://app.moonclerk.com/pay/1vmtqc1df9e).**
*****

### Getting the non-unified raw Dutch public transport data

**COMING SOON: Another step deeper into the rabbit hole.**