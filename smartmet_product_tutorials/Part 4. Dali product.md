
# Dali Product Creation Guide

This guide will walk you through step-by-step how to build a weather visualization Dali product. We will progressively add features to the code, allowing you to see changes at each stage and understand how each part of the code works.


## Step 0: Setup

### Setting Up Resources for Dali Products

To create a fully functioning Dali product, various resources are required, such as configuration files, CSS styles, and mapping data. You can either add these resources manually or copy them from an existing template. Below are the steps to get started:

#### 1. Clone the Template Repository

To simplify the setup, you can clone the **SmartMet WMS Templates** repository, which contains a collection of templates for Dali products:

```bash
git clone https://github.com/fmidev/smartmet-server-wms-templates
```

This repository includes all necessary resources such as isobands, symbols, and configuration files.

#### 2. (Optional) Using the Development Version of SmartMet-Server
If you are working with a development version of SmartMet-Server, follow the instructions in the README of the cloned repository to set up the environment and launch the demo product. This will ensure you have all the necessary paths and configurations correctly set up. 

#### 3. Using the Production Version of SmartMet-Server
If you are using the production version of SmartMet-Server, you should avoid changing any existing paths in the production environment. Instead, add needed files for the demo product.

Create the following file structure. You can copy and paste the required folders and files into your existing setup. Be careful not to override any existing folders. If a folder (e.g., `dali/customers`) already exists, do not replace it. Instead, copy only the required subfolders. For example, if you're adding a demo for a specific customer, copy just the `test` folder into the `customers` folder:
   



    dali
    ├── customers
    │   └── test
    │       ├── layers
    │       │   ├── maps
    │       │   │   └── map.css
    │       │   └── symbols
    │       │       ├── dot
    │       │       └── windarrow
    │       └── products
    │           └── demo.json
    └── resources
        ├── arrows
        │   ├── windarrow.css
        │   └── windarrow.json
        ├── layers
        │   ├── isobands
        │   │   ├── FFG24H-MS.css
        │   │   ├── FFG24H-MS.json
        │   │   ├── Temperature.css
        │   │   └── Temperature.json
        │   ├── numbers
        │   │   └── numbers.css
        │   └── symbols
        │       ├── smartweather.css
        │       └── smartweather.json
        ├── products
        │   ├── common_defs.json
        │   └── map_product_refs.json
        └── symbols
            ├── dot
            ├── weather
            │   └── smartsymbol
            │       ├── day
            │       │   └── *
            │       └── night
            │           └── *
            └── windarrow
            
#### 4. Start a New Demo Product


Create new dali product called "myDemo" for test customer and open it.

     dali
        ├── customers
        │   └── test
        │       └── products
        │           ├── demo.json
        │           └── myDemo.json


To monitor the progress of the product during development, open the product in a browser window. 

Example of the URL: <br>
http://[SmartMet-Server address]/dali?customer=test&product=myDemo&time=202410171200&type=png


Use today's date in place of the timestamp "202410171200".
                                            
## Step 1: Basic Map

### Add the projection and initial map dimensions

We begin by setting up the basic structure of the map. This defines the width, height, and geographic projection, which determines how the map is rendered based on latitude and longitude.

```json
{
  "title": "Demo",
  "abstract": "Longer description",
  "width": 1200,
  "height": 1600,
  "refs": "json:products/map_product_refs.json",
  "defs": "json:products/common_defs.json",
  "projection": {
    "xsize": 600,
    "ysize": 800,
    "crs": "EPSG:4326",
    "x1": 18,
    "y1": 58,
    "x2": 33,
    "y2": 71
  }
}
````
#### Explanation:

-   **`width` and `height`:** Defines the overall dimensions of the map (1200x1600 pixels).
-   **`projection`:** Sets the coordinate reference system (`EPSG:4326`), which is a standard geographic coordinate system based on latitude and longitude.
-   **`x1`, `y1`, `x2`, `y2`:** Defines the geographical extent of the map (from longitude 18°E to 33°E, and latitude 58°N to 71°N).

At this stage, you will see a blank map canvas.

----------

## Step 2: Adding the First Map (Temperature Visualization)

### Add temperature visualization using isobands

In this step, we add the first view, a map that visualizes temperature data. We use isobands to represent temperature ranges. Isobands are essentially contour lines or filled regions that group areas of the same temperature.

```json

