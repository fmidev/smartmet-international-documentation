
# AWS Environment Setup for Smartmet Alert Visualization

This guide provides a step-by-step process to set up a Smartmet Alert visualization in an AWS environment, including enabling password login, creating a user, installing necessary packages, and configuring directories.

## 1. Enable Login with Password

1. Open and edit the Cloud configuration file:

    ```bash
    sudo vi /etc/cloud/cloud.cfg
    ```

    - Set `ssh_pwauth` to true:

    ```
    ssh_pwauth: true
    ```

2. Open and edit the SSH configuration file:

    ```bash
    sudo vi /etc/ssh/sshd_config
    ```

    - Enable `PasswordAuthentication`:

    ```
    PasswordAuthentication yes
    ```

3. Restart the SSH service:

    ```bash
    sudo service sshd restart
    ```

## 2. Create Smartmet User

1. Create a new user named `smartmet`:

    ```bash
    sudo adduser smartmet
    ```

2. Add `smartmet` to the `wheel` group for sudo privileges:

    ```bash
    sudo usermod -a -G wheel smartmet
    ```

3. Set a password for the `smartmet` user:

    ```bash
    sudo passwd smartmet
    ```

## 3. Install Necessary Packages

Install `git`, `httpd`, and `php`:

```bash
sudo dnf install git httpd php8.2
```

## 4. Start and Enable Apache (httpd)

1. Start Apache:

    ```bash
    sudo systemctl start httpd
    ```

2. Enable Apache to start on boot:

    ```bash
    sudo systemctl enable httpd
    ```

## 5. Set Up Directories (Assuming Smartmet Environment Is Not Installed)

1. Create required directories:

    ```bash
    sudo mkdir /smartmet
    sudo mkdir -p /smartmet/editor/smartalert
    ```

2. Change ownership of the `/smartmet` directory to `smartmet`:

    ```bash
    sudo chown -R smartmet:smartmet /smartmet/
    ```

3. Navigate to the Apache `www` directory and change permissions:

    ```bash
    cd /var/www/html/
    sudo chown smartmet .
    ```

## 6. Clone the Smartmet-Alert Git Repository

Clone the repository to the `/var/www/html` directory:

```bash
git clone https://github.com/fmidev/smartalert-web
```

## 7. Create a Symlink to the Data Directory

1. Navigate to the cloned repository:

    ```bash
    cd /var/www/html/smartalert-web
    ```

2. Create a symlink to the data directory:

    ```bash
    ln -s /smartmet/editor/smartalert data
    ```

## 8. Modify capmap-config.js (Country Dependent)

1. Fetch the `capmap-config.js` file:

    ```bash
    wget http://smartmet-demo.fmi.fi/smartalert-ke/capmap-config.js
    ```

## 9. Modify index.html to Add Translation Files

Add the following lines in `index.html` to include translation files:

```html
<script type="text/javascript" src="i18n/translations-en-KE.js"></script>
<script type="text/javascript" src="i18n/translations-ke-KE.js"></script>
```

Replace en-KE and ke-KE in the file names with the appropriate 2-letter ISO country code for the target country.

## 10. Smartmet Alert on Windows

1. Install the correct version of Smartmet Alert.

2. Mount `/smartmet/editor` as the S-drive using SSHFS (e.g., use [WinFSP SSHFS](https://github.com/winfsp/sshfs-win) and [SSHFS-Win Manager](https://github.com/evsar3/sshfs-win-manager)).



## Next Steps

1. Configure the web server with SSL.
