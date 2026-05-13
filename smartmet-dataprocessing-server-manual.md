# SmartMet Dataprocessing Server Manual

Training manual for SmartMet international installations

Draft date: 2026-05-04 

Note. this is mostly AI generated and needs reviewing.

## 1. Purpose and scope

This manual describes how to install, use, configure, and extend the SmartMet dataprocessing server component used in SmartMet international installations.

The dataprocessing server is the part of the SmartMet environment that receives, downloads, converts, validates, stores, and distributes weather data. Its primary output format is QueryData (`.sqd`), which is used by SmartMet Workstation and SmartMet Server API components such as Timeseries and WMS.

This guide focuses on:

- Installing the base dataprocessing environment with `smartmet-install`.
- Understanding how `/smartmet` is organized.
- Understanding the data flow from incoming files to QueryData outputs.
- Installing packaged datasets such as GFS, GEM, METAR, SYNOP, and sounding data.
- Adding new local or external datasets.
- Scheduling and triggering production routines.
- Validating data with SmartMet command-line tools.
- Maintaining and troubleshooting the system.

This manual complements the training deck `smartmet-server-training-general-dataprocessing_v1.pptx` and the public FMI GitHub documentation listed in section 15.

## 2. System role in the SmartMet environment

SmartMet international installations normally have several logical components:

- **Dataprocessing server**: Downloads, receives, converts, validates, and stores data.
- **SmartMet Workstation clients**: Use QueryData through an SMB share and can return edited data.
- **SmartMet product/API server**: Serves data through APIs such as Timeseries, WMS, WFS, download, autocomplete, and application-specific endpoints.
- **External data sources**: Numerical weather prediction models, GTS observation feeds, local observation systems, radars, satellites, CSV/ASCII services, NetCDF services, or manually delivered files.

The dataprocessing server is not just a file store. It is the production engine that turns source data into the consistent data layout expected by Workstation and API services.

Typical flow:

```text
External source or local feed
  -> /smartmet/data/incoming/<source>
  -> routine in /smartmet/run/data/<dataset>/bin
  -> conversion with qdtools or ingest-model
  -> validation with qdstat, qdinfo, or related tools
  -> /smartmet/data/<producer>/<area>/<level>/querydata
  -> /smartmet/editor/in for SmartMet Workstation delivery
  -> SmartMet Server API reads configured QueryData producers
```

## 3. Operating system and base installation

The supported base platform for the international dataprocessing environment is a Red Hat compatible Linux distribution. Current public installation guidance points to:

- Rocky Linux 9 minimal installation, or
- AlmaLinux 9 minimal installation.

Before running the SmartMet installer:

1. Install Rocky Linux 9 or AlmaLinux 9 minimal.
2. Enable networking.
3. Enable NTP or another reliable time synchronization service.
4. Create and mount the largest data partition as `/smartmet`.
5. Log in as `root`.

The `/smartmet` mount is important. It holds operational data, runtime scripts, logs, temporary conversion files, Workstation exchange files, generated products, and local configuration. Size it for the largest expected model datasets plus retention.

### 3.1 Run the installer

On a Rocky Linux 9 or AlmaLinux 9 server, run:

```bash
cd /root
curl -O https://raw.githubusercontent.com/fmidev/smartmet-install/master/smartmet-install-9.sh
chmod a+x smartmet-install-9.sh
./smartmet-install-9.sh
```

The Rocky/Alma 9 installer currently performs the base repository and package setup for the international environment:

- Installs the SmartMet Open RPM repository.
- Installs `yum-utils`.
- Adds the Docker CE repository.
- Enables the CRB repository.
- Installs EPEL.
- Excludes EPEL `eccodes*` packages so SmartMet-compatible ecCodes packages are used.
- Disables the PostgreSQL 15 module.
- Installs `smartmet-base-international`.
- Updates installed packages.

After installation:

```bash
passwd smartmet
smbpasswd -a smartmet
```

The `smartmet` Linux account is used by the SmartMet file tree and by SMB access for SmartMet Workstation data exchange.

### 3.2 Time zone

Set the server operating system timezone correctly and check PHP timezone configuration where relevant:

```bash
timedatectl
timedatectl list-timezones
timedatectl set-timezone <Region/City>
vi /etc/php.d/smartmet.ini
```