"views": [
  {
    "qid": "map_1",
    "attributes": {
      "transform": "translate(0 0)"
    },
    "layers": [
      {
        "qid": "temperature_visualization_1",
        "producer": "gfs",
        "layer_type": "isoband",
        "parameter": "temperature",
        "isobands": "json:isobands/Temperature.json",
        "css": "isobands/Temperature.css",
        "attributes": {
          "shape-rendering": "crispEdges"
        }
      }
    ]
  }
]
```
#### Explanation:

-   **`layer_type`:** Specifies the type of layer being added to the map. Here, we are using `"isoband"`, which will visualize temperature in bands or regions.
-   **`isobands`:** Defines how the temperature ranges are grouped and colored. The ranges and colors are defined in `Temperature.json`.
-   **`css`:** Links to a CSS file (`Temperature.css`) that controls the appearance of the isobands.

At this point, the map will display temperature data as color-coded bands.

<details>
 <summary>Show image for this part</summary>
  
 <img src="https://raw.githubusercontent.com/fmidev/smartmet-international-documentation/refs/heads/main/docs/images/dali_1.png" alt="dali step 1" style="height: 75%; width:75%;"/>

</details>

<details>
 <summary>Show full code for this part</summary>
  

 ```json
{
  "title": "Demo",
  "abstract": "Longer description",
  "width": 1200,
  "height": 1600,
  "refs": "json:products/map_product_refs.json",
  "defs": "json:products/common_defs.json",
  "projection": {
    "xsize": 600,
    "ysize": 800,
    "crs": "EPSG:4326",
    "x1": 18,
    "y1": 58,
    "x2": 33,
    "y2": 71
  },
  "views": [
    {
      "qid": "map_1",
      "attributes": {
        "transform": "translate(0 0)"
      },
      "layers": [
        {
          "qid": "temperature_visualization_1",
          "producer": "gfs",
          "layer_type": "isoband",
          "parameter": "temperature",
          "isobands": "json:isobands/Temperature.json",
          "css": "isobands/Temperature.css",
          "attributes": {
            "shape-rendering": "crispEdges"
          }
        }
      ]
    }
  ]
}
```
 
</details>

----------

## Step 3: Adding Temperature Numbers

### Add temperature values displayed as numbers

In addition to visualizing temperature with isobands, we can also show the exact temperature values as numbers on the map.

```json

{
  "qid": "temperature_numbers_1",
  "producer": "gfs",
  "layer_type": "number",
  "css": "numbers/numbers.css",
  "parameter": "temperature",
  "label": {
    "dx": 0,
    "dy": 3,
    "precision": 0,
    "missing": "-"
  },
  "positions": {
    "x": 20,
    "y": 18,
    "dx": 30,
    "dy": 30,
    "ddx": 15
  }
}
```

#### Explanation:

-   **`layer_type: number`:** This layer displays the temperature as numeric values.
-   **`positions`:** Specifies how the temperature numbers are positioned on the map grid.
    -   **`x` and `y`:** Define the starting point for placing the numbers.
    -   **`dx` and `dy`:** Define the spacing between each number horizontally and vertically.
    -   **`ddx`:** Adjusts the offset for alternating rows.

Now, you will see temperature numbers placed on top of the isoband map.

<details>
 <summary>Show image for this part</summary>
  
 <img src="https://raw.githubusercontent.com/fmidev/smartmet-international-documentation/refs/heads/main/docs/images/dali_2.png" alt="dali step 2" style="height: 75%; width:75%;"/>

</details>

<details>
 <summary>Show full code for this part</summary>
  

 ```json

