# Setting Up the Weathermap: 

This example weathermap will serve as a display for the WMS product we’ll create later in this guide. By setting up the weathermap now, we’ll have a live visualization to track our progress, verify functionality, and view the product in action as we build it step by step.

***Note: The following example code is provided for instructional purposes only and should not be used in any real project. This sample code may lack the necessary security, scalability, and reliability features essential for production-level applications. Always consult official documentation and follow best practices when developing for real-world use cases.***

## Step 1: Download the SmartMet Demo Products

1. Go to the [SmartMet Demo Products GitHub page](https://github.com/fmidev/smartmet-demo-products).
2. Click on the **Code** button and select **Download ZIP** to download the repository.
3. Extract the ZIP file to your desired folder. This will give you access to all the demo product files.


## Step 2: Modify the Configuration

1. In the extracted folder, navigate to the **weathermap** directory.
2. Open the **map-config.js** file in a editor.
3. Find the **wmsserver** parameter and update it with the URL of the SmartMet-server.
   ```js
   serviceURL: 'http://[SmartMet-Server address]/wms',
   ```
4. Next, locate the **layers** list and add a new WMS-layer.
   ```js
   layers: {
    'ECMWF Total Cloud Cover': 'fmi:ecmwf:totalcloudcover',
    'ECMWF Precipitation': 'fmi:ecmwf:precipitation',
    'ECMWF 2m Temperature': 'fmi:harmonie:temperature',
    'TEST': 'test:model:temperature' // <- The added layer that will be made later
    }
   ``` 
5. Open the index.html to a browser window.

Now you should now have a functional weathermap HTML page that can display WMS-layers.

**Note**: Existing example layers are FMI-specific and might not be available on your setup. 

To identify the available layers from the Smartmet Server, perform a GetCapabilities request by navigating to the following URL in your browser:


```js
    https://[SMARTMET_SERVER]/wms?request=GetCapabilities&service=WMS
```

In the response, locate the <Layer> blocks to find layers such as *nhms:model:temperature* or *nhms:gfs:temperature*. Once identified, update map-config.js with the necessary layer configurations.