Keep production timestamps in data processing scripts UTC unless there is a clear system-specific reason to use local time. Model reference times, QueryData origin times, and file names are normally easier to operate when UTC is used consistently.

## 4. `/smartmet` directory layout

The standard dataprocessing environment is organized under `/smartmet`.

```text
/smartmet
+-- bin
+-- cnf
+-- data
+-- editor
+-- logs
+-- products
+-- run
+-- share
+-- tmp
`-- www
```

Main directories:

| Path | Purpose |
| --- | --- |
| `/smartmet/bin` | General system-wide helper programs and scripts. |
| `/smartmet/cnf` | System and local configuration. Includes cron snippets, trigger definitions, and data configuration files. |
| `/smartmet/data` | Data repository. Each producer or data family should have its own subtree. API services read data from here. |
| `/smartmet/data/incoming` | Landing area for externally delivered files before conversion. |
| `/smartmet/editor` | SMB shared directory used for SmartMet Workstation input and output. |
| `/smartmet/editor/in` | QueryData delivered to SmartMet Workstation clients. |
| `/smartmet/editor/out` | Typical location for edited data returned from Workstation, if enabled in the installation. |
| `/smartmet/logs` | Logs from production routines. Use a consistent dataset-specific log name under `/smartmet/logs/data`. |
| `/smartmet/products` | Generated products such as image outputs. |
| `/smartmet/run` | Runtime scripts and routine-specific configuration. Each dataset should have its own directory. |
| `/smartmet/share` | Shared libraries, tables, shapes, parameter files, or other common resources. |
| `/smartmet/tmp` | Temporary files used during downloads and conversions. |
| `/smartmet/www` | Web files served locally, if used. |

Recommended dataset layout:

```text
/smartmet/run/data/<dataset>
+-- bin
+-- cnf
`-- etc

/smartmet/cnf/data/<dataset>.cnf
/smartmet/cnf/cron/cron.d/<dataset>.cron
/smartmet/cnf/cron/cron.hourly/clean_data_<dataset>
/smartmet/data/incoming/<dataset>
/smartmet/data/<dataset>/<area>/<level>/querydata
/smartmet/logs/data/<dataset>.log
/smartmet/tmp/data/<dataset>_<timestamp>
```

Use one dataset or producer name consistently across paths, logs, scripts, and output file names.

## 5. QueryData basics

QueryData is the main internal data format for SmartMet Workstation and many SmartMet Server API deployments. It is a binary format that can store gridded data, point data, metadata, times, levels, parameters, and producer information.

Important QueryData metadata:

- **Parameters**: ID, name, description, interpolation, and precision.
- **Producer**: Numeric ID and producer name.
- **Time**: Origin time, first valid time, last valid time, timestep, and number of timesteps.
- **Area**: Projection and grid corner coordinates.
- **Levels**: Surface, pressure levels, hybrid/model levels, height levels, and other level types.

Useful tools from `smartmet-qdtools`:

| Tool | Use |
| --- | --- |
| `qdinfo` | Show QueryData metadata, parameters, times, grid, and producer information. |
| `qdstat` | Validate a QueryData file and calculate basic data statistics. |
| `qdpoint` | Extract a time series for a selected point. |
| `qdarea` | Extract area statistics. |
| `qdcrop` | Crop QueryData spatially. |
| `qdfilter` | Calculate new values from existing parameters. |
| `qdscript` | Run SmartTools macros from command line. |
| `qdset` | Change QueryData metadata, such as parameter names. |
| `qddiff` | Compare QueryData files. |
| `qdmissing` | Check missing data. |
| `gribtoqd` | Convert GRIB1 or GRIB2 to QueryData. |
| `csv2qd` | Convert ASCII/CSV point data to QueryData. |
| `bufrtoqd` | Convert BUFR observations to QueryData. |
| `metar2qd` | Convert METAR messages to QueryData. |
| `synop2qd` | Convert SYNOP messages to QueryData. |
| `nctoqd` | Convert CF-compliant NetCDF to QueryData. |

Basic checks:

