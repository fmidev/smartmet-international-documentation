1. Select from the settings of the workstation "keep real aspect ratio" off and maximize the workstation window. 
2. Zoom into the desired area and press CTRL+m and paste into notepad. (Projection string is the first line followed by map server addresses.) 
3. Repeat this into three different areas that will be configured as 1,2 and 3 "predefined areas" (country, small and large) in the SmartMet Workstation. 
4. Use the addresses below and paste the projection string after the address and open in browser. (Paste for example this into the  map URL's below: ``&SRS=EPSG:4326&BBOX=49.2401,28.0355,92.0532,50.3377&WIDTH=2569&HEIGHT=1338``. Preferably for Natural Earth maps 60px/deg lines and for blue marble 240px/deg lines. Note that high resolution maps may not work in case the area is too large map server returns just a blank file. In this case select lower resolution string. It is recommended to avoid any maps exceeding WIDTH or HEIGHT=5000.) 
5. Fetch each map for each three different areas and save each map in ``C:\smartmet\dropbox\ COUNTRY\SmartMet\control\maps\local`` with the corresponding names (for example ``local_bluemarble_large.png``) 
6. The CTRL+m gives you also the coordinates of the area zoomed. Paste the lat-lon coordinate string for each area in ``C:\smartmet\dropbox\COUNTRY\SmartMet\control\maps\local`` into configuration files: ``project_and_coordinates_large, ...small and ...medium``.
   

**Bluemarble**

https://wms.meteo.fi/geoserver/wms?SERVICE=WMS&LAYERS=MeteoFI%3Abluemarble&STYLES=&FORMAT=image%2Fpng8&VERSION=1.1.1&REQUEST=GetMap

**Natural Earth**

http://wms.fmi.fi/fmi-apikey/1806eb12-419c-4b79-8d6c-75aed490276e/geoserver/wms?SERVICE=WMS&LAYERS=naturalearth%3Anaturalearthcolor&STYLES=&FORMAT=image%2Fpng8&VERSION=1.1.1&REQUEST=GetMap

**Natural Earth Gray**

http://wms.fmi.fi/fmi-apikey/1806eb12-419c-4b79-8d6c-75aed490276e/geoserver/wms?SERVICE=WMS&LAYERS=naturalearth%3Anaturalearthgray&STYLES=&FORMAT=image%2Fpng8&VERSION=1.1.1&REQUEST=GetMap

**Open street maps**

https://wms.fmi.fi/fmi-apikey/1806eb12-419c-4b79-8d6c-75aed490276e/geoserver/wms?SERVICE=WMS&LAYERS=osm:osm&STYLES=&FORMAT=image%2Fpng8&VERSION=1.1.1&REQUEST=GetMap

**KAP -Maps (Only in wider European area)**

7 different available, modify "version%201" with numbers 201-207

http://wms.fmi.fi/fmi-apikey/1806eb12-419c-4b79-8d6c-75aed490276e/geoserver/wms?SERVICE=WMS&layers=KAP%3ABasicMap%20version%201&STYLES=&FORMAT=image%2Fpng8&VERSION=1.1.1&REQUEST=GetMap
