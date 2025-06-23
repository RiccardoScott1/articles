---
title: Geospatial Data Science with PostGIS, Docker, and OpenStreetMap
---

A collection of hands-on, developer-friendly tutorials for working with geospatial data. Whether you're clustering buildings, building a spatial analytics environment, or extracting data from OpenStreetMap — these guides will help you do it efficiently with open tools like **PostGIS**, **GDAL**, **Docker**, and the **Overpass API**.

---

## [[DBSCAN in PostGIS|Clustering Spatial Data with DBSCAN in PostGIS]]
*Cluster Building Polygons Directly in SQL—No Python Required*

Learn how to run spatial clustering on **non-point geometries** (like building footprints) using `ST_ClusterDBSCAN` in PostGIS. This tutorial walks you through DBSCAN clustering entirely within SQL, compares it with scikit-learn’s implementation, and shows how to choose the right distance strategy for your spatial use case.

---

## [[DBSCAN general|Unlocking the Power of Spatial Clustering with DBSCAN]]
*Understand How DBSCAN Works and When to Use It*

Before diving into code, understand the **theory and use cases** behind DBSCAN. This article explains how DBSCAN handles clusters of irregular shape, detects noise, and why it's superior to K-means for spatial analysis. It also compares different implementations and which ones support non-point geometries.

---

## [[Geoformats to PostGIS|Managing and Converting Geospatial Data with PostGIS and ogr2ogr]]
*Convert, Import, and Export Geospatial Formats with GDAL’s `ogr2ogr` Tool*

Need to move geospatial data between formats like **Shapefile**, **GeoJSON**, **KML**, or **GeoPackage**? Learn how to use `ogr2ogr`—a command-line tool from the GDAL suite—to seamlessly **convert, import, and export geospatial data** into a PostGIS database. Ideal for GIS analysts, devs, and data engineers.

---

## Building a Geospatial Data Science Stack with Docker Compose
*PostGIS, GDAL, and JupyterLab—Containerized and Ready to Go*

Tired of installing geospatial libraries manually? This guide shows how to spin up a **full geospatial data science environment** using Docker Compose. It includes containers for PostGIS, GDAL, and JupyterLab, with custom Dockerfiles for reproducibility. Great for ETL pipelines, analysis, and collaborative notebooks.

- Quickstart: [[geospatial stack a quick tutorial]]
- In depth walkthrough [[geospatial stack in depth]]

> Repo: [simple_geosetup on GitHub](https://github.com/RiccardoScott1/simple_geosetup)

---

##  [A Practical Guide to OpenStreetMap’s Overpass API Query Language](https://chatgpt.com/c/overpass-api-guide.md)
*Extract Exactly the Data You Need from OpenStreetMap*

Skip the full planet file—learn how to write efficient, custom Overpass QL queries to fetch **just the OpenStreetMap data you care about**. This guide walks through examples like finding cafés in Paris, bus stops in Berlin, or buildings missing addresses in NYC. Ideal for spatial devs, civic tech, and OSM analysts.

---

## Start Exploring

Whether you're new to geospatial development or looking to scale your workflows with Docker and PostGIS, these articles are designed to be **practical, code-driven, and ready to apply** to your own projects.

→ **Tip**: Start with [DBSCAN in PostGIS](https://chatgpt.com/c/clustering-dbscan-postgis.md) or [Overpass API](https://chatgpt.com/c/overpass-api-guide.md) for quick wins with real-world data.