```bash
qdinfo -P -T -x -z -r -q /smartmet/data/<producer>/<area>/<level>/querydata/<file>.sqd
qdstat /smartmet/data/<producer>/<area>/<level>/querydata/<file>.sqd
qdpoint -p "24.94,60.17" -P Temperature /smartmet/data/<producer>/<area>/<level>/querydata/<file>.sqd
```

## 6. Production scheduling and triggers

There are two common ways to start processing routines:

- Scheduled processing with cron.
- Event-driven processing with SmartMet trigger definitions.

### 6.1 Cron

Use `/smartmet/cnf/cron/cron.d` for scheduled SmartMet production scripts. Do not use the personal crontab of the `smartmet` user for production routines. The international documentation explicitly recommends `/smartmet/cnf/cron/cron.d`.

Example:

```cron
# /smartmet/cnf/cron/cron.d/test_obs.cron
5 * * * * /smartmet/run/data/test_obs/bin/fetch_convert.sh
```

Cron job design rules:

- Use absolute paths.
- Write logs to `/smartmet/logs/data`.
- Use a lock file if a new run can start before the previous one has finished.
- Keep source files, temporary files, and outputs separate.
- Validate output before moving it into the production data directory.

### 6.2 Cleaner jobs

Every routine should have retention rules. Put cleanup scripts in:

```text
/smartmet/cnf/cron/cron.hourly
```

Example:

```bash
#!/bin/sh
# /smartmet/cnf/cron/cron.hourly/clean_data_test_obs

cleaner -maxfiles 2 '_test_obs.sqd' /smartmet/data/test_obs
cleaner -maxfiles 2 '_test_obs.sqd' /smartmet/editor/in
find /smartmet/data/incoming/test_obs -type f -mmin +10080 -delete
```

Make cleaner scripts executable:

```bash
chmod 755 /smartmet/cnf/cron/cron.hourly/clean_data_test_obs
```

### 6.3 Triggers

Use `/smartmet/cnf/triggers.d` for routines that should run when a file or directory changes. This is useful for local GTS feeds, locally pushed observations, model files delivered by `rsync`, or post-processing that should start after another process finishes.

International documentation notes that trigger file names correspond to the triggering path, with `/` replaced by `:`.

Example concept:

```text
Triggering directory:
/smartmet/data/incoming/gts/metar

Trigger definition file name:
:smartmet:data:incoming:gts:metar
```

Use a trigger when data arrival time is irregular and the conversion should start as soon as files arrive. Use cron when the source is polled or when the model schedule is stable.

## 7. Installing packaged datasets

The fastest way to add common international datasets is to install the ready-made SmartMet data packages from the configured RPM repositories.

### 7.1 GFS

Install:

```bash
dnf install smartmet-data-gfs
```

Then edit:

```text
/smartmet/cnf/data/gfs.cnf
```

Typical settings include area name, geographic bounds, forecast intervals, and resolution. The GFS routine downloads NCEP GFS GRIB2 files, converts surface and pressure level data to QueryData, validates the result, stores `.sqd` files under `/smartmet/data/gfs/<area>/.../querydata`, and places compressed files into `/smartmet/editor/in` for SmartMet Workstation.

Useful checks:

```bash
/smartmet/run/data/gfs/bin/get_gfs.sh -d
/smartmet/run/data/gfs/bin/get_gfs.sh
ls -lh /smartmet/data/gfs/*/*/querydata
ls -lh /smartmet/editor/in/*gfs*.bz2
tail -100 /smartmet/logs/data/gfs_*.log
```

### 7.2 GEM

Install:

```bash
dnf install smartmet-data-gem
```

Check package-provided scripts and configuration under:

```text
/smartmet/run/data/gem
/smartmet/cnf/data
/smartmet/cnf/cron
```

GEM follows the same operational model as GFS: download or receive GRIB, convert to QueryData, validate, distribute, and clean old files.

### 7.3 METAR

For international installations, there are two common choices:

- `smartmet-data-metar` if the installation uses a packaged external METAR source.
- `smartmet-data-gts-metar` if local GTS METAR files are delivered to the server.

For local GTS METAR:

```bash
dnf install smartmet-data-gts-metar
```

Configure the message switch to deliver WMO FM-15 METAR messages to:

```text
/smartmet/data/incoming/gts/metar
```

