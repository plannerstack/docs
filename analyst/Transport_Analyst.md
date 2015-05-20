# Summary of current Transport Analyst capabilities

## Accessibility from or to a given point

This is the basic unit of accessibility analysis. For a given time window, travel modes, and departure point, we calculate the travel time to every street in the region at every minute of the window. This produces a series of travel times and a statistical distribution for every destination, as seen from one particular place. These results are combined with geographic data layers (employment, demographic data) to create histograms and cumulative curves expressing access to different segments of the job market, or access by different segments of the population to a particular facility. These curves can be explored interactively to find results such as the number of skilled workers within 45 minutes of a particular production facility.

![Accessibility][Accessibility]

## Multi-origin accessibility

The process described above is repeated for a large number of points that are representative of the region as a whole, calculating a location-based cumulative opportunities accessibility indicator for each of these points. We typically work with individual blocks or a regular grid, yielding output data with a spatial resolution of a few hundred meters.The resulting continuous accessibility surface reveals how the enabling and unifying effects of transportation networks are distributed spatially, and by extension how they are distributed among the various segments of the population. All calculations are carried out in such a way that the time thresholds and weighting factors can be adjusted interactively after the main calculation is complete.

![Multi-origin][Multi-origin]

This type of analysis requires many millions of separate calculations, which are handled by our highly optimized, parallelized system to provide results in minutes rather than hours or days (as was the case until recently). It is of course possible to upload your own GIS data to Transport Analyst and combine them with accessibility results:

![combine][combine]

## Aggregated many-to-many accessibility

The high-resolution results of the multi-origin accessibility can be normalized to account for variations in population density, then aggregated according to neighborhood or administrative boundaries to increase legibility and applicability to policy goals.

![many-to-many][many-to-many]

## Alternatives analysis

This is the true goal of Transport Analyst and our most sophisticated analysis, which draws on all the techniques described above. All Transport Analyst queries can be run for two or more alternative transport or demographic scenarios, and the results compared to reveal the impact of a network or service modification. In this image of the Transport Analyst interface, we can see the increase or decrease in accessibility from each block due to a new bus line in Mexico City:

![Alternatives][Alternatives]

## Visualizations for the General Public

To encourage debate about transportation planning and ensure communication with the general public, it is possible to publish sets of results in a simplified or “locked” interface on the web. For example, The New York Regional Plan Association’s site shows access to jobs by economic sector and level of education, with interactive isochrones and job counts in 10 categories: http://fragile-success.rpa.org/maps/jobs.html

![Visualizations][Visualizations]

This site developed with Waag Society showing access to schools in Amsterdam with interactive isochrones:
http://code.waag.org/scholen/

![School][School]

## Detailed large-scale case studies

Many custom case studies have been conducted using OpenTripPlanner Analyst. For example, the following image shows the reduction in accessibility for every location in NYC due to tunnel flooding during hurricane Sandy: http://www.citylab.com/commute/2013/01/best-maps-weve-seen-sandys-transit-outage-new-york/ 4488/

![large-scale][large-scale]

The Minnesota Accessibility Observatory has created maps of job access for the entire United States: http:// access.umn.edu/research/america/transit2014/maps/index.html

![Job access][Job access]

## Understanding a regional network

The Marseille Metropole mission of the Provence—Alpes—Côte d’Azur region in France uses OpenTripPlanner to visualize the metropolitan area’s transport options as a cohesive whole, to be understood in terms of access to facilities and employment: http://62.210.125.178/marseille/

![Regional network][Regional network]

## Visualization of individual transport supply

[Conveyal](http://conveyal.com/) also has tools to automatically generate subsets of a schematic network map that are relevant for a particular origin/destination pair, accounting for combined frequencies on common trunks and redundancy: http://[Conveyal](http://conveyal.com/).com/blog/2015/02/24/what-is-profile-routing/

![Individual transport][Individual transport]

Note: this functionality is currently part of another [Conveyal](http://conveyal.com/) product (Modeify), but it may be integrated with Transport Analyst eventually.

[Accessibility]: /images/tpa/accessibility.png "Accessibility"
[Multi-origin]: /images/tpa/multi-origin.png "Multi-origin"
[Combine]: /images/tpa/combine.png "Combine"
[many-to-many]: /images/tpa/many-to-many.png "many-to-many"
[Alternatives]: /images/tpa/alternatives.png "Alternatives"
[Visualizations]: /images/tpa/visualizations.png "Visualizations"
[School]: /images/tpa/school.png "School"
[large-scale]: /images/tpa/large-scale.png "large-scale"
[Job access]: /images/tpa/job-access.png "Job access"
[Regional network]: /images/tpa/regional-network.png "Regional network"
[Individual transport]: /images/tpa/individual-transport.png "Individual transport"
