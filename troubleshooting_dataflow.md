Work-in-progress

Troubleshooting dataflow/missing data from Smartmet

# Troubleshooting

#MISSING DATA CHECKLIST

## Is all data missing/old or only certain datasets?
* If all data, then check first connection to S-drive and age of files

Purpose
* Use this when weather input data is missing.
* S-drive (Windows) and /smartmet/editor/in (Linux) are the same location.

Following instructions can be used also as internal Incident report. 

## 0) What is missing? (Incident header (fill first)) 
* Date (UTC):
* Cycle hour (00/06/12/18):
* Missing source(s): GFS / ECMWF / OBS / local observations / other

### First detected by:
1) Quick availability check (Windows + Linux)
* Check from Smartmet Workstation if data is available
* Open S-drive and verify expected newest files are visible.
* SSH to smartmet-data-processing server.
* Confirm same state in Linux path:

> cd /smartmet/editor/in
> ls -lah
> find . -maxdepth 3 -type f | tail -n 30

## Decision
* If files exist on Linux but not in Windows S-drive: mount/share issue (not download issue).
* If files missing on both: continue with source and log checks.

2) Check data logs first

- Go to:

    cd /smartmet/logs/data

    ls -ltr | tail -n 20

- Open latest relevant log(s):

    tail -n 120 <latest_log_file>

    grep -Ei "error|failed|timeout|404|403|connection|ssl|quota|disk" <latest_log_file> | tail -n 40

3) Classify failure type

- Network/source unavailable: timeouts, DNS, SSL, HTTP 5xx.
- Authentication/authorization: HTTP 401/403, token/key expired.
- Product not published yet: 404 for latest cycle only.
- Disk/full/permissions: write denied, no space left on device.
- Parser/format issue: file downloaded but rejected by converter or ingest.

4) Source-by-source checks

4A) GFS boundary data (NCEP/NOMADS)

- Verify latest cycle folder exists and file count is correct.
- Expected count for 72h with 3h step is usually 25 files.
- Check if downloads are still running (partial files, retry loops in logs).

4B) ECMWF boundary data

- Verify latest cycle folder exists.
- Confirm .converted marker exists when conversion is required.
- If GRIB files exist but .converted is missing, conversion step likely failed.

4C) Observation data (NCEP obsproc)

- Check whether obs files for the cycle were downloaded.
- If older than 2-3 days, NCEP may no longer provide these files.

4D) Local observations

- Verify country/local CSV arrived before conversion time.
- Check station list and format compatibility if conversion failed.

5) Infrastructure checks

- Disk space:

    df -h

- Inode space:

    df -i

- Permission check in data target:

    ls -ld /smartmet/editor/in

    touch /smartmet/editor/in/.write_test && rm -f /smartmet/editor/in/.write_test

- Time sync sanity:

    date -u

6) Recovery actions

- If upstream is late: wait one retry window, then re-check logs.
- If single file failed: re-download only missing step/file.
- If conversion failed: re-run conversion for affected cycle.
- If mount issue only: restart/remount S-drive side and verify visibility.
- If auth issue: renew token/credentials and re-run downloader.

7) Close incident notes

- Root cause:
- Affected sources:
- Missing time window:
- Fix applied:
- Backfill completed (yes/no):
- Preventive follow-up task:

Minimum evidence to keep
- Screenshot or text of missing directory listing.
- Latest relevant log excerpt with error lines.

- File count before and after recovery.