The package documentation specifies WMO headers beginning with `SA`.

The conversion routine reads incoming METAR files, runs `metar2qd`, writes QueryData under:

```text
/smartmet/data/gts/metar/world/querydata
```

and copies a compressed version to:

```text
/smartmet/editor/in
```

### 7.4 SYNOP

Install:

```bash
dnf install smartmet-data-gts-synop
```

Configure the message switch to send:

```text
GTS WMO FM-12 SYNOP:
WMO headings: SM/// SI/// SN///
Directory: /smartmet/data/incoming/gts/synop

GTS WMO BUFR SYNOP:
WMO heading: IS///
Directory: /smartmet/data/incoming/gts/synop-bufr
```

The package provides conversion scripts for TAC SYNOP and BUFR SYNOP data.

### 7.5 Sounding

Install the sounding package when local GTS sounding data is available:

```bash
dnf install smartmet-data-gts-sounding
```

Check the package README and installed files for the expected incoming directory and message switch setup.

## 8. Model ingestion with `smartmet-data-models`

For model datasets beyond the older single-model packages, prefer `smartmet-data-models` where possible. It provides a shared ingestion routine for GRIB model data.

Supported model families in the public repository include:

- ECMWF
- ICON
- GFS
- GSM
- UKMO
- ARPEGE
- WRF

Install the base and a model-specific package:

```bash
dnf install smartmet-data-models smartmet-data-models-ecmwf
```

The shared command format is:

```bash
/smartmet/bin/ingest-model.sh -m <model> [-a <area>] [-t <yyyymmddThh>] [-i <input>] [-d] [-f]
```

The common package installs `ingest-model.sh` under `/smartmet/bin`. Model-specific conversion configuration remains under `/smartmet/run/data/<model>/cnf`.

Important options:

| Option | Meaning |
| --- | --- |
| `-m <model>` | Model name, required. |
| `-a <area>` | Area name. Defaults to `world`. |
| `-t <yyyymmddThh>` | Reference time to process. |
| `-i <input>` | Process a specific GRIB file. |
| `-d` | Debug mode. Converts but does not distribute final output. |
| `-f` | Force processing even if output appears complete. |

Configuration files are searched in this order:

```text
/smartmet/cnf/data/<model>-<area>.cnf
/smartmet/cnf/data/<model>.cnf
```

Example area configuration:

```bash
MODEL=ecmwf
MODEL_ID=240
MODEL_RAW_ROOT=/smartmet/data/incoming/ecmwf
MODEL_RAW_DIR=
MODEL_RAW_SFC='*ifs_sfc*${RT_DATE_MMDD}${RT_HOUR}*.grib2'
MODEL_RAW_PL='*ifs_pl*${RT_DATE_MMDD}${RT_HOUR}*.grib2'
MODEL_RAW_MASK='*.grib2'
AREA=bhutan
CROP=61,7,103,40
```

The ingestion script:

- Detects the reference time from GRIB metadata if `-t` is not given.
- Loads model or model-area configuration from `/smartmet/cnf/data`.
- Builds surface, pressure, and hybrid output paths.
- Converts with `gribtoqd`.
- Uses model-level configuration files such as `<model>-surface.cnf` and `<model>-pressure.cnf`.
- Optionally runs post-processing SmartTools scripts in `st.<level>.d`.
- Validates with `qdstat`.
- Compresses and distributes valid output to `/smartmet/data/.../querydata` and `/smartmet/editor/in`.

Recommended test sequence:

```bash
/smartmet/bin/ingest-model.sh -m ecmwf -a bhutan -d
qdinfo -P -T -x -z -r -q /smartmet/tmp/data/ecmwf_bhutan_*/*.sqd
qdstat /smartmet/tmp/data/ecmwf_bhutan_*/*.sqd
/smartmet/bin/ingest-model.sh -m ecmwf -a bhutan
```

## 9. Adding a new dataset

Use this process for any new dataset, whether it is GRIB, NetCDF, BUFR, METAR/SYNOP, CSV, or a local file format converted by a custom script.

### 9.1 Design checklist

Before writing scripts, define:

