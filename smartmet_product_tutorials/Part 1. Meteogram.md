
# Setting Up a Weather HTML Page Using FMI SmartMet Data

A meteogram is a graphical representation that provides a detailed view of weather conditions over a specific period for a chosen location. It typically displays multiple variables in one timeline-based chart.

***Note: The following example code is provided for instructional purposes only and should not be used in any real project. This sample code may lack the necessary security, scalability, and reliability features essential for production-level applications. Always consult official documentation and follow best practices when developing for real-world use cases.***

## Step 1: Download the SmartMet Demo Products

1. Go to the [SmartMet Demo Products GitHub page](https://github.com/fmidev/smartmet-demo-products).
2. Click on the **Code** button and select **Download ZIP** to download the repository. (Can also use git clone)
3. Extract the ZIP file to your desired folder. This will give you access to all the demo product files.

---

## Step 2: Start with the Meteogram
In the extracted folder, you’ll find two meteogram directories: **meteogram** and **meteogram-D3**. Both have the same configuration but use different charting libraries. You can choose either one, or try both.
1. In the extracted folder, navigate to the **meteogram** or **meteogram-D3** directory.
2. Open the **index.html** file in a browser.
 
   
3. Keep the browser **Console** open while developing.
   - Press `F12` or `Ctrl+Shift+I` to open the browser developer tools.
   - Select the **Console** tab to view error messages and logs during development.

---

## Step 3: Modify the Configuration

Now that the meteogram is running, you need to modify the configuration to connect it to the desired weather data.

1. In the **meteogram** folder, locate the **config.js** file.
2. Open **config.js** in a editor.
3. Find the **serviceURL** parameter and update it with the URL of the data service you want to use. For example:
   
   ```js
   serviceURL: 'https://opendata.fmi.fi/timeseries?',
   ```

4. Next, locate the **producer** parameter and update it. The producer is responsible for generating weather data, such as GFS (Global Forecast System). Example:
   
   ```js
   producer: 'gfs',
   ```

---

## Step 4: Verify that the Page Works

At this point, if the changes to **config.js** were correct, the meteogram should be able to retrieve weather data from the defined source.

1. Refresh the **index.html** page in the browser.
2. Check the **Console** to see if any errors or warnings appear. If everything is configured correctly, the weather data should display without issues, but the weather symbols are still missing.

---

## Step 5: Add Weather Symbols

Now, let’s add weather symbols to make the meteogram visually informative.

1. Download the **opendata-resources** ZIP from [this link](https://github.com/fmidev/opendata-resources).
2. Extract the ZIP file to your local machine.
3. Navigate to the **symbols/smartsymbol/light** directory in the extracted folder.
4. Copy all the files in the **light** directory.
5. In your **meteogram** folder, go to the **symbols** directory.
6. Paste all the copied files from **light** into this folder.

---

## Step 6: Test the Weather Page

1. Once you have copied the symbols, refresh the **index.html** page in your browser.
2. Check if the weather symbols are displayed correctly according to the forecast.
3. If everything is set up properly, the meteogram should now be functional, displaying both the weather data and corresponding symbols.

---

You should now have a functional weather HTML page that fetches real-time weather data and displays it using icons.
