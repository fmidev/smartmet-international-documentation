
# Guide: Setting Up Visual Studio Code and SSH Connections

This guide will walk you through downloading Visual Studio Code (VSC), testing an SSH connection via the terminal, and using VSC's SSH extension to connect to a remote server.

---

## Step 1: Download and Install Visual Studio Code

1. Open your browser and visit [Visual Studio Code's official website](https://code.visualstudio.com/).
2. Click the **Download** button for your operating system (Windows, macOS, or Linux).
3. Follow the installation instructions for your platform:
   - **Windows**: Run the downloaded `.exe` file and follow the setup wizard.
   - **macOS**: Open the `.dmg` file and drag VSC to your Applications folder.
   - **Linux**: Use your package manager or download the appropriate `.deb` or `.rpm` package.
4. Launch Visual Studio Code after installation.

---

## Step 2: Open the Visual Studio Code Terminal

1. Open Visual Studio Code.
2. Navigate to the **View** menu and select **Terminal**, or use the shortcut:
   - **Windows/Linux**: `Ctrl + \``
   - **macOS**: `Cmd + \``
3. The terminal will appear at the bottom of the VSC interface.


---

## Step 3: Test SSH Connection via the Terminal

1. In the VSC terminal, type the following command to test an SSH connection to a test IP:

   ```bash
   ssh user@ip-address
   ```

   Replace `user` with the username and `ip-address` with the IP of the server you’re testing.

2. If prompted, confirm the server's fingerprint by typing `yes`.

3. Enter the password when prompted.

4. If successful, you will see a welcome message from the server.

   To exit the SSH session, type:

   ```bash
   exit
   ```

---

## Step 4: Connect to a Remote Server via VSC SSH Extension

1. Open Visual Studio Code and go to the **Extensions** view by clicking the square icon on the left or pressing `Ctrl + Shift + X`.
2. Search for **Remote - SSH** in the Extensions Marketplace.
3. Click **Install** to add the extension to VSC.
4. Once installed, click the blue connection button on bottom left corner
<br><br>**OR**<br><br>
 open the **Command Palette** by pressing: `Ctrl + Shift + P` 
 and type `Remote-SSH: Connect to Host` and select it.

6. Enter your SSH connection string in the format:

   ```bash
   user@remote-server-ip
   ```

   Replace `user` with your username and `remote-server-ip` with the server's IP address.

7. If this is your first time connecting, confirm the server's fingerprint by typing `yes`.

8. Enter the password when prompted.

9. Once connected, VSC will reopen in a remote context.

---

## Step 5: Open a Folder Over SSH

1. While connected to the remote server, click **File** > **Open Folder**.
2. Browse and select a folder on the remote server.
3. Click **OK** or **Open** to open the folder in the remote workspace.

---