- Dataset name, for example `test_obs`, `local_synop`, `wrf_city`, or `radar_country`.
- Data owner and source URL, message switch feed, directory, or push method.
- Source format: GRIB1, GRIB2, BUFR, NetCDF, CSV, TAC text, raster, or other.
- Data type: gridded model, point observations, station observations, radar, sounding, edited data, or product.
- Area name, such as `world`, `country`, `city`, or project-specific name.
- Update frequency and expected latency.
- Required retention.
- Parameters and units.
- Station metadata, if point data.
- Producer ID and producer name.
- Output consumers: Workstation, Timeseries API, WMS, download plugin, or internal production only.

### 9.2 Create the directory structure

Example for a custom observation dataset:

```bash
DATASET=test_obs

mkdir -p /smartmet/data/incoming/$DATASET
mkdir -p /smartmet/data/$DATASET/world/querydata
mkdir -p /smartmet/run/data/$DATASET/bin
mkdir -p /smartmet/run/data/$DATASET/cnf
mkdir -p /smartmet/run/data/$DATASET/etc
mkdir -p /smartmet/logs/data
mkdir -p /smartmet/tmp/data
```

Keep source, temporary, and output files separate:

```text
/smartmet/data/incoming/<dataset>    source files
/smartmet/tmp/data/<dataset>_*       temporary working files
/smartmet/data/<dataset>/...         validated production QueryData
/smartmet/editor/in                  Workstation delivery copies
```

### 9.3 Choose the conversion tool

| Source format | Preferred conversion path |
| --- | --- |
| GRIB1/GRIB2 model data | `smartmet-data-models` or `gribtoqd`. |
| Existing packaged global models | Install package such as `smartmet-data-gfs` or `smartmet-data-gem`. |
| CSV/ASCII point data | `csv2qd`. |
| METAR TAC | `metar2qd` or packaged `smartmet-data-gts-metar`. |
| SYNOP TAC | `synop2qd` or packaged `smartmet-data-gts-synop`. |
| BUFR observations | `bufrtoqd` or packaged GTS BUFR routines. |
| CF-compliant NetCDF | `nctoqd`. |
| WRF NetCDF | `wrftoqd`. |
| Radar OPERA/HDF5/PGM | `h5toqd`, `radartoqd`, `pgm2qd`, or related tools. |

### 9.4 Validate manually first

Before scheduling, run the conversion manually as `smartmet` or with the same environment the production job will use.

```bash
sudo -iu smartmet
/smartmet/run/data/<dataset>/bin/<routine>.sh
tail -100 /smartmet/logs/data/<dataset>.log
qdstat /smartmet/data/<dataset>/<area>/querydata/<file>.sqd
qdinfo -P -T -x -z -r -q /smartmet/data/<dataset>/<area>/querydata/<file>.sqd
```

Do not add cron or triggers until the routine can run successfully by hand.

## 10. Worked example: custom CSV observations with `csv2qd`

This example is based on the local training exercise `csv2qd_excersize.txt`.

Goal:

- Fetch recent observations from a CSV/ASCII service.
- Convert selected parameters to QueryData.
- Store validated QueryData in `/smartmet/data/test_obs`.
- Copy compressed QueryData to `/smartmet/editor/in`.
- Schedule the routine hourly.

### 10.1 Input data

Example FMI Open Data request:

```text
http://opendata.fmi.fi/timeseries?producer=opendata&fmisid=100971&param=utctime,fmisid,stationname,temperature,humidity,dewpoint&format=ascii&separator=,&timesteps=24
```

For local training, modify station IDs, source URL, and parameters to match the target country or project.

### 10.2 Station metadata

Create:

```text
/smartmet/run/data/test_obs/etc/stations.csv
```

Example:

```csv
100971,100971,60.17523,24.94459,"Kaisaniemi"
101004,101004,60.20307,24.96131,"Kumpula"
```

The exact station file format must match the `csv2qd` options used by the routine.

### 10.3 Parameter metadata

Create:

```text
/smartmet/run/data/test_obs/etc/parameters.csv
```

Use a tested `csv2qd` parameter file from training or project configuration. Make sure the source CSV columns, SmartMet parameter names, units, missing value handling, and precision are correct.

### 10.4 Directory setup

