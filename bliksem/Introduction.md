# Bliksem Integration: The introduction

Bliksem Integration is a new open source transit data integration platform for timetables and realtime data. The toolkit consists of an ETL layer capable of extracting data from national standards into a normalised, versioned database. This database can be transformed in virtually any transmodel based concept such as GTFS or a NeTEx profile of choice. Using the stored schedules and punctuality information we are able to produce GTFS-Realtime feed. In this article Bliksem Labs provides you the insights to setup the system for the Dutch market. Future articles will cover setting up a GTFS-RT producer and a quick tutorial how timetables can be quickly build  by hand for situation were open data isn't available, and crowd sourcing is.

##The foundation

Our core infrastructure is called RID, (or in Dutch: Reisinformatie Integratie Database). This database is hosted in PostgreSQL 9.3 and is using PostGIS features to describe geographical shapes. In essence our database is time-dependent in journey structure, using an expanded validity per operating day. The ETL implementation is written in Python 2. Using our intermediate serialisation in JSON it is possible to use different programming language to interact with the database. The the GTFS-Realtime producer is based on OneBusAway's GTFS Realtime exporter and is written in Java. The current code is capable of converting Dutch BISON KV1, KV6, KV15 and KV17 and Dutch Railways AR-NU into GTFS-Realtime. 

![](http://plannerstack.files.wordpress.com/2014/03/rid.png)

## The installation

The basic requirements for using the integration database are:

```
Linux 3.13
PostgreSQL 9.3
PostGIS 2.0.2
Python 2.7.6
git
pip 1.5
```

Using pip we will install the following requirements:

```
lxml 3.2.5
psycopg 2.5.2
beautifulsoup 4.3
```

## Helpful commands

Check out the repositories by:

```
git clone https://github.com/bliksemlabs/bliksemintegration
git clone https://github.com/bliksemlabs/bliksemintegration-realtime
```

Start the process to install de dependancies for bliksemintegration:

```
cd bliksemintegration
pip install -r requirements.txt
```

Create a database and initialise it:

```
createuser -h 127.0.0.1 -U postgres -P -R -S rid
<< enter your password and remember it >>
createdb -h 127.0.0.1 -U postgres -O rid -E UTF8 ridprod
createdb -h 127.0.0.1 -U postgres -O rid -E UTF8 kv1tmp
createdb -h 127.0.0.1 -U postgres -O rid -E UTF8 ifftmp
cat sql_schema/rid.sql | psql -h 127.0.0.1 -U rid -d ridprod
cat sql_schema/kv1tmp.sql | psql -h 127.0.0.1 -U rid -d kv1tmp
```

To update the connection details, you should add them to settings/const.py

```
database_connect = "dbname='ridprod' host='127.0.0.1' user='rid' password='secret'"
kv1_database_connect = "dbname='kv1tmp' host='127.0.0.1' user='rid' password='secret'"
iff_database_connect = "dbname='ifftmp' host='127.0.0.1' user='rid' password='secret'"
```

We suggest that you check in your settings/const.py file, and keep your repository private. As you may notice you are suggested to fill in the ndovloket_user and ndovloket_password, which require a subscription. You can request access by https://ndovloket.nl/aanmelden/

# Running the Code

After you have successfully finished with the installation, the supporting scripts can do a lot of work for you. A lot of magic happens in the import.py process, we have managed to do a lot of work out of the box, and will be supporting this effort. But due to the quality of data, restrictions in downloading some scripts must be ran with user intervention. Each operation may have a different ETL strategy, more general: we download all the available files from our primary transit data provider NDOVloket, import this file using a agency specific transformation in an intermediate database, then load it into RID.

## Updating the source code

After the basic installation you might want to update the source code in the future. It is very easy, just browse to the repository and type:

```
git pull
```

## The automatic import

To start the automatic import process, which will take quite some time at the first run:

```
python import.py
```

The automatic import process tries to figure out the differences in the publications and only applies new data, therefore you can add this file to a cronjob. We suggest you run this code somewhere between 2am and 4am.

## The manual intervention
Some import steps are more complex, for example Connexxion. After manually downloading the latest open data (CC-0 licensed) timetable from http://kv1.openov.nl/connexxion/ You will have to manually import it, take care for the date in the filename. The date should be entered as well, you may wonder why. Connexxion publishes a timetable which is incomplete, prior to the date mentioned in the file name. Hence, if we would naively import it, and overwrite our current data, the result would be incorrect. The fundamental reason can be found in scheduling software projects, which are out of our reach.
python cxx-import.py "Export KV1 CXX versie 2 ingaande 23-02-14.zip" 2014-02-23

## Additional import scripts

AVV data:
```
python avv-import.py "filename"
```
NS data:
```
python ns-import.py "filename"
```
Qbuzz data:
```
python qbuzz-import.py "filename"
```
RET KV1 data:
```
python ret-import.py "filename"
python retbus-import.py "filename"
```
