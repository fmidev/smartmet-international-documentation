# Install SmartMet Workstation

1. Install Dropbox https://www.dropbox.com/desktop
3. Login with provided username and password
4. Choose Advanced settings and Configure Dropbox location to ``c:/SmartMet/Dropbox``
5. Go to directory ``c:\SmartMet\Dropbox\COUNTRY\SmartMet\MetEditor_X_Y\`` (Change COUNTRY to your country and X_Y to correct version)
6. Install font files ``MIRRI__.ttf`` and ``Synop.ttf`` (Right click -> Install)
7. Run ``bin_x64\VC_redist.x64.exe`` (if computer have this already installed, skip this step
8. Run ``bin_x64\tm752_x64.msi`` (deselect fortran and gsharp, let the configuration run with default settings, don't mind about error in the end)
9. Copy license file (if you have one) to ``c:/uniras/7v5/base`` folder and check)
10. Copy smartmet startup shortcut from ``c:\SmartMet\Dropbox\COUNTRY\SmartMet`` to desktop or to start menu
11. Open shortcut properties (right click, select properties). Modify the paths in lines "target" and "start in" of the shortcut to match with the smartmet installation path.
12. Map network drive to S: from your smartmet share

## Release Notes
* https://github.com/fmidev/smartmet-workstation/releases
