---
title: Geospatial Data Science with PostGIS, Docker, and OpenStreetMap
tags:
  - geospatial
  - postgis
  - dbscan
  - docker
  - openstreetmap
  - spatial-analysis
  - data-science
  - gis
  - energy-infrastructure
  - etl-pipeline
  - data-engineering
---

A collection of hands-on, developer-friendly tutorials for working with geospatial data. From enterprise ETL pipelines for critical infrastructure to spatial clustering algorithms—these guides cover everything from extracting OpenStreetMap data to building production-ready geospatial systems using **PostGIS**, **GDAL**, **Docker**, and modern data engineering practices.

---

## Energy Infrastructure Intelligence

### [[osm_energy_infra_long|Mapping the Invisible | How OpenStreetMap Reveals Our Power Grid]]
*When millions go dark, the invisible grid becomes visible—here's how crowd-sourced data illuminates the backbone of civilisation*

Discover how to transform OpenStreetMap's vast geographic database into actionable electrical infrastructure intelligence. This comprehensive case study demonstrates enterprise-grade ETL capabilities through a production pipeline that processes millions of power-related features—from transmission lines to renewable generators. Perfect for energy professionals, data engineers, and infrastructure analysts seeking to understand grid topology, assess renewable capacity, and support critical infrastructure planning.

### [[osm_energy_infra_tutorial|Building a Modern ETL Pipeline for Critical Infrastructure Data]]
*From chaos to clarity—how enterprise-grade data engineering transforms scattered geographic fragments into strategic intelligence*

A technical deep-dive into building scalable, high-performance ETL pipelines for geospatial infrastructure data. Learn the architectural decisions, performance optimisations, and domain expertise that enable processing country-scale datasets with sub-second query times. Features containerised microservices, intelligent unit conversion, and automated data quality validation—showcasing modern data engineering practices applied to energy sector challenges.

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

Whether you're building enterprise-grade ETL pipelines for critical infrastructure, clustering spatial data, or setting up geospatial analytics environments, these articles provide **practical, production-ready solutions** you can apply immediately.

**Recommended Starting Points:**
- **For Infrastructure & Energy Projects**: Start with [[osm_energy_infra_long|Mapping the Invisible]] to see enterprise ETL in action
- **For Spatial Analytics**: Try [[DBSCAN in PostGIS]] for immediate clustering results
- **For Development Setup**: Begin with [[geospatial stack a quick tutorial]] to get your environment running
- **For Data Extraction**: Explore [[OpenStreetMap Data]] to master the Overpass API

