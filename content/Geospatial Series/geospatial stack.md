# **Building a Geospatial Data Science Stack with Docker Compose**
*PostGIS, GDAL, and Jupyter—All in One Click*

In the world of geospatial data science, setting up your environment can often be as challenging as the analysis itself. Whether you're working with spatial databases, transforming raster and vector formats, or exploring geographic data in a notebook—your toolkit typically includes **PostGIS**, **GDAL**, and a **Jupyter** environment. But installing and configuring all of these tools locally can be a time-consuming headache, especially when dealing with different versions, system dependencies, or team collaboration.

Get started by pulling or cloning the [repo](https://github.com/RiccardoScott1/simple_geosetup)

**Enter Docker Compose.**
In this post, we’ll walk through how to set up a **modular, reproducible, and fully containerised geospatial development environment** using Docker Compose. Our stack includes:
- **PostGIS** – A spatially enabled PostgreSQL database for storing and querying geospatial data.
- **A GDAL-powered client** – For working with formats like GeoTIFF, Shapefile, GPKG, and more.
- **JupyterLab** – A web-based environment to run Python (or R) notebooks with access to PostGIS and GDAL.

By the end of the tutorial, you'll have a running environment where you can:
- Upload and query spatial data in PostGIS
- Use `ogr2ogr`, `gdalwarp`, or `gdal_translate` from your client container
- Explore and analyse data interactively in Jupyter, with libraries like `geopandas`, `psycopg2`, and `folium`

Whether you're a GIS analyst, a data scientist, or a developer prototyping location-based apps, this stack gives you a clean, isolated environment that’s easy to share and scale. 

Let's dive in and start building your geospatial toolkit—**no more dependency hell required**.

## Get it up and running
### Prerequisites
1. **Docker installed**: Ensure Docker is installed on your system. Docker is available for Windows, macOS, and Linux. Installation instructions can be found on the [Docker website](https://docs.docker.com/compose/install/).
2. **.env file**
You can create an `.env` file at the root of the repo to set the following environment variables
```bash
# postgres credentials and DB name to use
POSTGRES_USER=pg
POSTGRES_PASSWORD=pg
POSTGRES_DB=pg
# the token for jupyter and jupyter port number on your host machine
JUPYTER_TOKEN=mytoken
JUPYTER_PORT=8889
```  
### run the services
For convienience we added a `makefile` to the repository so you can (re)start all services and get the link to the Jupyter server with one command:
```bash
make all
```
or just run
```bash
docker compose up -d
```
if you only want a PostGIS and the Jupyter server run:
```bash
docker compose up -d geo_notebook
```

Using the env variables `JUPYTER_TOKEN=mytoken` and `JUPYTER_PORT=8889` the Jupyter server should now be accessible at: `http://127.0.0.1:8889/?token=mytoken`.

## Let's get some data and look at it
We'll get OpenStreetMap (OSM) data, load it to our PostGIS database with our `client` service, and look at the data in Jupyter with our `geo_notebook` service.

There are many ways and formats to download OSM data, here we'll use the OSM style `.json`, convert it to `.geojson` with the [osmtogeojson](https://github.com/tyrasd/osmtogeojson) tool (it's already installed in the `client`!).

The following commands are all run within the `client` container, so we'll run
```bash
docker compose run client bash
```
to get into the command line of the `client`.

Next, we get data from OpenStreetMap Overpass API via cURL:
```bash
curl -X GET "https://overpass-api.de/api/interpreter" \
--data-urlencode 'data=[out:json];area[name="Paris"]->.searchArea;node["amenity"="cafe"](area.searchArea)(48.52,1.73,49.22,3.00);out body;' \
-o data/raw/paris_cafes.json
```

This will download an OSM style `.json` file which we will convert to `.geojson` with the command-line tool `osmtogeojson`:
```bash
docker compose exec client osmtogeojson data/raw/paris_cafes.json > data/raw/paris_cafes.geojson
```
Now we have a file format that `ogr2ogr` has a driver for, so we can push our data to PostGIS:
```bash
ogr2ogr -f "PostgreSQL" \
PG:"host=$PGHOST user=$POSTGRES_USER dbname=$POSTGRES_DB password=$POSTGRES_PASSWORD" \
"data/raw/paris_cafes.geojson" \
-nln "paris_cafes_osm2gj" \
-overwrite \
-lco GEOMETRY_NAME=geom \
-lco FID=gid \
-lco PRECISION=NO \
-progress \
-skipfailures
```
>**Note**: There is a way to this without the extra dependency of `osmtogeojson`, by downloading the OSM `.osm` file. `ogr2ogr` has a driver to work with this format. However, we would have run one upload command per geometry type (points, lines, multilinestrings, multipolygons and other_relations) and the data would not be in ideal format. For example tags would get placed into one text column: `"addr:city"=>"Paris","addr:housenumber"=>"93",..., "opening_hours"=>"Mo-Su 08:00-02:00","wikidata"=>"Q17305044"`, requiring extra processing for being queried.*
### Query and Plot in a Notebook
Now we can open a notebook, query and plot the data:
http://127.0.0.1:${JUPYTER_PORT}/?token=${JUPYTER_TOKEN}
http://127.0.0.1:8889/?token=mytoken


## **Conclusion**
With just a few commands, you've built a self-contained geospatial data science environment using Docker Compose. This setup brings together **PostGIS**, **GDAL**, and **JupyterLab** in a clean, reproducible way—no more manual installs or conflicting dependencies.

Whether you're prototyping spatial workflows, sharing a setup with your team, or just exploring geodata more efficiently, this stack offers a simple, portable foundation that works out of the box. Clone the repo, run the services, and start analyzing—your geospatial toolkit is ready.