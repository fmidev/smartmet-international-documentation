
# Step-by-Step Guide to Creating a WMS Layer

In this guide, you will learn how to create a Web Map Service (WMS) layer. The WMS layer provides geographic information that can be used for various applications such as weather maps.

---

## Step 1: Understand the Structure

This is the minimum required structure for working layer.

```
WMS
├── customers
│   └── test
│       └── products
│           └── model
│               └── temperature.json
│
├── legends
│   └── templates
│       ├── isoband.json
│       ├── isoline.json
│       └── symbol.json
│
│
└── resources
    └── layers
        └── isobands
            ├── temperature.css
            └── temperature.json


```
------------------------

### `customers/`

The `customers` directory is where specific customer data and configurations are stored. Each customer gets its own directory for organizing custom WMS products.

#### `test/`

This is a single customer.

#### `products/`
Contains folders related to the products and models for the customer.

#### `model/`
Stores the weather models specific to this customer. It contains the configuration files that define how the models behave.

------------------------
    
### legends/

#### `templates/`

Contains files for different legends that explains the symbols, colors, and scales on a map. It helps users interpret features like temperature ranges, wind speed, or elevation, ensuring accurate understanding of the map's data.

------------------------
    
### resources/

The resources directory contains reusable assets such as layers, symbols, and other configurations that can be applied across different customers or products.

#### `layers/`

Different types of layers are listed here.


#### `isobands/`
This folder contains specific configurations and styles for the isoband layers used in visualizations.

------------------------

## Step 2: Create the Layer Definition

1. **Title and Abstract**:
   Open the temperature.json file under customer and start by defining the title and abstract of your WMS layer. These will help identify and describe the purpose of the layer.
   
   ```json
       {
         "title": "Temperature",
         "abstract": "Temperature 2m from default model producer"
       }
   ```

2. **Projection**:
   Specify the projection that your map will use. For now we leave it empty.

   ```json
       {
         "title": "Temperature",
         "abstract": "Temperature 2m from default model producer"
         "projection": {}
       }
   ```

3. **Define Views and Layers**:
   The `views` section holds your layer configurations. In this case, you will define a single layer visualizing temperature.

   Add the following to your layer definition:
   ```json
        {
          "title" : "Temperature",
          "abstract" : "Temperature 2m from default model producer",
          "projection": {},
          "views": [{
            "layers": [{
              "qid": "l",
              "layer_type": "isoband",
              "isobands": "json:isobands/temperature.json",
              "css": "isobands/temperature.css",
              "parameter": "Temperature",
              "attributes": {
                "shape-rendering": "crispEdges"
              }
            }]
          }]
        }
   ```
      - **qid**: Unique identifier for the layer.
      - **layer_type**: Type of the layer, in this case, it is "isoband" which represents contour lines.
      - **isobands**: Path to the JSON file that defines the isobands for visualization.
      - **css**: Path to the CSS file for styling the isobands.
      - **parameter**: Defines the weather data parameter, here it is "Temperature"
      - **attributes**: Additional attributes, like `"shape-rendering": "crispEdges"` to improve the rendering.



---

## Step 3: Set Up Isobands

You need to define the temperature ranges for isobands. This is done in the **temperature.json** file under resources. These values can be adjusted according to needs, in the example the temperature color is changed every two degrees

Example **temperature.json**:
<details>
 <summary>Expand</summary>
 
