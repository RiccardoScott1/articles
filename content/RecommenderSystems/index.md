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

# Building Recommender Systems

## The Challenge

Netflix knows what you want to watch next. Spotify discovers music you'll love. Amazon suggests products you didn't know you needed. Behind these digital experiences lies one of machine learning's most commercially valuable applications: **recommendation systems**.

Most tutorials focus on single algorithms with toy datasets. Real systems need algorithmic diversity, scalable infrastructure, and the flexibility to experiment with multiple approaches simultaneously.

This series shows you how.

## The Journey: From Algorithms to Architecture

Using Steam's gaming ecosystem as our foundation, we explore a complete recommendation system serving **15+ different algorithms**—from traditional collaborative filtering to modern neural networks. We handle **200,000 users**, **50,000 games**, and **15 million interactions** through unified graph-based architecture.

The journey reveals how algorithm choice, data modeling, and system design interact to create recommendation systems that both perform and scale.

---

## The Architecture Series

### [[1architecture-overview-blueprint|Building a Multi-Modal Recommender System]]
*When one size doesn't fit all*

Architecting systems that serve content-based filtering, collaborative filtering, graph embeddings, and deep learning through unified APIs. Microservices design enables algorithm experimentation without infrastructure rewrites.

### [[2graph-database-design-article|Graph Database Design for Recommender Systems]]
*Design patterns to make your Graph database serve as feature store, model store, and serving layer*

Designing Neo4j schemas where relationships become first-class citizens. Graph-native patterns that support multiple algorithms while maintaining query performance at scale.

### [[3data-import-strategies-article|Data Import Strategies | Neo4j Driver vs. Admin Bulk Import]]
*Transaction-based imports using Neo4j drivers versus bulk CSV imports using Neo4j admin tools*

Benchmarking transaction-based imports against bulk CSV approaches for Neo4j. When to use each pattern and how wrong choices impact development cycles.

---

## The Algorithm Series

### [[4content-based-recommendations-article|Content-Based Recommendations | From Features to Vectors]]
*When features become entities*

Modelling features as graph nodes rather than matrix columns. Users and items coexist in unified feature spaces where similarity becomes graph traversal.

### [[5collaborative-filtering-article|Collaborative Filtering with Graph Algorithms]]
*Leveraging Neo4j's Graph Data Science library to compute real-time user-user and item-item similarities over 15 million interactions—no matrices, no bottlenecks, sub-100 ms query times.*

Using Neo4j's Graph Data Science library for collaborative filtering without memory constraints. Relationship-native algorithms that scale while maintaining interpretability.

### [[6fastrp-universal-embeddings-article|FastRP | Universal Node Embeddings for Multi-Type Recommendations]]
*Cross-type recommendations in unified embedding spaces*

Creating universal embedding spaces where users, games, groups, and friends coexist as neighbors. Cross-type recommendations without complex entity mapping.

### [[7matrix-factorization-approaches|Matrix Factorization Approaches | ALS, BPR, and Beyond]]
*When millions of interactions hide billion-dollar insights*

Implementing ALS, BPR, and hybrid approaches that integrate with graph-derived features. How classical techniques enhance modern neural architectures.

### [[8deep-learning-recommendations|Deep Learning for Recommendations | Two-Tower and Neural CF]]
*When neural networks learn what linear algebra cannot*

Neural Collaborative Filtering and Two-Tower architectures for capturing non-linear interactions. Balancing expressiveness with serving performance through architectural design.

---

## The Production Series

### [[9recommendation-metrics|Recommendation Metrics / Beyond Accuracy]]
*What gets measured gets optimised—choose your metrics wisely*

Implementing ranking metrics, novelty measures, and diversity assessments that capture recommendation quality beyond simple accuracy.

### [[10production-api-design|Production-Ready API Design for Recommendations]]
*From research prototype to production—design APIs that survive contact with reality*

FastAPI services with error handling, monitoring, and flexible algorithm serving. Building APIs that evolve with your recommendation system.

---

## Why This Approach Works

**Technology Stack:**
- **Neo4j & GDS**: Graph database for relationship-native algorithms
- **FastAPI**: High-performance serving with automatic documentation  
- **PyTorch Lightning**: Scalable deep learning with built-in best practices
- **Docker**: Containerised microservices for independent scaling

**Key Features:**
- 15+ algorithms accessible through unified APIs
- Sub-100ms queries across millions of relationships
- Comprehensive evaluation with business-aligned metrics
- Hybrid strategies combining multiple approaches

## Getting Started

**Start with the architecture overview**—then explore algorithms, evaluation, and deployment patterns. Each article builds on previous concepts while standing alone for specific challenges.

The patterns scale from Steam's gaming ecosystem to recommendation systems across industries. Choose your path based on your current needs and system constraints.