{
  "title": "Demo",
  "abstract": "Longer description",
  "width": 1200,
  "height": 1600,
  "refs": "json:products/map_product_refs.json",
  "defs": "json:products/common_defs.json",
  "projection": {
    "xsize": 600,
    "ysize": 800,
    "crs": "EPSG:4326",
    "x1": 18,
    "y1": 58,
    "x2": 33,
    "y2": 71
  },
  "views": [
    {
      "qid": "map_1",
      "attributes": {
        "transform": "translate(0 0)"
      },
      "layers": [
        {
          "qid": "temperature_visualization_1",
          "producer": "gfs",
          "layer_type": "isoband",
          "parameter": "temperature",
          "isobands": "json:isobands/Temperature.json",
          "css": "isobands/Temperature.css",
          "attributes": {
            "shape-rendering": "crispEdges"
          }
        },
        {
          "qid": "temperature_numbers_1",
          "producer": "gfs",
          "layer_type": "number",
          "css": "numbers/numbers.css",
          "parameter": "temperature",
          "label": {
            "dx": 0,
            "dy": 3,
            "precision": 0,
            "missing": "-"
          },
          "positions": {
            "x": 20,
            "y": 18,
            "dx": 30,
            "dy": 30,
            "ddx": 15
          }
        }
      ]
    }
  ]
}
```
 
</details>

----------

## Step 4: Adding a City Marker (Helsinki)

### Add a circle symbol for Helsinki

Next, we will add a marker symbol to represent the city of Helsinki on the map.

```json

`{
  "qid": "city_marker_1",
  "layer_type": "symbol",
  "positions": {
    "layout": "latlon",
    "locations": [
      {
        "longitude": 24.93417,
        "latitude": 60.17556
      }
    ],
    "dx": 0,
    "dy": 0
  },
  "symbol": "dot",
  "scale": 5
}
```

#### Explanation:

-   **`layer_type: symbol`:** Specifies that we are adding a symbol layer.
-   **`positions`:** Places the symbol using latitude and longitude coordinates. Here, we use the coordinates of Helsinki (longitude: 24.93417, latitude: 60.17556).
-   **`symbol`:** Defines the type of symbol used, in this case, a small dot.
-   **`scale`:** Controls the size of the symbol (5 units in this case).

This step will add a small dot representing Helsinki on the map.

<details>
 <summary>Show image for this part</summary>
  
 <img src="https://raw.githubusercontent.com/fmidev/smartmet-international-documentation/refs/heads/main/docs/images/dali_3.png" alt="dali step 3" style="height: 75%; width:75%;"/>

</details>

<details>
 <summary>Show full code for this part</summary>
  

 ```json

{
  "title": "Demo",
  "abstract": "Longer description",
  "width": 1200,
  "height": 1600,
  "refs": "json:products/map_product_refs.json",
  "defs": "json:products/common_defs.json",
  "projection": {
    "xsize": 600,
    "ysize": 800,
    "crs": "EPSG:4326",
    "x1": 18,
    "y1": 58,
    "x2": 33,
    "y2": 71
  },
  "views": [
    {
      "qid": "map_1",
      "attributes": {
        "transform": "translate(0 0)"
      },
      "layers": [
        {
          "qid": "temperature_visualization_1",
          "producer": "gfs",
          "layer_type": "isoband",
          "parameter": "temperature",
          "isobands": "json:isobands/Temperature.json",
          "css": "isobands/Temperature.css",
          "attributes": {
            "shape-rendering": "crispEdges"
          }
        },
        {
          "qid": "temperature_numbers_1",
          "producer": "gfs",
          "layer_type": "number",
          "css": "numbers/numbers.css",
          "parameter": "temperature",
          "label": {
            "dx": 0,
            "dy": 3,
            "precision": 0,
            "missing": "-"
          },
          "positions": {
            "x": 20,
            "y": 18,
            "dx": 30,
            "dy": 30,
            "ddx": 15
          }
        },
        {
          "qid": "city_marker_1",
          "layer_type": "symbol",
          "positions": {
            "layout": "latlon",
            "locations": [
              {
                "longitude": 24.93417,
                "latitude": 60.17556
              }
            ],
            "dx": 0,
            "dy": 0
          },
          "symbol": "dot",
          "scale": 5
        }
      ]
    }
  ]
}
```
 
</details>

----------

## Step 5: Adding Europe Country Borders

### Overlay country borders on the map

To enhance the map's context, we can add country borders by referencing a pre-defined boundary dataset.

```json

"ref:refs.Europe_boundaries"
```
#### Explanation:

-   **`ref:refs.Europe_boundaries`:** This references the country borders dataset from the `map_product_refs.json` file, which contains geographical boundaries for Europe.

Now, country borders will be visible on top of the temperature visualization.

<details>
 <summary>Show image for this part</summary>
  
 <img src="https://raw.githubusercontent.com/fmidev/smartmet-international-documentation/refs/heads/main/docs/images/dali_4.png" alt="dali step 4" style="height: 75%; width:75%;"/>

