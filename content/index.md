---
title: Developer-Focused Data Science Tutorials
tags:
  - DataScience
  - Geospatial
  - RAG
  - PostgreSQL
  - OpenSource
---

Welcome to a collection of hands-on, developer-friendly tutorials covering **geospatial data science**, **retrieval-augmented generation (RAG)**, and **PostgreSQL full-text search**. These guides focus on practical implementations using open-source tools and modern development practices.

---

## [[Geospatial Series/index|Geospatial Data Science]]

*PostGIS, Docker, GDAL, and spatial analysis workflows*

Build robust geospatial data pipelines and perform advanced spatial analysis using industry-standard open-source tools.

### Key Topics:
- **[[Geospatial Series/geospatial stack a quick tutorial|Docker-based Geospatial Stack]]** - Set up PostGIS, GDAL, and Jupyter in minutes
- **[[Geospatial Series/DBSCAN general|DBSCAN Clustering]]** - Density-based spatial clustering for irregular shapes
- **[[Geospatial Series/DBSCAN in PostGIS|PostGIS DBSCAN]]** - Run clustering directly in SQL without Python
- **[[Geospatial Series/OpenStreetMap Data|OpenStreetMap & Overpass API]]** - Extract custom geographic data with precision
- **[[Geospatial Series/Geoformats to PostGIS|Data Conversion with ogr2ogr]]** - Convert between geospatial formats and import to PostGIS

**Perfect for:** GIS analysts, data scientists, and developers working with location-based applications

---

## [[RAG on a Web Domain/index|RAG on a Web Domain]]

*Chat with entire websites using open-source AI tools*

Build a full-stack RAG pipeline that crawls, embeds, and enables conversational interactions with any website's content.

### Key Components:
- **[[RAG on a Web Domain/Quickstart|Quick Start Guide]]** - Complete RAG pipeline overview
- **[[RAG on a Web Domain/Crawl4AI domain crawler|Crawl4AI Implementation]]** - Domain-aware web crawling and content extraction
- **[[RAG on a Web Domain/Setting up N8N and Supabase for a Domain aware RAG App|N8N & Supabase Setup]]** - Workflow automation and vector storage
- **[[RAG on a Web Domain/Ollama on Digital Ocean|Self-hosted LLM Deployment]]** - Run your own models on DigitalOcean

**Perfect for:** Developers building AI-powered knowledge bases, chatbots, or content discovery systems

---

## [[PostgreSQL Full Text Search/index|PostgreSQL Full Text Search]]

*Powerful search without external dependencies*

Master PostgreSQL's built-in full-text search capabilities to implement sophisticated search functionality directly in your database.

### What You'll Learn:
- **[[PostgreSQL Full Text Search/Full Text Search in PosgreSQL|FTS Fundamentals]]** - Complete guide to PostgreSQL search features
- **[[PostgreSQL Full Text Search/Full Text Search in PosgreSQL Tutorial|Hands-on Tutorial]]** - Practical examples with real-world datasets
- Advanced indexing strategies with GIN indexes
- Weighted search across multiple fields
- Result ranking with `ts_rank` and `ts_rank_cd`
- Performance optimization techniques

**Perfect for:** Backend developers, database architects, and teams wanting to avoid external search infrastructure

---

## Why These Tutorials?

**Developer-First Approach:** Every tutorial includes working code, Docker configurations, and real-world examples you can run immediately.

**Open-Source Focus:** No vendor lock-in. All tutorials use free, open-source tools that you can deploy anywhere.

**Production-Ready:** Techniques and patterns that scale from prototypes to production systems.

**Modern Tooling:** Docker, Git workflows, and cloud deployment strategies integrated throughout.

---

## Getting Started

Each tutorial series is self-contained with its own setup instructions. Choose based on your current project needs:

- **Need spatial analysis?** → Start with [[Geospatial Series/geospatial stack a quick tutorial|Geospatial Stack]]
- **Building AI applications?** → Jump to [[RAG on a Web Domain/Quickstart|RAG Quickstart]]
- **Want better search?** → Begin with [[PostgreSQL Full Text Search/Full Text Search in PosgreSQL Tutorial|FTS Tutorial]]

---

*All code examples, Docker configurations, and sample datasets are available in the linked GitHub repositories.*