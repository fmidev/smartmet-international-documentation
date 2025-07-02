[TOC]

# SmartMet Analysis Installation and Configuration Guide 

Software consists of client and server software. Both are written in Java programming language. This guide explains how to install and configure both client and server software.

## Common requirements for client and backend server

- SmartMet Server with Q3 plugin
- Necessary WMS servers

# Server (backend services)

SmartMet Analysis server can be installed on any Linux server. It is not dependent on any specific Linux distribution

## Backend server requirements

- Latest release of Java 8 JDK Full release. Please make sure you have Full JDK (JRE is not sufficient) and the version is 8. Recommended is Bellsoft Liberica.
- Podman or Docker


## Installation

Create required user for backend

```
groupadd -g <gid> mirri
useradd -u 1002 -g 1002 -c "SmartMet Analysis" -d /home/mirri -m -s /bin/bash mirri
```

Install Bellsoft Java (example for RPM-based system) by downloading package from [their website](https://bell-sw.com/pages/downloads/#jdk-8-lts)

```
dnf install ./bellsoft-jdk8u452+11-linux-amd64-full.rpm
```

Create necessary directory structures
```
mkdir -p /var/run/spring-boot-applications/
mkdir -p /var/log/mirri-server/
chown mirri:mirri /var/run/spring-boot-applications/ /var/log/mirri-server/
```

Install Podman and some helper apps (docker can be used instead of podman if necessary)
```
dnf install tmux vim podman
```

Install MongoDB database as a container
```
podman run --name mongo2612 -v /home/mongodb/data:/data/db --network host -d mongo:2.6.12 --auth
```

Create systemd configuration file `/etc/systemd/system/mongodb-podman.service` for MongoDB with contents
```
[Unit]
Description=MongoDB via Podman container

[Service]
Restart=on-failure
ExecStart=/usr/bin/podman start mongo2612
ExecStop=/usr/bin/podman stop mongo2612
KillMode=none
Type=forking

[Install]
WantedBy=multi-user.target
```

Reload systemd
```
systemctl daemon-reload
```
Start mongo command line tool inside container
```
podman ps
podman exec -it <id> mongo
```
Create users, define passwords and databases in MongoDB
```
MongoDB shell version: 2.6.12
connecting to: test
> use admin
switched to db admin
> db.addUser( { user: "admin", pwd: "ADMIN_PASSWORD", roles: [ "userAdminAnyDatabase" ] } )
WARNING: The 'addUser' shell helper is DEPRECATED. Please use 'createUser' instead
Successfully added user: { "user" : "admin", "roles" : [ "userAdminAnyDatabase" ] }
> use admin
switched to db admin
> db.auth("admin", "ADMIN_PASSWORD" )
1
> use woml_conceptualmodelanalysis
switched to db woml_conceptualmodelanalysis
> db.addUser( { user: "woml_conceptualmodelanalysis_agent",pwd: "ANALYSIS_PASSWORD",roles: [ "readWrite", "dbAdmin" ] })
WARNING: The 'addUser' shell helper is DEPRECATED. Please use 'createUser' instead
Successfully added user: {
	"user" : "woml_conceptualmodelanalysis_agent",
	"roles" : [
		"readWrite",
		"dbAdmin"
	]
}
```
Define local server firewall rules if necessary
```
firewall-cmd --permanent --add-port=1025-1200/tcp
firewall-cmd --reload
```

## Configuration

# Client (forecaster tool)

## Installation

SmartMet Analysis client is usually installed to same computer as SmartMet Workstation. It is possible to install it to other computers as well. Software is Java based but does not require Java installation in operating systems. Java is already bundled with the software. 

1. Acquire the newest software package from FMI. This software has to be custom built for every installation by FMI staff so it is not available online for download. 

2. Unzip the downloaded SmartMet Analysis installation package to `c:/smartmet` folder in your computer. It will install into folder `smartmet-analysis-<country>`

3. Program is started from file `program-files/execute_analysis_editor_production.bat.` If necessary, you can create a shortcut for this on the computer Desktop. There should normally not be need to edit this file unless you want to use a different Java than the one bundled. Also included WMS and other servers are defined here but they should be pre-configured by FMI in normal cases.

## Software configuration

Basic parameters for backend servers are configured in file `program-files/production<country>.properties`

Usually following configuration parameters need to be modified:

SmartMet Server address
```
smartmet.host=<hostname or ip address>
smartmet.http.port=<80 or 443>
smartmet.protocol=<http or https>
```

Analysis backend database parameters. Change at least hostname, default port should be ok unless changed for some reason (firewall for example).
```
#DB
fi.fmi.mirri.server.rmi.default.hostname=<hostname or ip address>
fi.fmi.mirri.server.rmi.default.port=1089
```

Define which NWP models are allowed to use as base for products. Names should match SmartMet Server `q3plugin.conf` track name.
```
#Analysis Tools
fi.fmi.mirri.plugin.interaction.woml.tools.models=ECMWF,GFS
```

### Other parameters

Define track name for observations as set in SmartMet Server `q3plugin.conf`
```
# Observation data track
jnlp.fi.fmi.mirri.plugin.service.observationdata.data.tracklist=OBS
```

Allows filtering of which NWP model appear in layer list. Set to empty to include all models, otherwise only ones mentioned here appear in the list.
```
jnlp.smartmet.modeldata.filteredTracks=
````

Allows filtering which sounding models appear in sounding tool. Set to empty to include all models, otherwise only ones mentioned here appear in the list.
```
#Sounding models - if empty then all models are used
jnlp.spring.sounding.tracks=MEPS,EC
```

## Product configuration

You can predefine end user products in the software. The product is generated from the following components
- Backgrounds from WMS layers (basemap, SYNOP, ...)
- Backgrounds from NWP model data (different parameters visualized in various ways..)
- Edited components like fronts, symbols, rain areas, text boxes etc.
- Configurable timestamp/header

Once a product is predefined it is available from the product menu (disk icon). See more details in the user manual on this.

### Product configuration to add new product to product list

Available products added to the list are defined in `mirwaShare/printSpec/printSpecAnalysis<country>.xml` in the following section

```
  <alias name="default-print-specs-analysis" alias="print-specs-analysis" />
  <bean id="default-print-specs-analysis" class="java.util.HashMap">
    <constructor-arg>
      <map>
        <entry key="Ukraine"      value-ref="ukraine-print-spec-1" />
        <entry key="Ukraine west" value-ref="ukraine-print-spec-2" />
        <entry key="Ukraine east" value-ref="ukraine-print-spec-3" />
        <entry key="Europe" value-ref="ukraine-print-spec-4" />
      </map>
    </constructor-arg>
  </bean>
  ```
  This example has 4 products: three different types for Ukraine area and one for whole Europe.

  Example configuration for first product `ukraine-print-spec-1`:
  ```
    <bean id="ukraine-print-spec-1" class="fi.fmi.mirri.workstation.gui.print.PrintSpec">
    <constructor-arg value="
      EPSG:3857:17.898582,43.578516,45.073913,52.889876@
      700,0@
      true@
      1.0@
      30@
      dd.MM.yyyy HH:mm' - Forecast Ext'@
      5@
      240@
      true"
    />
  </bean>
  ```
  Explanation for the different parameters can be found below
  ```
  EPSG:999920:20.0341968,56.7353176,32.665266,60.0873119@  <!--Image projection-->
  700,0@                                                   <!--Image size - if second value is 0 editor calculates it-->
  true@                                                    <!--Show information (time stamp and text)-->
  1.0@                                                     <!--Components scale value 0.5 .. 2.0 -->
  30@                                                      <!--Information font size-->
  dd.MM.yyyy HH:mm' - Forecast'@                           <!--Information format, additional text inside single quotes -->
  5@                                                       <!--Information placecode 0,1,2 (bottom right/center/left)  - 3,4,5 (top right/center/left) -->
  240@                                                     <!--Information fadeout or Expand information background color-->
  true                                                     <!--Expand information to Image outside-->
  ```
  ### Additional notes on Image projection

  Definition of the area and projection info is done in `mirwaShare/projection/projectionAnalysisUkraine.xml` configuration file.

  Areas defined in `area-map-analysis`bean and frameworks in `area-border-analysis`. Based on the these definitions it is possible to create 
```
...
  <bean id="ukraine-map-1" class="fi.fmi.common.geospatial.projection.AreaFactory"
    factory-method="Create">
    <constructor-arg
      value="EPSG:3857:39.4345,40.1392,47.5426,44.113" />        <!-- 39.4345,40.1392,47.5426,44.113 -->  lon(x),lat(y),lon(x),lat(y) -->
      <!-- OR -->
      value="EPSG:3857:43.5,42.2,440000#290000" />              <!-- 43.5,42.2,440000#290000 -->  lon(x),lat(y) (center point) ,width(m),height(m) -->
  </bean>

  <entry key="Ukraine 1" value-ref="ukraine-map-1" />
...
```
These area values can also be visually obtained/defined in the software itself
- start the editor without logging in
- select some ready-made projection and zoom to the area you want as the base
- select the ratio of the edges of the area by changing the ratio/size of the edges

To do the above please see some pointers below (also consult user guide for basic operations)

Zoom/panning basics

- Globe in the lower right corner of the editor - click / double click (remains valid and click closes) opens a window
- Use the mouse wheel to zoom in and out (doesn't go out of bounds)
- Slider zooms in and out (does not go beyond limits)
- Move the frame to the desired location
- Double click returns the initial state
- CTRL-double click takes the view further away from the center view

Read and recover basic information about the area and periphery

At the bottom left is an area that contains information about the area. Different content can be viewed by clicking the right mouse button (recycles information)

- point information
```
[x= .., y= ..]
[lat=..,lon=..]
```
the combination of the previous

regional information
```
EPSG:3857:35.532603,38.759447,51.444496,45.382017 (CTRL-double click copy of the information - the information can be used as such in the area definition)
[width=1065,height=598]
3955471.307320699,4687272.8546602735,5726775.175354935,5681864.134551796 (CTRL-double click copy of information - no use)
```

A second approach is to use the Product Dialogue in the software

- start the editor without logging in
- select a map template view as the only layer
- select productization selection button (disc button)
- select the dialog "Save  (ctrl)-View" by clicking on the shift-ctrl combination

Now you get a new dialogue `Check print details - currentScreen`
- set/replace from there "Dimension value to 800" (set standard width and height to be calculated automatically by projection)
- set/replace the next line in the "Area" field to `EPSG:3857:25.0.5000000#3600000` (Europe)
- select the "View" button so that the map template appears in a PNG-capable application which is usually the default web browser in the system

After this the specification of a more detailed map area can be started as follows:

- Make changes to area parameters (25.0.50.0.40000000#3600000) - lon(x),lat(y),width(m),height(m)
for example: EPSG:3857:43.5.42.2,440000#290000 (Ukraine)

Fine-tune the area to its final shape by adjusting the center and width/height in meters

Extract the "Area" values and transfer them to the structure required by the file `projectionAnalysis<Country>.xml` (or `printSpecAnalysis<Country>.xml`)

  ### Additional notes on Image size
  
  Product size can be defined by entering either page size (pixels) and the other zero, in which case it is automatically calculated. Size can also be given to either page allowing it to distort the projection if the values are not correct (sometimes, however, it is necessary)

  ### Additional notes on Information format

  Use the same format as [SimpleDateFormat](https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html).