</details>

<details>
 <summary>Show full code for this part</summary>
  

 ```json

{
  "title": "Demo",
  "abstract": "Longer description",
  "width": 1200,
  "height": 1600,
  "refs": "json:products/map_product_refs.json",
  "defs": "json:products/common_defs.json",
  "projection": {
    "xsize": 600,
    "ysize": 800,
    "crs": "EPSG:4326",
    "x1": 18,
    "y1": 58,
    "x2": 33,
    "y2": 71
  },
  "views": [
    {
      "qid": "map_1",
      "attributes": {
        "transform": "translate(0 0)"
      },
      "layers": [
        {
          "qid": "temperature_visualization_1",
          "producer": "gfs",
          "layer_type": "isoband",
          "parameter": "temperature",
          "isobands": "json:isobands/Temperature.json",
          "css": "isobands/Temperature.css",
          "attributes": {
            "shape-rendering": "crispEdges"
          }
        },
        {
          "qid": "temperature_numbers_1",
          "producer": "gfs",
          "layer_type": "number",
          "css": "numbers/numbers.css",
          "parameter": "temperature",
          "label": {
            "dx": 0,
            "dy": 3,
            "precision": 0,
            "missing": "-"
          },
          "positions": {
            "x": 20,
            "y": 18,
            "dx": 30,
            "dy": 30,
            "ddx": 15
          }
        },
        {
          "qid": "city_marker_1",
          "layer_type": "symbol",
          "positions": {
            "layout": "latlon",
            "locations": [
              {
                "longitude": 24.93417,
                "latitude": 60.17556
              }
            ],
            "dx": 0,
            "dy": 0
          },
          "symbol": "dot",
          "scale": 5
        },
        "ref:refs.Europe_boundaries"
      ]
    }
  ]
}
```
 
</details>

----------


## Step 6: Adding Wind Speed and Direction

### Visualize wind speed and direction with arrows and numbers

In this step, we will create the second map to visualize wind speed and direction. Wind speed is shown using numbers, while arrows are used to represent wind direction.

```json
{
  "qid": "map_2",
  "attributes": {
    "id": "view2",
    "transform": "translate(600 0)"
  },
  "layers": [
    {
      "qid": "WindSpeed-number",
      "producer": "gfs",
      "layer_type": "number",
      "css": "numbers/numbers.css",
      "parameter": "WindSpeedMS",
      "label": {
        "dx": 0,
        "dy": 3,
        "precision": 1,
        "missing": "-"
      },
      "positions": {
        "x": 5,
        "y": 8,
        "dx": 50,
        "dy": 50,
        "ddx": 21
      }
    },
    {
      "qid": "wind_arrow",
      "producer": "gfs",
      "layer_type": "arrow",
      "css": "arrows/windarrow.css",
      "u": "WindUMS",
      "v": "WindVMS",
      "attributes": {
        "id": "wind_arrows",
        "class": "WindArrow"
      },
      "arrows": "json:arrows/windarrow.json",
      "positions": {
        "x": 12,
        "y": 20,
        "dx": 50,
        "dy": 50,
        "ddx": 21
      }
    },
    "ref:refs.Europe_boundaries"
  ]
}
```

### Explanation:
- **Wind Speed Numbers:**
  - The **`layer_type: number`** displays wind speed at specific points as numbers.
  - **`positions`**: Defines how the wind speed numbers are spaced out on the grid.
  
- **Wind Direction Arrows:**
  - The **`layer_type: arrow`** visualizes wind direction. Arrows point in the direction of the wind.
  - **`u` and `v`**: Represent the components of wind in the horizontal and vertical directions.

At this stage, your map will visualize wind speed using both numbers and arrows.

<details>
 <summary>Show image for this part</summary>
  
 <img src="https://raw.githubusercontent.com/fmidev/smartmet-international-documentation/refs/heads/main/docs/images/dali_5.png" alt="dali step 5" style="height: 75%; width:75%;"/>

</details>

