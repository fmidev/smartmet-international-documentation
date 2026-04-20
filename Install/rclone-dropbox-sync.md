# rclone Dropbox Setup (Headless Environment)

This guide explains how to install and configure **rclone** to sync data with **Dropbox** from a smarmet dataprocessing server  and automate syncing using cron. Workstation needs to also be configured to moun the local /data/in as S: drive

**You will need to have rclone installed on your local machine to get the token from dropbox.
**
---

## 1. Install rclone

```bash
sudo dnf install rclone
```

Verify installation:

```bash
rclone version
```

---

## 2. Configure rclone (Headless Setup)

Start configuration:

```bash
rclone config
```

### Step-by-step

1. Create a new remote:

   ```
   n) New remote
   ```

2. Name your remote:

   ```
   name> {countrycode}_dropbox
   ```

3. Select storage type:

   ```
   14 / Dropbox
   ```

4. Leave OAuth credentials empty (default):

   ```
   client_id>
   client_secret>
   ```

5. Skip advanced config:

   ```
   n
   ```

6. Headless authentication:

   ```
   Use web browser? n
   ```

7. On your local machine **with a browser**, run:

   ```bash
   rclone authorize "dropbox"
   ```

8. Copy the generated token and paste it:

   ```
   config_token> {PASTE_TOKEN_HERE}
   ```

9. Confirm configuration:

   ```
   y
   ```

10. Quit config:

```
q
```

---

## 3. Test the Connection

List files in Dropbox root:

```bash
rclone ls {countrycode}_dropbox:/
```

---

## 4. Prepare Remote Directory Structure

Create required directory in Dropbox:

```bash
rclone mkdir {countrycode}_dropbox:{country}/data/in
```

---

## 5. Sync Data to Dropbox

Manual sync command:

```bash
rclone -v sync /smartmet/editor/in {countrycode}_dropbox:{country}/data/in/
```

### Notes

* `sync` mirrors source → destination (deletes extra files in destination)
* Use `copy` instead if you want to avoid deletions

---

## 6. Automate with Cron

Create cron file:

```
/smartmet/cnf/cron/cron.d/dropbox
```

### Example (runs every 15 minutes)

```bash
# Sync editor/in data to Dropbox for regional offices
*/15 * * * * rclone -v sync /smartmet/editor/in/ {countrycode}_dropbox:{country}/data/in/ > /smartmet/logs/data/dropbox_sync.log
```

---

## On the Smartmet-Workstation
Map the local DropBox directory that is used for data syncing as the S-drive

```
net use s: \\localhost\c$\Smartmet\Dropbox\{country}\data
```

--- 