```json
[
	{
		"qid": "temperature_14_inf",
		"lolimit": 14,
		"attributes": {
			"class": "Temperature_14_inf"
		}
	},
	{
		"qid": "temperature_12_14",
		"lolimit": 12,
		"hilimit": 14,
		"attributes": {
			"class": "Temperature_12_14"
		}
	},
	{
		"qid": "temperature_10_12",
		"lolimit": 10,
		"hilimit": 12,
		"attributes": {
			"class": "Temperature_10_12"
		}
	},
	{
		"qid": "temperature_8_10",
		"lolimit": 8,
		"hilimit": 10,
		"attributes": {
			"class": "Temperature_8_10"
		}
	},
	{
		"qid": "temperature_6_8",
		"lolimit": 6,
		"hilimit": 8,
		"attributes": {
			"class": "Temperature_6_8"
		}
	},
	{
		"qid": "temperature_4_6",
		"lolimit": 4,
		"hilimit": 6,
		"attributes": {
			"class": "Temperature_4_6"
		}
	},
	{
		"qid": "temperature_2_4",
		"lolimit": 2,
		"hilimit": 4,
		"attributes": {
			"class": "Temperature_2_4"
		}
	},
	{
		"qid": "temperature_1_2",
		"lolimit": 1,
		"hilimit": 2,
		"attributes": {
			"class": "Temperature_1_2"
		}
	},
	{
		"qid": "temperature_0_1",
		"lolimit": 0,
		"hilimit": 1,
		"attributes": {
			"class": "Temperature_0_1"
		}
	},
	{
		"qid": "temperature_1_0",
		"lolimit": -1,
		"hilimit": 0,
		"attributes": {
			"class": "Temperature_1_0"
		}
	},
	{
		"qid": "temperature_2_1",
		"lolimit": -2,
		"hilimit": -1,
		"attributes": {
			"class": "Temperature_2_1"
		}
	},
	{
		"qid": "temperature_3_2",
		"lolimit": -3,
		"hilimit": -2,
		"attributes": {
			"class": "Temperature_3_2"
		}
	},
	{
		"qid": "temperature_4_3",
		"lolimit": -4,
		"hilimit": -3,
		"attributes": {
			"class": "Temperature_4_3"
		}
	},
	{
		"qid": "temperature_5_4",
		"lolimit": -5,
		"hilimit": -4,
		"attributes": {
			"class": "Temperature_5_4"
		}
	},
	{
		"qid": "temperature_6_5",
		"lolimit": -6,
		"hilimit": -5,
		"attributes": {
			"class": "Temperature_6_5"
		}
	},
	{
		"qid": "temperature_8_6",
		"lolimit": -8,
		"hilimit": -6,
		"attributes": {
			"class": "Temperature_8_6"
		}
	},
	{
		"qid": "temperature_10_8",
		"lolimit": -10,
		"hilimit": -8,
		"attributes": {
			"class": "Temperature_10_8"
		}
	},
	{
		"qid": "temperature_12_10",
		"lolimit": -12,
		"hilimit": -10,
		"attributes": {
			"class": "Temperature_12_10"
		}
	},
	{
		"qid": "temperature_14_12",
		"lolimit": -14,
		"hilimit": -12,
		"attributes": {
			"class": "Temperature_14_12"
		}
	},
	{
		"qid": "temperature_inf_14",
		"hilimit": -14,
		"attributes": {
			"class": "Temperature_inf_14"
		}
	}
]
```
</details>

- **qid**: Unique descriptive id.
- **lolimit**: The lowest temperature where this class will apply.
- **hilimit**: The highest temperature where this class will apply.
- **class**: The CSS class applied.
 

---

## Step 4: Style the Layer with CSS

The **temperature.css** file is responsible for styling the isobands. 

Example **temperature.css**:

<details>
 <summary>Expand</summary>
 
```css
    .Temperature_inf_14	{ stroke:none; fill: rgb(154,8,117) }
    .Temperature_14_12	{ stroke:none; fill: rgb(17,141,250) }
    .Temperature_12_10	{ stroke:none; fill: rgb(64,169,255) }
    .Temperature_10_8	{ stroke:none; fill: rgb(101,189,247) }
    .Temperature_8_6	{ stroke:none; fill: rgb(0,103,140) }
    .Temperature_6_5	{ stroke:none; fill: rgb(2,123,163) }
    .Temperature_5_4	{ stroke:none; fill: rgb(0,155,186) }
    .Temperature_4_3	{ stroke:none; fill: rgb(34,188,212) }
    .Temperature_3_2	{ stroke:none; fill: rgb(103,219,230) }
    .Temperature_2_1	{ stroke:none; fill: rgb(163,243,247) }
    .Temperature_1_0	{ stroke:none; fill: rgb(212,255,255) }
    .Temperature_0_1	{ stroke:none; fill: rgb(5,179,138) }
    .Temperature_1_2	{ stroke:none; fill: rgb(2,212,149) }
    .Temperature_2_4	{ stroke:none; fill: rgb(138,237,187) }
    .Temperature_4_6	{ stroke:none; fill: rgb(204,255,208) }
    .Temperature_6_8	{ stroke:none; fill: rgb(235,252,207) }
    .Temperature_8_10	{ stroke:none; fill: rgb(235,255,122) }
    .Temperature_10_12	{ stroke:none; fill: rgb(255,234,128) }
    .Temperature_12_14	{ stroke:none; fill: rgb(247,212,35) }
    .Temperature_14_inf	{ stroke:none; fill: rgb(199, 106, 0) }

```
</details>