<details>
 <summary>Show full code for this part</summary>
  

 ```json

{
  "title": "Demo",
  "abstract": "Longer description",
  "width": 1200,
  "height": 1600,
  "refs": "json:products/map_product_refs.json",
  "defs": "json:products/common_defs.json",
  "projection": {
    "xsize": 600,
    "ysize": 800,
    "crs": "EPSG:4326",
    "x1": 18,
    "y1": 58,
    "x2": 33,
    "y2": 71
  },
  "views": [
    {
      "qid": "map_1",
      "attributes": {
        "transform": "translate(0 0)"
      },
      "layers": [
        {
          "qid": "temperature_visualization_1",
          "producer": "gfs",
          "layer_type": "isoband",
          "parameter": "temperature",
          "isobands": "json:isobands/Temperature.json",
          "css": "isobands/Temperature.css",
          "attributes": {
            "shape-rendering": "crispEdges"
          }
        },
        {
          "qid": "temperature_numbers_1",
          "producer": "gfs",
          "layer_type": "number",
          "css": "numbers/numbers.css",
          "parameter": "temperature",
          "label": {
            "dx": 0,
            "dy": 3,
            "precision": 0,
            "missing": "-"
          },
          "positions": {
            "x": 20,
            "y": 18,
            "dx": 30,
            "dy": 30,
            "ddx": 15
          }
        },
        {
          "qid": "city_marker_1",
          "layer_type": "symbol",
          "positions": {
            "layout": "latlon",
            "locations": [
              {
                "longitude": 24.93417,
                "latitude": 60.17556
              }
            ],
            "dx": 0,
            "dy": 0
          },
          "symbol": "dot",
          "scale": 5
        },
        "ref:refs.Europe_boundaries"

      ]
    },
    {
      "qid": "map_2",
      "attributes": {
        "id": "view2",
        "transform": "translate(600 000)"
      },
      "layers": [
        {
          "qid": "WindSpeed-number",
          "producer": "gfs",
          "layer_type": "number",
          "css": "numbers/numbers.css",
          "parameter": "WindSpeedMS",
          "label": {
            "dx": 0,
            "dy": 3,
            "precision": 1,
            "missing": "-"
          },
          "positions": {
            "x": 5,
            "y": 8,
            "dx": 50,
            "dy": 50,
            "ddx": 21
          }
        },
        {
          "qid": "wind_arrow",
          "producer": "gfs",
          "layer_type": "arrow",
          "css": "arrows/windarrow.css",
          "u": "WindUMS",
          "v": "WindVMS",
          "attributes": {
            "id": "wind_arrows",
            "class": "WindArrow"
          },
          "arrows": "json:arrows/windarrow.json",
          "positions": {
            "x": 12,
            "y": 20,
            "dx": 50,
            "dy": 50,
            "ddx": 21
          }
        },
        {
          "layer_type": "symbol",
          "positions": {
            "layout": "latlon",
            "locations": [
              {
                "longitude": 24.93417,
                "latitude": 60.17556
              }
            ],
            "dx": 0,
            "dy": 0
          },
          "symbol": "dot",
          "scale": 5
        },
        "ref:refs.Europe_boundaries"
      ]
    }
  ]
}
```
 
</details>

---

## Step 7: Adding Weather Symbols for Multiple Cities

### Display weather conditions using smart symbols

In this step, we will create a new map view to display weather conditions using symbols (e.g., sunny, rainy). These symbols will appear for various cities.

```json
{
  "qid": "map_3",
  "attributes": {
    "id": "view3",
    "transform": "translate(0 800)"
  },
  "layers": [
    "ref:refs.Europe_boundaries",
    {
      "qid": "weathersymbol-1",
      "producer": "gfs",
      "layer_type": "symbol",
      "css": "symbols/smartweather.css",
      "symbols": "json:symbols/smartweather.json",
      "parameter": "smartsymbol",
      "scale": 1.4,
      "positions": {
        "layout": "latlon",
        "locations": [
          {
            "longitude": 24.941325,
            "latitude": 60.173437
          },
          {
            "longitude": 22.264824,
            "latitude": 60.45451
          },
          {
            "longitude": 25.747257,
            "latitude": 62.242603
          },
          {
            "longitude": 25.726967,
            "latitude": 66.503059
          },
          {
            "longitude": 25.469885,
            "latitude": 65.021545
          },
          {
            "longitude": 29.736916,
            "latitude": 62.607662
          },
          {
            "longitude": 27.414838,
            "latitude": 68.419817
          }
        ],
        "dx": 0,
        "dy": 0
      },
      "attributes": {
        "id": "weather_1",
        "class": "Weather"
      }
    }
  ]
}
```