```bash
mkdir -p /smartmet/data/incoming/test_obs
mkdir -p /smartmet/data/test_obs/world/querydata
mkdir -p /smartmet/run/data/test_obs/bin
mkdir -p /smartmet/run/data/test_obs/etc
mkdir -p /smartmet/logs/data
mkdir -p /smartmet/tmp/data
```

### 10.5 Conversion script

Create:

```text
/smartmet/run/data/test_obs/bin/fetch_convert.sh
```

Example:

```bash
#!/bin/bash
set -u

DATASET=test_obs
AREA=world
BASE=/smartmet
INDIR=$BASE/data/incoming/$DATASET
OUTDIR=$BASE/data/$DATASET/$AREA/querydata
EDITORDIR=$BASE/editor/in
TMPDIR=$BASE/tmp/data/${DATASET}_$(date -u +%Y%m%d%H%M)
LOGFILE=$BASE/logs/data/${DATASET}.log
ETC=$BASE/run/data/$DATASET/etc
TIMESTAMP=$(date -u +%Y%m%d%H%M)
OUTFILE=${TIMESTAMP}_${DATASET}_${AREA}.sqd

exec >> "$LOGFILE" 2>&1

echo "[$(date -u +%Y-%m-%dT%H:%M:%SZ)] Start $DATASET"

mkdir -p "$INDIR" "$OUTDIR" "$EDITORDIR" "$TMPDIR"

URL='http://opendata.fmi.fi/timeseries?producer=opendata&fmisid=100971&param=utctime,fmisid,stationname,temperature,humidity&format=ascii&separator=,&timesteps=24'

curl -fsSL "$URL" -o "$TMPDIR/data.csv"

csv2qd \
  -P "$ETC/parameters.csv" \
  -S "$ETC/stations.csv" \
  -O timeid \
  -p Temperature,Humidity \
  -i "$TMPDIR/data.csv" \
  -o "$TMPDIR/$OUTFILE"

if qdstat "$TMPDIR/$OUTFILE"; then
  bzip2 -k "$TMPDIR/$OUTFILE"
  mv -f "$TMPDIR/$OUTFILE" "$OUTDIR/"
  mv -f "$TMPDIR/$OUTFILE.bz2" "$EDITORDIR/"
  echo "[$(date -u +%Y-%m-%dT%H:%M:%SZ)] Published $OUTFILE"
else
  echo "[$(date -u +%Y-%m-%dT%H:%M:%SZ)] ERROR: invalid QueryData $OUTFILE"
  exit 1
fi

rm -rf "$TMPDIR"
```

Make it executable:

```bash
chmod 755 /smartmet/run/data/test_obs/bin/fetch_convert.sh
```

Run manually:

```bash
/smartmet/run/data/test_obs/bin/fetch_convert.sh
tail -100 /smartmet/logs/data/test_obs.log
qdinfo -P -T -x -z -r -q /smartmet/data/test_obs/world/querydata/*.sqd
```

### 10.6 Schedule it

Create:

```text
/smartmet/cnf/cron/cron.d/test_obs.cron
```

Example:

```cron
5 * * * * /smartmet/run/data/test_obs/bin/fetch_convert.sh
```

Create cleaner:

```text
/smartmet/cnf/cron/cron.hourly/clean_data_test_obs
```

Example:

```bash
#!/bin/sh
cleaner -maxfiles 48 '_test_obs_world.sqd' /smartmet/data/test_obs
cleaner -maxfiles 48 '_test_obs_world.sqd.bz2' /smartmet/editor/in
find /smartmet/data/incoming/test_obs -type f -mmin +10080 -delete
```

```bash
chmod 755 /smartmet/cnf/cron/cron.hourly/clean_data_test_obs
```

## 11. Adding local GRIB model data

Use this pattern when a country, partner, or local model system pushes GRIB files to the dataprocessing server.

### 11.1 Incoming directory

Create an incoming directory:

```bash
mkdir -p /smartmet/data/incoming/localmodel
mkdir -p /smartmet/run/data/localmodel/cnf
mkdir -p /smartmet/logs/data
mkdir -p /smartmet/tmp/data
```

Arrange file delivery with `rsync`, `sftp`, `scp`, mounted storage, or a message/data switch. The delivered files should have stable naming that includes model run time when possible.

### 11.2 Configuration

Create:

