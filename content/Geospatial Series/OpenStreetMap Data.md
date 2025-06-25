---
title: A Practical Guide to OpenStreetMap's Overpass API Query Language
tags:
  - OpenStreetMap
  - OverpassAPI
  - GIS
  - DataExtraction
  - Geospatial
---
*Mastering Overpass QL: A Hands-On Guide to Extracting Custom OSM Data with Precision and Ease*

The **Overpass API** is a powerful and efficient way to extract targeted geographic data from **OpenStreetMap (OSM)**. Instead of downloading large and complex OSM datasets, the Overpass API lets you query and retrieve **only the specific data they need**, such as buildings, roads, or amenities, based on location or tags.

At its core is the [**Overpass Query Language (Overpass QL)**](https://wiki.openstreetmap.org/wiki/Overpass_API/Overpass_QL) — a flexible scripting language that makes it easy to **filter, search, and process OSM data**. Whether you're looking for **bus stops in London**, **rivers in Germany**, or **cycling infrastructure in Amsterdam**, Overpass QL helps you access precise OSM data for your project.

## Understanding Overpass Query Language (Overpass QL) 
Overpass QL is a **declarative** language that **describes what data to retrieve** rather than how to retrieve it. It is structured around:  
- **Data Types**: Nodes (points), Ways (lines/polygons), and Relations (grouped elements).  
- **Filters**: Selecting elements based on location, tags, and attributes.  
- **Output Options**: Controlling what data is returned and in what format.  
### **Basic Structure of an Overpass Query**  
A typical Overpass QL query consists of:  
1. **A bounding box or area filter** (optional but recommended for efficiency).
2. **A query to select elements** (nodes, ways, or relations).  
3. **Filters** to refine results based on tags or properties.  
4. **An output statement** to format the results.  

## **Basic Overpass QL Queries**  

### **1. Find All Cafés in Paris**  
This query retrieves all cafés (`amenity=cafe`) within the city of Paris:  
```ql
[out:json];
area[name="Paris"]->.searchArea;
node["amenity"="cafe"](area.searchArea);
out body;
```

Paste the query [here](https://overpass-turbo.eu/) and you'll see there is a city called `Paris` in the US and a cafe in Lignano, Italy in a hotel with the tag `name=Paris`.
![[area_paris_results.png]]

If you want to focus on only Paris, France, use the bounding box. To get the bounding box coordinates:
- zoom the map around Paris 
-  run: 
```ql
relation({{bbox}});
out count({{bbox}}); /* only return counts to keep the query light */
```
![[get_bounding_box.png]]

- click export 
- copy standalone query
you get the query above with the actual bounding box coordinates, that you can use in other queries or API calls:
![[export_query.png]]
```ql

relation(48.525700252688765,1.7330932617187502,49.21939110725795,3.0006408691406254);
out count(48.525700252688765,1.7330932617187502,49.21939110725795,3.0006408691406254);
```

The **bounding box** `(48.52,1.73,49.22,3.00)` represents **(South Latitude, West Longitude, North Latitude, East Longitude)**, in other words: bottom-left top-right coordinates.

Now you can filter on the boundary coordinates of Paris, France as well as on the area name. 
```ql
[out:json];
area[name="Paris"]->.searchArea;
node["amenity"="cafe"](area.searchArea)(48.52,1.73,49.22,3.00);
out body;
```

This query only returns results in Paris, France:
![[area_paris_and_bbox_results.png]]

The query above combines both area filters and bounding boxes, but it's important to know how they differ. `area[name="Paris"]` uses OSM's administrative boundaries (like city limits), but as we saw it's prone to data entry errors. `(48.52,1.73,49.22,3.00)` is a coordinate-based bounding box. Overpass will return results that match both conditions — i.e., cafés within the named Paris area and within the bounding box.

### **2. Get All Highways in a Specific Bounding Box**  
This query finds all highways (roads, paths, etc.) within a specific **latitude/longitude bounding box**:  
```ql
[out:json];
way["highway"](48.52,1.73,49.22,3.00);
out geom;
```
- The `["highway"]` filter selects all road-related features.  
- The `out geom;` statement returns full geometries (lat/lon coordinates).

### **3. Find Bus Stops Near Berlin and Export as .csv**
To find **public transport bus stops** around **Berlin** and **output the Overpass QL query as a CSV**, use the `out:csv` statement and specify the fields you want in the output. 

The following query:
```ql
[out:csv(::id, name, ::lat, ::lon)];
area["name"="Berlin"]->.searchArea;
node["highway"="bus_stop"](area.searchArea);
out;
```
- Searches for all **nodes** tagged with `"highway"="bus_stop"` within that area.  
- Defines the **Berlin area** using its name.
- Outputs the data in **CSV format**: `out:csv(::id, name, ::lat, ::lon);`**  
   - `::id` → OSM node ID (unique identifier).  
   - `name` → The name of the bus stop (if available).  
   - `::lat, ::lon` → Latitude and longitude coordinates.  

**The Output for CSV Data**  
```csv
@id,name,@lat,@lon
27239370,Hahneberg,52.5238455,13.1449355
27587012,U Olympia-Stadion,52.5165172,13.2491345
27586412,Güterbahnhof Ruhleben,52.5283572,13.2203842
```

## **Advanced Overpass QL Features**

### **4. Fetching Only Recent Changes (Changed in the Last 7 Days)**  
If you need to monitor recent changes, Overpass can fetch only the **data modified within the last week**:  
```ql
[out:json];
way["highway"](newer:"2024-03-25T00:00:00Z");
out geom;
```

This retrieves **all roads that have been edited** since **March 25, 2024**.  

### **5. Finding Buildings with Missing Addresses**  
To identify buildings **without an address tag** in New York:  
```ql
[out:json];
area[name="New York"]->.searchArea;
way["building"](area.searchArea);
(._; - way["addr:housenumber"](area.searchArea););
out geom;
```

This **Overpass QL** query is designed to find **buildings in New York that are missing an address (`addr:housenumber`)**. Let's break it down **line by line**:  
- **`[out:json];`** Sets the output format to JSON. (Overpass API supports different output formats, e.g., XML, CSV, JSON).
- **`area[name="New York"]->.searchArea;`**: 
  - finds the **administrative boundary** of New York,  `->.searchArea;` **stores the result in a variable** called `.searchArea` for later use.
  - **Why use `area`?** OpenStreetMap stores large administrative boundaries as **relations** instead of nodes or ways. Directly searching within a **bounding box** can be inefficient, so using an **area filter** is often better.  
- **`way["building"](area.searchArea);`**Finds all ways (polygons) tagged as "building" inside the New York area.** A **way** is a collection of **nodes** that form a **line or polygon**. Buildings are usually represented as **closed ways (polygons)**.
- **`(._; - way["addr:housenumber"](area.searchArea););`**Filters out buildings that already have an address (`addr:housenumber`). 
  **Breaking it down:**  
  - `._;` → **Refers to the previous selection** (all buildings in New York).  
  - `way["addr:housenumber"](area.searchArea);` → **Finds buildings that have an address (`addr:housenumber`).**  
  - `-` (minus operator) → **Removes** buildings that **already have an address** from the previous selection.  
  As **end result** the query **keeps only buildings that are missing an address**.

- **`out geom;`**: Outputs the final result, including geometries (latitude/longitude coordinates).
  - `out geom;` -> Returns metadata **with** geometry data (coordinates of buildings).  
  - `out body;` -> Returns only metadata **without** geometry.  

## **Output Formats**  
Overpass API supports different output formats depending on your needs:  
- **JSON (`out:json;`)** – Easy to parse and use in web applications.  
- **XML (`out:xml;`)** – Useful for structured data processing.  
- **CSV (`out:csv(name,lat,lon);`)** – Ideal for spreadsheets.  
- **Map Display (`out meta;`)** – Can be visualised directly in Overpass Turbo. 

For example, to output **only node IDs and coordinates in CSV format**:  
```ql
[out:csv(::id, ::lat, ::lon)];
area[name="London"]->.searchArea;
node["amenity"="restaurant"](area.searchArea);
out;
```

## **Running Overpass Queries**  
### **1. Using Overpass Turbo (Web Interface)**  
The easiest way to test Overpass QL queries is via **[Overpass Turbo](https://overpass-turbo.eu/)**. It provides:  
- A **map-based interface** to run and visualize queries.
- **Auto-formatting** for Overpass QL.
- **Export options** (GeoJSON, KML, GPX).

### **2. Using Overpass API Directly (Command Line or Scripts)**  
For programmatic access, use the Overpass API endpoint:  
```bash
curl -X GET "https://overpass-api.de/api/interpreter" --data-urlencode 'data=[out:json];area[name="Paris"]->.searchArea;node["amenity"="cafe"](area.searchArea);out body;'
```

Or in Python using **requests**:  
```python
import requests

query = """
[out:json];
area[name="Berlin"]->.searchArea;
node["amenity"="bicycle_rental"](area.searchArea);
out body;
"""
url = "https://overpass-api.de/api/interpreter"
response = requests.get(url, params={'data': query})
data = response.json()
```

## **Conclusion**  
The **Overpass API Query Language** is a **versatile and efficient tool** for extracting custom OSM data. Whether you're looking for specific amenities, analysing transportation networks, or tracking changes in geographic data, Overpass QL provides a **powerful way to interact with OpenStreetMap**.  

If you're new to Overpass QL, start with **simple queries in Overpass Turbo**, then explore more advanced filtering techniques as you get comfortable.  

Try running queries on **[Overpass Turbo](https://overpass-turbo.eu/)** and experiment with real-world data!