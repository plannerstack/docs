# Bliksem Integration: schedules

# Introduction

Bliksem Integration is an open source Extract­Transformation­and­Load (ETL) framework for public transport schedules and facilitating continuous integration. The intermediate database is closely based on NeTEx/Transmodel and supports versioning. This document describes the data flow of Bliksem Integration and can be used as reference in academic works.

## Source data

Travel information data is obtained from various sources. Most prominently ndovloket.nl is used as the leading datasource for all public transport schedule information in The Netherlands. Dutch timetables for bus, tram, lightrail, ferries and metro are published in BISON KV1 (“Interface 1”), a Transmodel 8 scheme, serialised in a character separated file format. Timetables by Dutch Railways (NS) come in International File Format (IFF), a proprietary text based format by Hewlett­Packard (HP) for use in their journey planner. Other international sources such as HAFAS, Belgium Local Transport Automatic Connexion (BLTAC) and DINO are supported out of the box.

## Output data

The intermediate datastore allows serialisation to various common file formats such as General Transit Feed Specification (GTFS), but also output to various journey planners such as HP, Navitia and RRRR has been added. Alternatively the intermediate database can be used as computation platform to obtain derivatives such as network­frequencies and clustering of quays. The intermediate database is ideal to generate ESRI shapefiles or direct database integration with a Geographic Integration System such as QGIS.

## Usage

Bliksem Integration is currently in use by OVapi for the publication of integrated GTFS feeds. Users of this data include: Nokia Here, Bing Maps, GoAbout, OpenTripPlanner, and GeOps. Within HP the toolchain is used to facilitate easy conversion between various sources directly to IFF. A light serialisation is available to match historical trip and real­time data. Derivative geo­products are published by openOV which include quay locations and frequencies for all lines, which are used in environmental studies.

# Extract-Transformation-and-Load

The ETL­platform divided into an automatic downloader and extract routine for timetables which is referred to as import.py. Various transformation routines for different formats and agencies are made available in the folder importers. Finally the export to various file formats is made available in the folder exporters.

## Extract

The automatic download script import.pylaunches various importers. An importer may contain a sync()function to update the data source to the most recent schedules available. In case more recent timetables have been published, the sync function will download each of them to the /tmpfolder on the filesystem. At the moment of writing this path is not configurable. When the download is successful, the information is parsed and transformed.

## Transformation

The intermediate serialisation prior to the database import is JSON. We will discuss the different sections which map to the transmodel based design. In general three different identifications can be observed in a record. The id, a unique auto­incremented numeric value to identify the record, an operator_id, which is an unique value prefixed with the operatorname, and a privatecodecommonly a primary key is used within original datasource.

### Datasource

The datasource of the importer defines where the data is obtained from. The datasource contains a mandatory attributes operator_idand name, describing an unique identification of the datasource provider, which may not be the transit agency. Furthermore a description, emailand URLmay be provided to identify contact information of this datasource.

### Version

Each import instance is considered a new version. Depending on the defined mergestrategya previous versionmay be removed, appended, or overwritten. A version must reference a datasource in datasourceref, and supply an privatecodesuch as the filename of the source, a startdateand an enddate. Additional attributes may include a description, versionmajor, versionminor, and operator_id, an identification given to this dataset by the provider.

### Operator

The agency that is responsible for the trip is refered as operator.

### DestinationDisplay

The destination display information contains the heading of the vehicle which, in the general case, is equal to the name terminating stop or station. Mandory attributes include privatecode, name, shortnameand may include a vianamedestination and an operator_id.

### Line

A line records contains information of multiple routes having the same name and identification for the general public. The fields a line record must specify are privatecode, operator_id, publiccodeas the line number, the transportmodeused for the line one of BOAT, BUS, METRO, TRAIN, TRAM. Optional fields include a reference to the operator table as operatorref, if the line is real­time monitored, a color_shieldand color_text used

for the rendering on maps and journey planners and URLrefering to more information about the line.

### StopPoint

Stop points are modelled as logical elements in the datasource. Each stop point describes a stop found in the timetable, while stops may be similar and equal in name and/or location they are not merged as such between different operators, which will eventually happen in the quays table. The following attributes of a stop can be described privatecode, operator_id, publiccodethe number the public can read from a stopshield, if the stop isscheduled, being part of a stoparea as stoparearef, the name, the town, latitudeand longitudein WGS84, rd_xand rd_yin EPGS:28892, the timezone, platformcode, if the stop is accessible for the visual and mobility impared as visualimpairmentsuitableand restrictedmobilitysuitable.

### StopArea

A collection of related stoppoints are grouped as StopArea. Which may have the following attributes: privatecode, operator_id, name, town, latitudeand longitudein WGS84, a timezoneand a publiccode.

### AvailabilityCondition

A collection of consecutive days of operation is recorded as availability condition. Because the availability condition references the earlier defined version record, removing this record in cascade will explicitly remove the availability condition and all references to it. The table journeys references an availability condition. The record contains a privatecode, operator_id, unitcodewhich might denote a hierarchy in timetables or geographical partition thereof, a reference to the version table as versionref, a name, and a fromdateand an enddate.

### ProductCategory

Product categories allow the agency to commercially differentiate between different types of service such as a Thalys, R­NET or just an ordinary “bus”. The category is recorded as privatecode, operator_id, shortnameand name.