```text
/smartmet/cnf/data/localmodel-country.cnf
```

Example:

```bash
MODEL=localmodel
MODEL_ID=501
MODEL_RAW_ROOT=/smartmet/data/incoming/localmodel
MODEL_RAW_DIR=
MODEL_RAW_SFC='*${RT_DATE_HH}*surface*.grib2'
MODEL_RAW_PL='*${RT_DATE_HH}*pressure*.grib2'
MODEL_RAW_MASK='*.grib2'
AREA=country
CROP=20,50,35,72
```

### 11.3 Conversion parameter files

Create or adapt:

```text
/smartmet/run/data/localmodel/cnf/localmodel-surface.cnf
/smartmet/run/data/localmodel/cnf/localmodel-pressure.cnf
```

These files define how GRIB parameters map to QueryData parameters. Start from an existing model package with similar GRIB metadata. Verify mappings with:

```bash
grib_ls <file>.grib2
grib_get -p shortName,typeOfLevel,level,paramId <file>.grib2 | sort -u
```

Then test:

```bash
/smartmet/bin/ingest-model.sh -m localmodel -a country -i /smartmet/data/incoming/localmodel/example.grib2 -d
```

### 11.4 Publish

When debug mode output validates:

```bash
/smartmet/bin/ingest-model.sh -m localmodel -a country -f
```

Add cron or trigger only after manual processing is reliable.

## 12. Configuring data for SmartMet Server API

The dataprocessing server publishes QueryData into `/smartmet/data`. SmartMet Server API must then be configured to read those producers.

The exact API configuration depends on whether the server is deployed directly, in Docker, or in Kubernetes, and which plugins are enabled. Common tasks are:

- Add the new QueryData producer to the API server configuration.
- Ensure the API process can read the QueryData path.
- Confirm the producer name used by the API matches QueryData producer metadata.
- Decide whether the producer should be visible in Timeseries, WMS, download, or other plugins.
- Configure parameter aliases or mappings if API-facing names differ from QueryData names.
- Reload or restart the API component after configuration changes.

Use `qdinfo` to confirm producer and parameter metadata before changing API configuration:

```bash
qdinfo -P -T -x -z -r -q /smartmet/data/<producer>/<area>/<level>/querydata/<file>.sqd
```

Typical API smoke tests:

```text
https://<server>/timeseries?producer=<producer>&lonlat=<lon>,<lat>&param=place,time,temperature&format=debug
https://<server>/wms?service=WMS&request=GetCapabilities
```

For download plugin configurations, remember that QueryData parameters can be requested by common names such as `Temperature`, while grid-engine/download output may use FMI-name style parameters for grid data.

## 13. Operations and troubleshooting

### 13.1 Daily health checks

Check latest outputs:

```bash
find /smartmet/data -name '*.sqd' -type f -printf '%TY-%Tm-%Td %TH:%TM %p\n' | sort | tail -50
find /smartmet/editor/in -type f -printf '%TY-%Tm-%Td %TH:%TM %p\n' | sort | tail -50
```

Check logs:

```bash
ls -lh /smartmet/logs/data
tail -100 /smartmet/logs/data/<dataset>.log
```

Check disk usage:

```bash
df -h /smartmet
du -sh /smartmet/data/* | sort -h
du -sh /smartmet/tmp/*
```

Check cron and services:

```bash
systemctl status crond
ls -l /smartmet/cnf/cron/cron.d
ls -l /smartmet/cnf/cron/cron.hourly
```

### 13.2 Common problems

| Symptom | Likely cause | Check or fix |
| --- | --- | --- |
| No new files in `/smartmet/data` | Cron not running, source missing, script failing, wrong permissions | Check `crond`, logs, manual script run, source URL/path. |
| Files in incoming but no output | Trigger not configured, conversion fails, wrong file pattern | Run routine manually, check trigger naming, verify file masks. |
| Output exists but Workstation does not see it | File not copied to `/smartmet/editor/in`, Samba issue, permission issue | Check `.bz2` files, `smbpasswd`, Samba service, ownership. |
| API does not show dataset | API config missing, producer name mismatch, API cannot read path | Check `qdinfo`, API config, container mounts, service logs. |
| QueryData invalid | Bad parameter mapping, missing station metadata, unit mismatch, corrupt input | Run `qdstat`, `qdinfo`, inspect converter logs. |
| Disk fills up | Cleaner missing, retention too high, temp cleanup failed | Add cleaner, clean `/smartmet/tmp`, reduce retention. |
| Wrong times in API | Timezone confusion, wrong origin time, source metadata issue | Use UTC, inspect `qdinfo -T`, check source timestamps. |

