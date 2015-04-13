# Local check of OSM (Openstreetmap) fixes

We update our graph and servers on a nightly base, which includes updating our OSM data.  However, troubleshooting with a delay of 24 hours is not the most efficient method. Therefore we suggest the following: 

#Preparation

##Install 

* JOSM: https://josm.openstreetmap.de/ in order to create (local) OSM fixes)
* Wget http://www.gnu.org/software/wget/
* JDK http://www.oracle.com/technetwork/java/javase/downloads/index.html

## Plannerstack customers only:

* OTP build: User our preconfigured OTP build http://otp.plannerstack.org/otp.jar (instead of step 4 below)
* Realtime support: After step 7 you can add our graph.properties file for real time support using our GTFS-RT feed. http://otp.plannerstack.org/Graph.properties
Actions

# Open terminal

```
mkdir Routing
cd Routing
```

# Create a build of OTP 
Plannerstack customers can use the served build as explained above.

```
echo "CLONE OTP"`
git clone https://github.com/opentripplanner/OpenTripPlanner.git
echo "BUILD OTP"
cd OpenTripPlanner
mvn clean verify -DskipTests # Or don't skip tests if you don't want to
# otp-0.15.0-SNAPSHOT.jar is nu in /target (or with a different version if newer)
mkdir build
```

# Test
Open josm and download the adjusted part OSM stuk dat je wilt testen
Save As (in the build folder)
```
java -jar target/otp-0.15.0-SNAPSHOT.jar --build build --inMemory
http://127.0.0.1:8080 #Select Walk Only
```
Check if OSM fix works (if so, make sure to publish your fix to OpenStreetMap!)