### AdministrativeZone

A set of stops and lines are typically financed by an governmental agency. The administrative zone is used as a inbound reference from PointInJourneyPatternto this grouping. The properties which can be assigned to a zone are: privatecode, operator_id, nameand a description.

### TimeDemandGroup

A group of journeys are considered part of the same Time Demand Group when their arrival and departure times in a time dependent notation are equal. Hence each trip does have a different initial departure time, but are parallel with respect of their relative times. The table is merely a grouping referenced by the Journeyand PointInTimeDemandGrouptables, thus only contains two attributes: operator_idand privatecode.

### Route

A route is a grouping of consecutive geo­points referenced by the JourneyPatternand PointInRoutetables. A route can reference to a line by lineref, the route defines the path on the road on the rail track. The main difference between a Routeand a JourneyPatternis the geographical component. A single instance of a route refers to how a vehicle travels from stop to stop geographically speaking, each point contains coordinates, which may be in a finer granularity than consecutive stops. Route information allows to create pretty maps and exact distance calculations, JourneyPatternswould create straight lines between stops.

### JourneyPattern

A journey pattern is a grouping of consecutive stops and refers to a route via routerefas the path to these stops. Additional properties such as privatecode, operator_id, its directiontype, inbound or outbound, and its general headsign as destinationdisplayrefare supported. References to Journey Patterns are from the journeytable as pointinjourneypattern.

### Journey

The journey contains all properties required to describe a single trip. Next to the privatecode and operator_idan availabilityconditionrefallows the journey to be scheduled on an exact day. Each journey must also include a JourneyPatternref, a TimeDemandRef, a ProductCategoryRef. Additional attributes are NoticeAssignmentRef, the first DepartureTimeof the trip, a blockrefto announce a sequence of trips are ran by the same vehicle thus transfer times may be zero, a nameof the journey, and boolean attributes: lowfloorto allow easy access, haslifttorampfor access by wheelchairs, haswififor wireless internet access, bicycleallowedif bycycles can be taken on board, ondemandif the trip should be requested by phone or another medium, isvirtualto announce that the trip is operated by a different agency.

### JourneyTransfers

Two journeyrecords be referenced and coupled by a Journey Transfers. The agency is able to mark explicit points where passengers have guaranteed transfers. The properties of this record must include: an operator_id, a journeyrefand pointrefto transfer from, onwardjourneyrefand onwardpointrefto transfer to, and an optional transfer_type which is used in exporting to GTFS.

### NoticeAssignment

To allow messages and attributes to be assigned to specific journeys a notice assignment can be scheduled. The properties of an assignment include: a noticegroupref, privatecode, operator_id, name, validfrom, validthru.

### NoticeGroup

A notice assignment references a noticegroupwhich in its case is the reference for a group of individual messages and attributes. Its properties are merely: operator_idand noticeref.

### Notice

An individual textual notice is spelled out in the nameproperty. Futher attributes include: privatecode, operator_id. publiccode, shortcodeare used to store attributes specific to a journey.

## Derivative VIEWs

### ServiceJourney

The table ServiceJourney consists of all non­virtual trips. Hence all trips that are operated by the agency that published them. Or in SQL terms:
```SQL
SELECT * FROM journey WHERE isvirtual = FALSE;
```

## Load

### Shape-Counter

The shape­counter table is integrated to a single table which is easily exported to an ESRI­shape file. For each geographical route available in the database the following attributes are added: the internal line number, mode of transport, agency name and public line number. Based on the availability conditions present a derivative summation per day is calculated for the specified period.

```SQL
drop table if exists shape_counter;
create table shape_counter as (
SELECT
routeref,l.operator_id as line_operator_id,transportmode,o.name as operator,publiccode,l.name,monday,tuesday,wednesday,thursday,friday,saturday,sunday,geom FROM

(SELECT
routeref, sum((extract(isodow from sum((extract(isodow from sum((extract(isodow from sum((extract(isodow from sum((extract(isodow from sum((extract(isodow from sum((extract(isodow from FROM servicejourney as j

validdate)=1)::int4) as monday, validdate)=2)::int4) as tuesday, validdate)=3)::int4) as wednesday, validdate)=4)::int4) as thursday, validdate)=5)::int4) as friday, validdate)=6)::int4) as saturday, validdate)=7)::int4) as sunday

JOIN availabilityconditionday as ad USING (availabilityconditionref)

JOIN journeypattern as jp ON (jp.id = j.journeypatternref)
WHERE ad.isavailable = true AND ad.validdate BETWEEN '2014­03­24' and '2014­03­30'
GROUP BY routeref) as routecount JOIN route as r ON (r.id = routeref) JOIN line as l ON (l.id = lineref) JOIN operator as o ON (o.id = operatorref)
JOIN (SELECT routeref,st_makeline(array_agg(ST_SetSRID(ST_MakePoint(longitude, latitude), 4326) ORDER BY pointorder)) as geom

FROM pointinroute GROUP BY routeref) as shapes USING (routeref)
WHERE monday+tuesday+wednesday+thursday+friday+saturday+sunday > 0
and (o.operator_id not like 'IFF:%' or l.transportmode = 'TRAIN') AND o.operator_id not

like 'EETC:%' );﻿
```

![](https://plannerstack.uservoice.com/assets/75331675/Screen%20Shot%202014-10-29%20at%2009.17.49.png)