# Install SmartMet Workstation

This guide walks you through installing SmartMet Workstation on a Windows computer.

## Before you begin

Make sure you have the following ready:

- A Windows computer (64-bit)
- Administrator rights on the computer (ask your IT department if unsure)
- Dropbox account credentials (username and password) provided by your administrator
- Network drive path for SmartMet data (provided by your administrator)

> **Note on placeholder names in this guide:**
> Throughout this guide, replace `COUNTRY` with your country name (e.g. `Kenya`) and `X_Y` with the latest version number (e.g. `2_6`). Your administrator will tell you which values to use.

## Installation steps

### Step 1: Install Dropbox

1. Open your web browser and go to https://www.dropbox.com/desktop
2. Download and run the Dropbox installer.
3. Log in using your organization's Dropbox account. Do not create a new personal account — your IT department can provide you with the correct credentials.
4. Before continuing, open **File Explorer** and create the folder `c:\SmartMet`. Dropbox will create its own `Dropbox` subfolder inside it, but the parent folder must exist first.
5. During setup, when asked where to place the Dropbox folder, click **Advanced Settings** and set the folder location to: `c:\SmartMet\Dropbox`
6. In the same setup dialog, **uncheck** the option **Make files available online only**. SmartMet needs the files to be stored locally on your computer to work correctly.
7. Click **OK** to confirm and finish the installation.
8. Wait for Dropbox to finish downloading all the files. You can see the sync progress in the Dropbox icon in your system tray (bottom-right corner of your screen).

### Step 2: Install fonts

SmartMet uses special fonts for weather symbols. You need to install them.

1. Open File Explorer and navigate to: `c:\SmartMet\Dropbox\COUNTRY\SmartMet\MetEditor_X_Y\`
2. Find the file **MIRRI__.ttf**. Double-click it and click the **Install** button at the top of the window that opens.
3. Do the same for the file **Synop.ttf**.

### Step 3: Install Microsoft Visual C++ Redistributable

This is a Microsoft component that SmartMet needs to run. If it is already installed on your computer, the installer will let you know and you can skip this step.

1. In File Explorer, navigate to: `c:\SmartMet\Dropbox\COUNTRY\SmartMet\MetEditor_X_Y\bin_x64\`
2. Double-click **VC_redist.x64.exe** and follow the on-screen instructions.

### Step 4: Install the graphics library (Uniras)

1. In the same `bin_x64` folder, double-click **tm752_x64.msi** to start the installer.
2. When prompted to select components, **deselect** Fortran and GSharp. Leave everything else at default settings.
3. Click through the installer to complete the installation. You may see an error message at the very end — this is normal and can be safely ignored.

### Step 5: Copy the license file

If you have been provided a license file:

1. Copy the license file to the following folder: `c:\uniras\7v5\base\`
2. If the folder does not exist, create it first.

If you do not have a license file, you can still use SmartMet without one for testing purposes. However, operational computers should have a license installed for fast graphics performance. Contact your administrator to obtain a license.

### Step 6: Create a desktop shortcut

1. Open File Explorer and navigate to: `c:\SmartMet\Dropbox\COUNTRY\SmartMet\`
2. Find the SmartMet shortcut file (it will have the SmartMet icon).
3. Right-click the shortcut and select **Copy**.
4. Go to your Desktop, right-click on an empty area, and select **Paste**.

### Step 7: Configure the shortcut

The shortcut needs to point to the correct location of your SmartMet installation.

1. On your Desktop, right-click the SmartMet shortcut and select **Properties**.
2. In the **Target** field, enter the following (replace `COUNTRY` and `X_Y` with your values):

   ```
   "c:\SmartMet\Dropbox\COUNTRY\SmartMet\MetEditor_X_Y\bin_x64\SmartMet.exe" -p "c:\SmartMet\Dropbox\COUNTRY\SmartMet\MetEditor_X_Y\Control\smartmet_COUNTRY.conf" -t "SmartMet 5.x.x.x - COUNTRY"
   ```

3. In the **Start in** field, enter:

   ```
   "c:\SmartMet\Dropbox\COUNTRY\SmartMet\MetEditor_X_Y\bin_x64"
   ```

4. Click **OK** to save the changes.

### Step 8: Map the network drive

SmartMet needs access to a shared network drive for weather data.

1. Open **File Explorer**.
2. Right-click **This PC** (in the left panel) and select **Map network drive...**.
3. In the **Drive** dropdown, select **S:**.
4. In the **Folder** field, enter the network path provided by your administrator.
5. Check the box **Reconnect at sign-in** so the drive connects automatically when you log in.
6. Check the box **Connect using different credentials**. The network drive requires specific credentials that are different from your Windows login — your administrator will provide them.
7. Click **Finish**. Windows will ask for a username and password — enter the credentials provided by your administrator.

## Troubleshooting

**SmartMet does not start when I double-click the shortcut**
- Right-click the shortcut and select **Properties**. Check that the paths in **Target** and **Start in** are correct and match your installation folder.
- Make sure Dropbox has finished syncing all files (check the Dropbox icon in your system tray).

**Files are missing or Dropbox shows files as "online only"**
- Open Dropbox **Preferences** > **Sync** and make sure the SmartMet folder is set to **Available offline**. See Step 1, sub-step 6.

**"Missing DLL" or "VCRUNTIME" error when starting SmartMet**
- Reinstall the Microsoft Visual C++ Redistributable by running `VC_redist.x64.exe` again (see Step 3).

**License error when starting SmartMet**
- Verify that your license file is located in `c:\uniras\7v5\base\`. Ask your administrator if you do not have a license file.

**Network drive S: is not accessible**
- Check that you are connected to your organization's network. If you are working remotely, make sure your VPN is connected.
- Try disconnecting and re-mapping the drive (see Step 8).

## Release Notes

* https://github.com/fmidev/smartmet-workstation/releases