You can customize this to match your design preferences.

---

## Step 5: Legend

A legend explains the colors and symbols on a map, making data like isobands (e.g., temperature ranges) easily understandable. Adding a legend helps users quickly interpret map information at a glance.

SmartMet Server can generate legends using template files. Place these templates in the ```legends/templates``` directory.

<details>
 <summary>isoband.json</summary>
 
```json
    {
  "defs": {
    "layers": [
      {
        "tag": "isoband",
        "attributes": {
          "id": "isoband_symbol_id"
        },
        "layers": [
          {
            "tag": "rect",
            "attributes": {
              "width": "14",
              "height": "9"
            }
          }
        ]
      }
    ]
  },
  "layer_type": "legend",
  "x": 99999,
  "y": 99999,
  "dx": 0,
  "dy": 9,
  "isobands": "isoband_color_scale",
  "symbols": {
    "css": "isoband_css",
    "symbol": "isoband_symbol_id",
    "attributes": {
      "stroke": "black",
      "stroke-width": "0.5"
    }
  },
  "labels": {
    "separator": "-",
    "dx": 18,
    "dy": 9
  },
  "layers": [
    {
      "tag": "text",
      "cdata": "isoband_header",
      "attributes": {
        "x": 99999,
        "y": 99999,
        "class": "Units"
      }
    },
    {
      "tag": "text",
      "cdata": "isoband_unit",
      "attributes": {
        "x": 99999,
        "y": 99999,
        "class": "Units"
      }
    }
  ]
}


```
</details>

<details>
 <summary>isoline.json</summary>
 
```json
    {
  "defs": {
    "layers": [
      {
        "tag": "symbol",
        "attributes": {
          "id": "isoline_symbol_id"
        },
        "layers": [
          {
            "tag": "path",
            "css": "isoline_css",
            "attributes": {
              "d": "M10 10 L20 10 Z",
              "class": "isoline_class"
            }
          }
        ]
      }
    ]
  },
  "layer_type": "legend",
  "x": 99999,
  "y": 99999,
  "dx": 0,
  "dy": 0,
  "symbols": {
    "symbol": ""
  },
  "layers": [
    {
      "layer_type": "symbol",
      "symbol": "isoline_symbol_id",
      "positions": {
        "x": 99999,
        "y": 99999,
        "dx": 99999,
        "dy": 99999
      }
    },
    {
      "tag": "text",
      "cdata": "isoline_header",
      "attributes": {
        "x": 99999,
        "y": 99999
      }
    }
  ]
}


```
</details>

<details>
 <summary>symbol.json</summary>
 
```json
    {
  "layer_type": "legend",
  "x": 99999,
  "y": 99999,
  "dx": 0,
  "dy": 0,
  "symbols": {
    "symbol": ""
  },
  "layers": [
    {
      "tag": "text",
      "cdata": "symbol_group_header",
      "attributes": {
        "x": 99999,
        "y": 99999
      }
    },
    {
      "layer_type": "symbol",
      "symbol": "symbol_file_name",
      "positions": {
        "x": 99999,
        "y": 99999,
        "dx": 99999,
        "dy": 99999
      }
    },
    {
      "tag": "text",
      "cdata": "symbol_text",
      "attributes": {
        "x": 99999,
        "y": 99999
      }
    }
  ]
}


```
</details>


Since various types of layers are in use, the server generates legends for each layer type based on the template files. However, the symbol legend may not be particularly useful in this case. To improve clarity, in real project we could override it by creating a custom legend rather than relying on the server-generated one.



---

## Step 6: Test and Verify

Open the weathermap in your browser to check if everything is functioning correctly. In the top-right corner, locate the layer selector, and find the layer you just added. Selecting this layer should display it on the map.