### Explanation:
- **Smart Symbols:**
  - The **`layer_type: symbol`** shows different weather symbols at various geographic locations.
  - The **`positions`** object defines the latitude and longitude of cities where the symbols will appear.
  - **`scale: 1.4`** sets the size of the weather symbols.

Weather symbols such as sun, clouds, or rain will now be visible on the map for cities like Helsinki, Turku, and others.

<details>
 <summary>Show image for this part</summary>
  
 <img src="https://raw.githubusercontent.com/fmidev/smartmet-international-documentation/refs/heads/main/docs/images/dali_6.png" alt="dali step 6" style="height: 75%; width:75%;"/>

</details>

<details>
 <summary>Show full code for this part</summary>
  

 ```json

{
  "title": "Demo",
  "abstract": "Longer description",
  "width": 1200,
  "height": 1600,
  "refs": "json:products/map_product_refs.json",
  "defs": "json:products/common_defs.json",
  "projection": {
    "xsize": 600,
    "ysize": 800,
    "crs": "EPSG:4326",
    "x1": 18,
    "y1": 58,
    "x2": 33,
    "y2": 71
  },
  "views": [
    {
      "qid": "map_1",
      "attributes": {
        "transform": "translate(0 0)"
      },
      "layers": [
        {
          "qid": "temperature_visualization_1",
          "producer": "gfs",
          "layer_type": "isoband",
          "parameter": "temperature",
          "isobands": "json:isobands/Temperature.json",
          "css": "isobands/Temperature.css",
          "attributes": {
            "shape-rendering": "crispEdges"
          }
        },
        {
          "qid": "temperature_numbers_1",
          "producer": "gfs",
          "layer_type": "number",
          "css": "numbers/numbers.css",
          "parameter": "temperature",
          "label": {
            "dx": 0,
            "dy": 3,
            "precision": 0,
            "missing": "-"
          },
          "positions": {
            "x": 20,
            "y": 18,
            "dx": 30,
            "dy": 30,
            "ddx": 15
          }
        },
        {
          "qid": "city_marker_1",
          "layer_type": "symbol",
          "positions": {
            "layout": "latlon",
            "locations": [
              {
                "longitude": 24.93417,
                "latitude": 60.17556
              }
            ],
            "dx": 0,
            "dy": 0
          },
          "symbol": "dot",
          "scale": 5
        },
        "ref:refs.Europe_boundaries"

      ]
    },
    {
      "qid": "map_2",
      "attributes": {
        "id": "view2",
        "transform": "translate(600 000)"
      },
      "layers": [
        {
          "qid": "WindSpeed-number",
          "producer": "gfs",
          "layer_type": "number",
          "css": "numbers/numbers.css",
          "parameter": "WindSpeedMS",
          "label": {
            "dx": 0,
            "dy": 3,
            "precision": 1,
            "missing": "-"
          },
          "positions": {
            "x": 5,
            "y": 8,
            "dx": 50,
            "dy": 50,
            "ddx": 21
          }
        },
        {
          "qid": "wind_arrow",
          "producer": "gfs",
          "layer_type": "arrow",
          "css": "arrows/windarrow.css",
          "u": "WindUMS",
          "v": "WindVMS",
          "attributes": {
            "id": "wind_arrows",
            "class": "WindArrow"
          },
          "arrows": "json:arrows/windarrow.json",
          "positions": {
            "x": 12,
            "y": 20,
            "dx": 50,
            "dy": 50,
            "ddx": 21
          }
        },
        {
          "layer_type": "symbol",
          "positions": {
            "layout": "latlon",
            "locations": [
              {
                "longitude": 24.93417,
                "latitude": 60.17556
              }
            ],
            "dx": 0,
            "dy": 0
          },
          "symbol": "dot",
          "scale": 5
        },
        "ref:refs.Europe_boundaries"
      ]
    },
    {
      "qid": "map_3",
      "attributes": {
        "id": "view3",
        "transform": "translate(0 800)"
      },
      "layers": [
        "ref:refs.Europe_boundaries",
        {
          "qid": "weathersymbol-1",
          "producer": "gfs",
          "layer_type": "symbol",
          "css": "symbols/smartweather.css",
          "symbols": "json:symbols/smartweather.json",
          "parameter": "smartsymbol",
          "scale": 1.4,
          "positions": {
            "layout": "latlon",
            "locations": [
              {
                "longitude": 24.941325,
                "latitude": 60.173437
              },
              {
                "longitude": 22.264824,
                "latitude": 60.45451
              },
              {
                "longitude": 25.747257,
                "latitude": 62.242603
              },
              {
                "longitude": 25.726967,
                "latitude": 66.503059
              },
              {
                "longitude": 25.469885,
                "latitude":  65.021545
              },
              {
                "longitude": 29.736916,
                "latitude":  62.607662
              },
              {
                "longitude": 27.414838,
                "latitude":  68.419817
              }
            ],
            "dx": 0,
            "dy": 0
          },
          "attributes": {
            "id": "weather_1",
            "class": "Weather"
          }
        }
      ]
    }
  ]
}
```
 
