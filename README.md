# SmartMet International Deployment Documentation

Documentation for SmartMet systems that are deployed internationally.

## SmartMet API / Dataprocessing Server

Software that provides various APIs to query and download weather data and images etc.  

* Rocky Linux 9 (minimal installation variant)
* Installation guide https://github.com/fmidev/smartmet-install

### Default configuration locations

...TBA...

### Data

* Add GFS https://github.com/fmidev/smartmet-data-gfs
  * dnf install smartmet-data-gfs
* Add GEM https://github.com/fmidev/smartmet-data-gem
  * dnf install smartmet-data-gem
* Add SYNOP  https://github.com/fmidev/smartmet-data-gts-synop
* Add SOUNDING  https://github.com/fmidev/smartmet-data-gts-sounding
* Add METAR  https://github.com/fmidev/smartmet-data-gts-metar
* If local GTS files are not available use packages 

### Cron

* Use /smartmet/cnf/cron/cron.d to run scheduled scripts. Do not use smartmet users crontab.

### Triggers

* Use /smartmet/cnf/triggers.d to run scripts when some file or directory updates
* Trigger file name is the path to triggering directory, in file name replace / with :

### CLI tools

* https://github.com/fmidev/smartmet-qdtools

### NOTES

* Do not modify system provided files, they may be overwritten with package updates

## Kubernetes Cluster

* RKE2 three node cluster
* Installation guide https://docs.rke2.io/install/ha
* Docker disk space debugging https://github.com/fmidev/smartmet-international-documentation/blob/main/docker_disk_space_debug.md

## Install Guides for SmartMet Workstation software

* Smartmet Workstation [Install Guide](/Install/SmartMet%20Workstation.md)
* Smartmet Alert [Install Guide](/Install/SmartMet%20Alert.md)
* SmartMet Analysis [Install Guide](Install/SmartMet%20Analysis.md)

## SmartAlert CAP (Common Alerting Protocol)

* Visualization demo code [GitHub.com smartalert-web](https://github.com/fmidev/smartalert-web)
* Quide how customer can edit Smartmet alert Editor text fields [Guide](/Customer%20editable%20fields.pdf). Beta version, comments required before using.


## Source Codes
* SmartMet Workstation https://github.com/fmidev/smartmet-workstation
* SmartMet Server https://github.com/fmidev/smartmet-server