### 13.3 Manual recovery pattern

1. Stop or disable the failing cron/trigger temporarily.
2. Preserve the failing input file for debugging.
3. Run the routine manually with debug output if available.
4. Inspect converter command output and logs.
5. Validate temporary `.sqd` with `qdstat` and `qdinfo`.
6. Fix configuration, not package-owned files.
7. Re-run manually.
8. Re-enable cron or trigger.

Do not modify package-provided files directly unless you accept that package updates may overwrite them. Prefer local configuration under `/smartmet/cnf` and local routine files under `/smartmet/run/data/<dataset>`.

## 14. Best practices for new production routines

- Use one clear dataset name everywhere.
- Keep configuration outside the script when settings differ between installations.
- Use `/smartmet/cnf/data/<dataset>.cnf` for installation-specific settings.
- Use `/smartmet/run/data/<dataset>/bin` for executable routines.
- Use `/smartmet/run/data/<dataset>/cnf` or `etc` for conversion tables, station files, SmartTools macros, and mappings.
- Write logs to `/smartmet/logs/data`.
- Use UTC in file names and logs.
- Use a temporary directory and publish only after validation.
- Validate with `qdstat` before moving output to production.
- Copy only final compressed Workstation files to `/smartmet/editor/in`.
- Add cleaner rules at the same time as the production routine.
- Test manually before adding cron or triggers.
- Keep a short README in each custom routine directory explaining source, schedule, owner, and output paths.

## 15. Source links and further reading

Primary sources used for this manual:

- Local training deck: `smartmet-server-training-general-dataprocessing_v1.pptx`
- Local training exercise: `csv2qd_excersize.txt`
- SmartMet international documentation: https://github.com/fmidev/smartmet-international-documentation
- SmartMet install scripts: https://github.com/fmidev/smartmet-install
- SmartMet data GFS package: https://github.com/fmidev/smartmet-data-gfs
- SmartMet data GEM package: https://github.com/fmidev/smartmet-data-gem
- SmartMet GTS SYNOP package: https://github.com/fmidev/smartmet-data-gts-synop
- SmartMet GTS METAR package: https://github.com/fmidev/smartmet-data-gts-metar
- SmartMet data models ingestion: https://github.com/fmidev/smartmet-data-models
- SmartMet QDTools: https://github.com/fmidev/smartmet-qdtools
- SmartMet Server: https://github.com/fmidev/smartmet-server
- Download plugin documentation: https://github.com/fmidev/smartmet-plugin-download

## 16. Quick reference

Install base environment:

```bash
curl -O https://raw.githubusercontent.com/fmidev/smartmet-install/master/smartmet-install-9.sh
chmod a+x smartmet-install-9.sh
./smartmet-install-9.sh
passwd smartmet
smbpasswd -a smartmet
```

Install common datasets:

```bash
dnf install smartmet-data-gfs
dnf install smartmet-data-gem
dnf install smartmet-data-gts-metar
dnf install smartmet-data-gts-synop
dnf install smartmet-data-gts-sounding
dnf install smartmet-data-models smartmet-data-models-ecmwf
```

Important paths:

```text
/smartmet/cnf/data
/smartmet/cnf/cron/cron.d
/smartmet/cnf/cron/cron.hourly
/smartmet/cnf/triggers.d
/smartmet/data/incoming
/smartmet/data
/smartmet/editor/in
/smartmet/logs/data
/smartmet/run/data
/smartmet/tmp/data
```

Validate output:

```bash
qdstat file.sqd
qdinfo -P -T -x -z -r -q file.sqd
```

Check recent production:

```bash
find /smartmet/data -name '*.sqd' -type f -printf '%TY-%Tm-%Td %TH:%TM %p\n' | sort | tail -50
tail -100 /smartmet/logs/data/<dataset>.log
df -h /smartmet
```