---

## Step 7: More Layers

To make the WMS product more informative and visually rich, we can add additional layers such as borders, cities, and temperature numbers. These layers help provide geographical context and specific data points, enhancing the overall utility of the map.

#### 1. Adding Borders
Borders help in visualizing country or region boundaries on the map.

Add map layer to your temperature.json views list.

```json
    {
      "qid": "borders",
      "layer_type": "map",
      "css": "maps/map.css",
      "map": {
        "lines": true,
        "schema": "esri",
        "table": "europe_country_wgs84",
        "minarea": 15
      },
      "attributes": {
        "class": "Border"
      }
    }
```
In this configuration:

* layer_type: map indicates the type of layer to display borders.
* css: "maps/map.css" specifies that this layer requires an external CSS file to define its appearance.
* The class: "Border" attribute in the code applies specific styling to the borders.

Since the styling is not embedded directly in the code, the map will not display properly unless the referenced map.css file is included.

For now we only want to add style for border. Add following CSS code to  ```resources/maps/map.css```

```css
    .Border	{ fill: none; stroke: black; stroke-width: 0.8 }
```
Repeat step 5 to make sure everything works.

#### 2. Adding cities

Adding cities helps users locate specific urban areas. You can mark cities on the map using symbols.

Add symbol layer to your temperature.json views list.

```json

    {
      "layer_type": "symbol",
      "positions": {
        "layout": "latlon",
        "locations": [
          {
            "longitude": 24.93417,
            "latitude": 60.17556
          },
          {
            "longitude": 23.7608,
            "latitude": 61.4981
          }
        ],
        "dx": 0,
        "dy": 0
      },
      "symbol": "dot",
      "scale": 6
    }
    
```
Again we need a resource called "dot". 

Add following code to ```resources/symbols/map```

```css
    <symbol id="dot">
      <desc>Simple round marker</desc>
      <circle cx="0" cy="0" r="1"/>
    </symbol>

```
Repeat step 5 to make sure everything works.

#### 3. Adding temperature numbers



Add number layer to your temperature.json views list.

```json
    {
      "qid": "l2",
      "layer_type": "number",
      "parameter": "temperature",
      "label": {
        "missing": "-",
        "dx": 0,
        "dy": 8
      },
      "mindistance": 50,
      "positions": {
            "x": 20,
            "y": 18,
            "dx": 30,
            "dy": 30,
            "ddx": 15
          }
    }
```
Repeat step 5 to make sure everything works.


<details>
 <summary>Show image of final product</summary>
  
 <img src="https://raw.githubusercontent.com/fmidev/smartmet-international-documentation/refs/heads/main/docs/images/wms_1.png" alt="wms end product" style="height: 75%; width:75%;"/>

</details>

<details>
 <summary>Show full temperature.json code</summary>
  

 ```json
{
  "title" : "Temperature",
  "abstract" : "Temperature 2m from default model producer",
  "projection": {},
  "views": [{
    "layers": [{
      "qid": "l",
      "layer_type": "isoband",
      "isobands": "json:isobands/temperature.json",
      "css": "isobands/temperature.css",
      "parameter": "Temperature",
      "attributes": {
        "shape-rendering": "crispEdges"
      }
    },
    {
      "qid": "borders",
      "layer_type": "map",
      "css": "maps/map.css",
      "map": {
        "lines": true,
        "schema": "esri",
        "table": "europe_country_wgs84",
        "minarea": 15
      },
      "attributes": {
        "class": "Border"
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
          },
          {
            "longitude": 23.7608,
            "latitude": 61.4981
          }
        ],
        "dx": 0,
        "dy": 0
      },
      "symbol": "dot",
      "scale": 6
    },
    {
      "qid": "l2",
      "layer_type": "number",
      "parameter": "temperature",
      "label": {
        "missing": "-",
        "dx": 0,
        "dy": 8
      },
      "mindistance": 50,
      "positions": {
            "x": 20,
            "y": 18,
            "dx": 30,
            "dy": 30,
            "ddx": 15
          }
    }]
  }]
}

```
 
</details>



##  Conclusion
By following these steps, you have successfully created a working WMS layer. You can try playing with values or expand on this by adding more layers or different weather parameters.