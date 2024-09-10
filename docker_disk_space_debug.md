
# Guide: Resolving Docker Disk Space Issues


## 1. Initial Assessment

- **Check Disk Usage:**
  Run the following command to identify which directories are consuming the most space:
  ```bash
  sudo du -sh /var/lib/docker/*
  ```
  This helps pinpoint where the disk usage is concentrated.

- **Inspect Docker Disk Usage:**
  Use the command below to get an overview of Docker’s disk usage:
  ```bash
  sudo docker system df
  ```
  This provides details about the size of images, containers, and volumes.

## 2. Common Causes

- **Large Container Logs:**
  Logs inside containers can grow large if not properly rotated. Restarting the container might clear these logs, but this is a temporary solution.

- **Improper Docker Setup:**
  If `/var/lib/docker` is not on a separate partition, it can fill up the root filesystem, causing significant issues.

## 3. Immediate Cleanup Steps

- **Restart the Problematic Container:**
  Temporarily free up space by restarting the container:
  ```bash
  sudo docker stop <container_id>
  sudo docker start <container_id>
  ```

- **Manually Clear Logs:**
  If restarting doesn't help, manually clear the logs by copying `/dev/null` over the large log file:
  ```bash
  sudo cp /dev/null /var/lib/docker/containers/<container_id>/<log_file>
  ```

- **Stop and Start Docker Stack:**
  Stop and start the entire Docker stack to reset log files and reclaim space:
  ```bash
  sudo docker-compose stop
  sudo docker-compose start
  ```

## 4. Using Portainer for Stack Management

- **Accessing Portainer:**
  If you have access to Portainer, you can manage Docker containers and stacks through its web interface. If Portainer is not accessible externally, ensure you can reach it via your internal network.

- **Restarting the Stack in Portainer:**
  1. Log into Portainer.
  2. Navigate to the "Stacks" section.
  3. Find the relevant stack and click `Stop`.
  4. After the stack stops, click `Start` to restart it.

- **Updating Stack Configurations:**
  If log rotation settings were updated in a Docker Compose file, use Portainer to pull the latest configuration and redeploy:
  1. Click on the stack to view its details.
  2. Click `Pull & Redeploy` to apply the new settings.

## 5. Permanent Solutions

- **Implement Log Rotation:**
  Ensure log rotation is configured to prevent excessive log file growth. Add the following to your Docker Compose files:
  ```yaml
  logging:
    driver: "json-file"
    options:
      max-size: "10m"
      max-file: "3"
  ```

- **Separate `/var/lib/docker`:**
  If possible, configure your system to mount `/var/lib/docker` on a separate partition to prevent Docker from consuming root filesystem space.

## 6. Post-Cleanup Verification

- **Verify Services:**
  After resolving the issue, ensure that all services are running correctly. Check that each service is responsive by accessing its respective endpoints.

- **Check System Status:**
  Monitor disk usage regularly to ensure the issue does not recur.

## 7. Problem Reporting

In case the issue is not resolved or if it occurs frequently, it’s important to report the problem with adequate information for troubleshooting. Include the following outputs:

- **Disk Usage Report:**
  ```bash
  sudo du -sh /var/lib/docker/*
  ```

- **Docker Disk Usage Summary:**
  ```bash
  sudo docker system df
  ```

- **List of Running Containers:**
  ```bash
  sudo docker ps
  ```

- **Specific Container Logs:**
  Include the logs for containers consuming excessive space:
  ```bash
  sudo docker logs <container_id>
  ```

- **Docker Compose File (if applicable):**
  Provide the Docker Compose file to verify configurations such as log rotation settings.

- **Relevant Screenshots from Portainer (if applicable):**
  If using Portainer, include screenshots of stack settings, container sizes, or any relevant configurations.