</details>

---

## Step 8: Adding a Time-Offset Map (6 Hours Ahead)

### Create a forecast map with a time offset

In this step, we will create another map similar to the first one, but it will show a forecast with a time offset of 6 hours.

```json
{
  "time_offset": 360,
  "qid": "map_4",
  "attributes": {
    "id": "view4",
    "transform": "translate(600 800)"
  },
  "layers": [
    {
      "qid": "temperature_visualization_4",
      "producer": "gfs",
      "layer_type": "isoband",
      "parameter": "temperature",
      "isobands": "json:isobands/Temperature.json",
      "css": "isobands/Temperature.css"
    },
    {
      "qid": "Temperature-numbers-4",
      "producer": "gfs",
      "layer_type": "number",
      "css": "numbers/numbers.css",
      "parameter": "temperature",
      "label": {
        "dx": 0,
        "dy": 3,
        "precision": 0,
        "missing": "-"
      },
      "positions": {
        "x": 20,
        "y": 18,
        "dx": 30,
        "dy": 30,
        "ddx": 15
      }
    },
    {
      "layer_type": "symbol",
      "positions": {
        "layout": "latlon",
        "locations": [
          {
            "longitude": 24.93417,
            "latitude": 60.17556
          }
        ],
        "dx": 0,
        "dy": 0
      },
      "symbol": "dot",
      "scale": 5
    },
    "ref:refs.Europe_boundaries"
  ]
}
```

### Explanation:
- **Time Offset**: 
  - The key addition here is the **`time_offset: 360`** parameter, which sets the forecast 6 hours into the future.
  
Now, you will see a map that predicts temperatures 6 hours ahead of the current time.

<details>
 <summary>Show image for this part</summary>
  
 <img src="https://raw.githubusercontent.com/fmidev/smartmet-international-documentation/refs/heads/main/docs/images/dali_7.png" alt="dali step 7" style="height: 75%; width:75%;"/>

</details>

