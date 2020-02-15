# Overhead Imagery Segmentation Dataset Generator (OISDG)

This project was developed to create datasets for semantic segmentation with overhead imagery. 

The satellite or aerial images are obtained from a server supported by [MapProxy](https://mapproxy.org/), as specified in a configuration file, and the segmentation masks as built through OSM data from [OpenStreetMap](https://www.openstreetmap.org). 

Depending on the configuration, a segmentation mask is created for particular types of objects. OSM data can be drawn normally, or a flood-fill algorithm can be used to fill the street segments. Depending on the quality of the image data, the flood-fill can deliver good or bad results.

*When using the data, please take note of the license terms of the data sources.* 

# How To Use It

This section describes how you can use the tool. This example uses the scenario shown above.

## Create The Configuration

The configuration defines how the dataset is supposed to be generated. The structure of the configuration is explained by the following comments.

```java
{
/*
* This part defines the area for which a data set is to be defined.
* The range is indicated by the left upper and right lower corners.
* The fixed points are not necessarily strict. The following figure shows this using an example. 
* The green dots represent the defined corner points and the black rectangle the tiles.
* This means that instead of the defined orange rectangle, the orange highlighted area is included (under certain zoom-levels).
*
* The minimum and maximum zoom level can also be restricted. 
* If this is left at 5 and 19, all possible sizes for the range are automatically created.
*
* 2km 		5
* 1km 		6
* 500m		7
* 250m		8
* 125m		9
* 62.5m		10
* 31.25m	11
* 15.625m	12
*/
    "location_range": {
        "top_left_location": {
            "latitude": 48.779346,
            "longitude": 9.172896
        },
        "bottom_right_location": {
            "latitude": 48.774943,
            "longitude": 9.184387
        },
        "zoom_range": {
            "min": 5,
            "max": 17
        }
    },
/*
* Defines the image size of the tiles and the configuration for the map tile proxy service.
*/
	"map_api": {
		"tile_size": 512,
                "config_path": "config-example1.yaml"
	},
/*
* This part determines timeouts and attempts for the Overpass-Api.
*/
    "overpass_api": {
		"max_tries": 5,
		"connection_establish_timeout": 5,
		"response_timeout": 30 
	},
/*
* Which categories are needed and how they are drawn is defined here.
* For each category a query, the extension radius and the drawing options are specified.
* The current range of the query is defined with "{{bbox}}", so this should be included in every query.
* The expansion factor prevents objects at the edge from being truncated by the query.
* The color of the category and type which can be "area", "line" or "dot" is always specified in the drawing options.
* If a category consists of areas and lines, it must be defined twice with the same color as in the example water and highway.
* Depending on the type, there are additional options.
* - Area: There are no additional options here.
* - Line: The line thickness can also be specified here.
* - Dot: The radius can also be defined here.
* 
* Additionally flood-fill can be added to all types. The specified value determines the threshold.
* An example can be seen in the second category for highway.
*/
	"categories": [
	    {
			"overpass_api_query": "way[area][highway]({{bbox}}); way[area][amenity=parking]({{bbox}});",
			"expand_view": 0.0005,
                        "min_pixel_ratio": 0.0,
                        "max_pixel_ratio": 1.0,
			"draw_options": {
				"type": "area",
				"color": {"b":0, "g":255, "r":0}
			}
	    },
	    {
			"overpass_api_query": "way[!area][highway]({{bbox}}); way[!area][amenity=parking]({{bbox}});",
			"expand_view": 0.0005,
                        "min_pixel_ratio": 0.0,
                        "max_pixel_ratio": 1.0,
			"draw_options": {
				"type": "line",
				"color": {"b":0, "g":255, "r":0},
				"line_width": 3,
				"flood_fill": 20
			}
	    },
	    {
			"overpass_api_query": "way[area][natural=water]({{bbox}}); way[area][natural=dam]({{bbox}});",
			"expand_view": 0.0005,
                        "min_pixel_ratio": 0.0,
                        "max_pixel_ratio": 1.0,
			"draw_options": {
				"type": "area",
				"color": {"b":255, "g":0, "r":0}
			}
	    },
	    {
			"overpass_api_query": "way[waterway=river]({{bbox}});",
			"expand_view": 0.0005,
                        "min_pixel_ratio": 0.0,
                        "max_pixel_ratio": 1.0,
			"draw_options": {
				"type": "line",
				"color": {"b":255, "g":0, "r":0},
				"line_width": 3
			}
	    },
	    {
			"overpass_api_query": "way[railway]({{bbox}});",
			"expand_view": 0.0005,
                        "min_pixel_ratio": 0.0,
                        "max_pixel_ratio": 1.0,
			"draw_options": {
				"type": "line",
				"color": {"b":0, "g":0, "r":255},
				"line_width": 3
			}
	    },
	    {
			"overpass_api_query": "node[natural=tree]({{bbox}});",
			"expand_view": 0.0005,
                        "min_pixel_ratio": 0.0,
                        "max_pixel_ratio": 1.0,
			"draw_options": {
				"type": "dot",
				"color": {"b":0, "g":0, "r":0},
				"circuit_radius": 3
			}
	    },
	    {
			"overpass_api_query": "way[building][start_date](if:date(t[start_date])>1900 && date(t[start_date])<2020)({{bbox}});",
			"expand_view": 0.0005,
                        "min_pixel_ratio": 0.0,
                        "max_pixel_ratio": 1.0,
			"draw_options": {
				"type": "area",
				"color": {"b":255, "g":255, "r":255}
			}
	    }
	]
}
```

## Run The OISDG

After the configuration has been created, the tool can be started with the following command, where `--output_path` defines the location to store the dataset and `--config_path` the path to the configuration created above. 

```
python3 oi_segment_dataset_generator.py --output_path ./dataset --config_path ./config.cfg
```
