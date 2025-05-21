# SmartMet International Documentation

Documentation for SmartMet systems that are deployed internationally.

## SmartMet Data Processing Server

* Rocky Linux 9 minimal
* Install Guide https://github.com/fmidev/smartmet-install

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

* RKE2 3 node cluster
* Install Guide https://docs.rke2.io/install/ha
* Docker disk space debugging https://github.com/fmidev/smartmet-international-documentation/blob/main/docker_disk_space_debug.md
## Instal Guides for SmartMet Workstation software

* Smartmet Workstation [Install Guide](/Install/SmartMet%20Workstation.md)
* Smartmet Alert [Install Guide](/Install/SmartMet%20Alert.md)

## SmartAlert CAP (Common Alerting Protocol)

* Visualization demo code [GitHub.com smartalert-web](https://github.com/fmidev/smartalert-web)
* Quide how to edit Smartmet alert Editor text fields [Guide](/Customer%20editable%20fields.pdf)