<details>
 <summary>Show full code</summary>
  

 ```json

{
  "title": "Demo",
  "abstract": "Longer description",
  "width": 1200,
  "height": 1600,
  "refs": "json:products/map_product_refs.json",
  "defs": "json:products/common_defs.json",
  "projection": {
    "xsize": 600,
    "ysize": 800,
    "crs": "EPSG:4326",
    "x1": 18,
    "y1": 58,
    "x2": 33,
    "y2": 71
  },
  "views": [
    {
      "qid": "map_1",
      "attributes": {
        "transform": "translate(0 0)"
      },
      "layers": [
        {
          "qid": "temperature_visualization_1",
          "producer": "gfs",
          "layer_type": "isoband",
          "parameter": "temperature",
          "isobands": "json:isobands/Temperature.json",
          "css": "isobands/Temperature.css",
          "attributes": {
            "shape-rendering": "crispEdges"
          }
        },
        {
          "qid": "temperature_numbers_1",
          "producer": "gfs",
          "layer_type": "number",
          "css": "numbers/numbers.css",
          "parameter": "temperature",
          "label": {
            "dx": 0,
            "dy": 3,
            "precision": 0,
            "missing": "-"
          },
          "positions": {
            "x": 20,
            "y": 18,
            "dx": 30,
            "dy": 30,
            "ddx": 15
          }
        },
        {
          "qid": "city_marker_1",
          "layer_type": "symbol",
          "positions": {
            "layout": "latlon",
            "locations": [
              {
                "longitude": 24.93417,
                "latitude": 60.17556
              }
            ],
            "dx": 0,
            "dy": 0
          },
          "symbol": "dot",
          "scale": 5
        },
        "ref:refs.Europe_boundaries"

      ]
    },
    {
      "qid": "map_2",
      "attributes": {
        "id": "view2",
        "transform": "translate(600 000)"
      },
      "layers": [
        {
          "qid": "WindSpeed-number",
          "producer": "gfs",
          "layer_type": "number",
          "css": "numbers/numbers.css",
          "parameter": "WindSpeedMS",
          "label": {
            "dx": 0,
            "dy": 3,
            "precision": 1,
            "missing": "-"
          },
          "positions": {
            "x": 5,
            "y": 8,
            "dx": 50,
            "dy": 50,
            "ddx": 21
          }
        },
        {
          "qid": "wind_arrow",
          "producer": "gfs",
          "layer_type": "arrow",
          "css": "arrows/windarrow.css",
          "u": "WindUMS",
          "v": "WindVMS",
          "attributes": {
            "id": "wind_arrows",
            "class": "WindArrow"
          },
          "arrows": "json:arrows/windarrow.json",
          "positions": {
            "x": 12,
            "y": 20,
            "dx": 50,
            "dy": 50,
            "ddx": 21
          }
        },
        {
          "layer_type": "symbol",
          "positions": {
            "layout": "latlon",
            "locations": [
              {
                "longitude": 24.93417,
                "latitude": 60.17556
              }
            ],
            "dx": 0,
            "dy": 0
          },
          "symbol": "dot",
          "scale": 5
        },
        "ref:refs.Europe_boundaries"
      ]
    },
    {
      "qid": "map_3",
      "attributes": {
        "id": "view3",
        "transform": "translate(0 800)"
      },
      "layers": [
        "ref:refs.Europe_boundaries",
        {
          "qid": "weathersymbol-1",
          "producer": "gfs",
          "layer_type": "symbol",
          "css": "symbols/smartweather.css",
          "symbols": "json:symbols/smartweather.json",
          "parameter": "smartsymbol",
          "scale": 1.4,
          "positions": {
            "layout": "latlon",
            "locations": [
              {
                "longitude": 24.941325,
                "latitude": 60.173437
              },
              {
                "longitude": 22.264824,
                "latitude": 60.45451
              },
              {
                "longitude": 25.747257,
                "latitude": 62.242603
              },
              {
                "longitude": 25.726967,
                "latitude": 66.503059
              },
              {
                "longitude": 25.469885,
                "latitude":  65.021545
              },
              {
                "longitude": 29.736916,
                "latitude":  62.607662
              },
              {
                "longitude": 27.414838,
                "latitude":  68.419817
              }
            ],
            "dx": 0,
            "dy": 0
          },
          "attributes": {
            "id": "weather_1",
            "class": "Weather"
          }
        }
      ]
    },
    {
      "time_offset": 360,
      "qid": "map_4",
      "attributes": {
        "id": "view4",
        "transform": "translate(600 800)"
      },
      "layers": [
        {
          "qid": "temperature_visualization_4",
          "producer": "gfs",
          "layer_type": "isoband",
          "parameter": "temperature",
          "isobands": "json:isobands/Temperature.json",
          "css": "isobands/Temperature.css"
        },
        {
          "qid": "Temperature-numbers-4",
          "producer": "gfs",
          "layer_type": "number",
          "css": "numbers/numbers.css",
          "parameter": "temperature",
          "label": {
            "dx": 0,
            "dy": 3,
            "precision": 0,
            "missing": "-"
          },
          "positions": {
            "x": 20,
            "y": 18,
            "dx": 30,
            "dy": 30,
            "ddx": 15
          }
        },
        {
          "layer_type": "symbol",
          "positions": {
            "layout": "latlon",
            "locations": [
              {
                "longitude": 24.93417,
                "latitude": 60.17556
              }
            ],
            "dx": 0,
            "dy": 0
          },
          "symbol": "dot",
          "scale": 5
        },
        "ref:refs.Europe_boundaries"
      ]
    }
  ]
}
```
 
</details>

---

With these steps, you now have four maps visualizing different weather parameters (temperature, wind speed/direction, weather symbols, and a time-offset forecast).

