---
title: Recommender Systems
tags:
  - recommender-systems
  - machine-learning
  - graph-database
  - neo4j
  - architecture
  - content-based-filtering
  - collaborative-filtering
  - deep-learning
  - embeddings
  - fastrp
  - feature-engineering
  - data-pipeline
  - performance
  - scalability
  - steam
  - production-systems
  - gds
  - personalisation
---


# WIP 
# Recommender Systems

A comprehensive deep dive into building production-scale recommender systems using graph databases, covering architecture design, data engineering, and multiple algorithmic approaches.

The complete implementation, including matrix factorisation variants and graph integration patterns, is available in the Steam Recommender Systems repository. 

todo make public and add link

## Articles in this Series

### [[1architecture-overview-blueprint|Building a Multi-Modal Recommender System]]
*Architecture Overview*

Complete system architecture for serving 15+ different recommendation algorithms through a unified API. Covers microservices design, Docker orchestration, and the decision framework for choosing between content-based filtering, collaborative filtering, FastRP embeddings, and deep learning approaches.

**Key Topics:** System architecture, microservices, algorithm comparison, production deployment

### [[2graph-database-design-article|Graph Database Design for Recommender Systems]]
*Schema Design & Performance*

In-depth exploration of Neo4j schema design patterns that enable multiple recommendation algorithms to coexist. Covers performance optimisation, constraint strategies, and schema evolution patterns for production systems.

**Key Topics:** Database design, schema patterns, performance tuning, graph algorithms

### [[3data-import-strategies-article|Data Import Strategies | Neo4j Driver vs. Admin Bulk Import]]
*Data Engineering & ETL*

Comprehensive comparison of transaction-based imports versus bulk CSV imports for Neo4j. Includes performance benchmarks, memory optimisation strategies, and decision frameworks for different data loading scenarios.

**Key Topics:** Data import, ETL pipelines, performance benchmarking, batch processing

### [[4content-based-recommendations-article|Content-Based Recommendations | From Features to Vectors]]
*Feature Engineering & Vector Similarity*

Advanced feature engineering techniques for content-based filtering using graph structures. Covers sparse vector creation, Neo4j GDS integration, and multiple recommendation strategies with real-world performance metrics.

**Key Topics:** Feature engineering, vector similarity, content filtering, explainable AI

## Core Technologies

- **Neo4j & GDS**: Graph database and data science library for scalable graph algorithms
- **FastAPI**: High-performance API serving layer
- **Docker**: Containerised microservices architecture
- **PostgreSQL**: Raw data storage and ETL source
- **PyTorch**: Deep learning models and two-tower architectures
- **Python**: Data processing with pandas/polars

## System Capabilities

- **Multi-Algorithm Support**: 15+ different recommendation approaches
- **Real-Time Serving**: Sub-100ms recommendation response times
- **Massive Scale**: 50M+ user-game interactions, 200K+ active users
- **Explainable Results**: Clear reasoning for content-based recommendations
- **Production Ready**: Docker orchestration, monitoring, and A/B testing capabilities

## Dataset

All implementations use Steam gaming data featuring:
- 2M+ unique games with rich metadata
- Complex user interaction patterns
- Social relationships and group memberships
- Multi-modal features (genres, developers, player counts)

This series demonstrates how to move from recommendation research to production recommendation systems that scale.