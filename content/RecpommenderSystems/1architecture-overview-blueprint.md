---
title: Building a Multi-Modal Recommender System
tags:
  - recommender-systems
  - architecture
  - microservices
  - neo4j
  - graph-database
  - machine-learning
  - content-based-filtering
  - collaborative-filtering
  - deep-learning
  - embeddings
  - fastrp
  - fastapi
  - pytorch
  - docker
  - steam
  - production-systems
  - algorithms
  - data-pipeline
  - schema-design
  - performance
  - scalability
---
*Architecture Overview*

Recommender systems power everything from Netflix to Spotify. But what happens when you need to compare fundamentally different approaches—content-based filtering, collaborative filtering, graph embeddings, and deep learning—all within a single production system?

This is the story of building a multi-modal Steam game recommender that serves 15+ different algorithms through a unified API, processing millions of user-game interactions in a graph database.

## The Core Challenge: One System, Multiple Paradigms

Most recommendation tutorials focus on a single approach. Real production systems need to experiment with many. Our architecture handles this by creating a modular pipeline where each recommendation paradigm can coexist:

```
Raw Steam Data (PostgreSQL)
    ↓ 
 ETL Pipeline (to_graph service)
    ↓
 Neo4j Graph Database
    ↓
 Model Training (model service) 
    ↓
 FastAPI Serving Layer (api service)
    ↓
 REST Recommendations
```

The beauty lies in the separation of concerns. Each service handles one responsibility, yet they compose into a system that can serve content-based recommendations at 9am and deep learning embeddings at 9:01am.

## Service Architecture: Microservices

### The ETL Foundation

The `pg_persistor` service handles raw Steam API ingestion. Nothing fancy—just reliable PostgreSQL storage for millions of user interactions, game metadata, and social connections.

The `to_graph` service transforms this relational data into Neo4j's graph format. The system offers two approaches, each optimised for different scenarios.

**Transaction-Based Import (Live Data)**
```python
def transfer_data_batch(self, query_pg: str, query_neo4j: str, data_type: str):
    for batch in pd.read_sql(query_pg, con=self.pg_connection, chunksize=10_000):
        records = batch.to_dict(orient="records")
        with self.graph.session() as session:
            session.run(query_neo4j, records=records)
```

This approach streams data directly from PostgreSQL to Neo4j in 10K record batches. It's perfect for incremental updates and smaller datasets where you need transaction guarantees.

**Parquet-Based Import (Bulk Data)**  
```python
def write_playtimes():
    df = pl.read_parquet("Games_filtered.parquet")
    for batch in df.iter_slices(n_rows=5_000):
        records = batch.to_dicts()
        with driver.session() as session:
            session.run(playtime_query, records=records)
```

For initial loads of 50M+ records, the system bypasses PostgreSQL entirely. Raw data gets converted to Parquet files, then loaded using Polars for maximum throughput.

**Why Two Approaches?** Transaction-based import handles ongoing updates reliably but caps at ~10K records/second. Parquet import hits 50K+ records/second but requires preprocessing.

### The Model Training Engine
The `model` service is where algorithms come alive. Each recommendation approach gets its own module, but they all inherit from a common `Model` base class:

```python
class Model:
    def _project(self) -> Graph:
        """Create GDS graph projection"""
        return self.gds.graph.project(self.proj.graph_name, 
                                     self.proj.node_projection, 
                                     self.proj.relationship_projection)
    
    def _write_sim_to_db(self, G: Graph):
        """Write similarities back to Neo4j"""
        raise NotImplementedError
        
    def run(self):
        G = self._project()
        self._write_sim_to_db(G)
        self._post_clean()
```

This pattern means adding a new algorithm is as simple as implementing `_write_sim_to_db()`. The projection, cleanup, and orchestration are handled automatically.

### The API Layer: 15+ Algorithms in One Factory
The API service uses a factory pattern to serve any trained model:
```python
class ModelFactory:
    def __init__(self):
        self._model = {}
    
    def register_model(self, model_name: ModelName, model: Type[Recommendation]):
        self._model[model_name] = model
    
    def get_model(self, model_name: str) -> Recommendation:
        return self._model[model_name]()

# Registration happens at startup
model_factory.register_model("apps_content_based_knn", RecGamesContentBasedKNN)
model_factory.register_model("apps_collaborative_weighted", RecGamesCollaborativeWeighted)
model_factory.register_model("apps_fastrp_direct", RecGamesFastRP)
```

Now any HTTP client can switch algorithms with a single parameter:
```bash
curl "localhost/recommendations/games?user_id=123&model=apps_fastrp_direct"
curl "localhost/recommendations/games?user_id=123&model=apps_content_based_knn"
```

## Four Paradigms, One Graph

### Content-Based: Features as Graph Nodes

```
┌─────────┐     ┌──────────┐     ┌─────────┐
│  User   │────▶│   Game   │────▶│ Feature │
│  Alice  │ OWNS│ Cyberpunk│ HAS │   RPG   │
└─────────┘     └──────────┘     └─────────┘
     │                               ▲
     └───────────LIKES───────────────┘
      (inherited from owned games)
```

Most content-based systems use feature matrices. We model features as graph nodes, creating richer representations:

```python
def _prepare_features(self):
    # Games connect to their features (genres, developers)
    self.gds.run_cypher('''
        MATCH (a:App)-[:HAS_GENRE]->(g:Genre)
        MERGE (a)-[:FEATURE]->(g:Feature:Genre)
    ''')
    
    # Users inherit features from owned games
    self.gds.run_cypher('''
        MATCH (u:User)-[:OWNS]->(a:App)-[:FEATURE]->(f:Feature)
        WITH u, f, count(a) as weight
        MERGE (u)-[:LIKES {weight: weight}]->(f)
    ''')
```

Now users and games exist in the same feature space. KNN similarity becomes a graph algorithm rather than a matrix operation.

### Collaborative Filtering: Graph Algorithms at Scale

```
User-Item Bipartite Graph → Item-Item Similarities

┌───────┐    ┌──────┐    ┌───────┐
│ User1 │───▶│Game A│◀───│ User2 │
└───────┘    └──────┘owns└───────┘
     │          ║          │
 owns│       similar       │owns
     ▼          ║          ▼
┌──────┐ ═══════╬═══════ ┌──────┐
│Game B│        ║        │Game C│
└──────┘     Jaccard     └──────┘
              Index
```

Traditional collaborative filtering computes user-user or item-item similarities in memory. Neo4j GDS can handle graphs with billions of relationships:

```python
# Project user-item bipartite graph
G = gds.graph.project(
    "user_item",
    ["User", "App"], 
    {"PLAYED": {"orientation": "UNDIRECTED"}}
)

# Compute item-item similarities using Jaccard
results = gds.nodeSimilarity.write(
    G,
    writeRelationshipType="ITEM_SIMILAR",
    similarityCutoff=0.1,
    topK=20
)
```

The algorithm runs in parallel across the entire graph, writing similarities back as new relationships. No matrices, no memory constraints.

### FastRP: Universal Embeddings Across Entity Types

```
Multi-Entity Graph → Unified Embedding Space

┌──────┐   ┌──────┐   ┌───────┐   ┌────────┐
│ User │   │ Game │   │Friend │   │ Group  │
└──────┘   └──────┘   └───────┘   └────────┘
    │         │          │           │
    └─────────┼──────────┼───────────┘
              ▼FastRP    ▼
    ┌─────────────────────────────────────┐
    │    128-Dimensional Vector Space     │
    │  [0.2, -0.1, 0.5, ..., 0.3, -0.7]   │
    │     Cross-Type Similarities         │
    └─────────────────────────────────────┘
```

>The breakthrough insight: users, games, groups, and friends can all be embedded in the same vector space using Fast Random Projection.

```python
def fastrp(self, G) -> None:
    results = self.gds.fastRP.mutate(
        G,
        embeddingDimension=128,
        randomSeed=42,
        mutateProperty="embedding",
        iterationWeights=[1.0, 1.0, 1.0]  # 3 iterations
    )
```

This creates 128-dimensional embeddings for every node. Now you can recommend:
- Games similar to other games
- Users similar to other users  
- Games similar to users (cross-type recommendations)
- Friends who like similar games

All from the same embedding space.

### Deep Learning: Two-Tower Architecture

```
Two-Tower Neural Architecture

 User Features          Item Features
┌─────────────┐       ┌─────────────┐
│ Age: 25     │       │ Genre: RPG  │
│ Country: US │       │ Price: $60  │
│ Playtime: H │       │ Rating: 9.1 │
└─────────────┘       └─────────────┘
       │                     │
       ▼                     ▼
┌─────────────┐       ┌─────────────┐
│ User Tower  │       │ Item Tower  │
│   Neural    │       │   Neural    │
│   Network   │       │   Network   │
└─────────────┘       └─────────────┘
       │                     │
       ▼                     ▼
┌─────────────┐       ┌─────────────┐
│User Embedding│      │Item Embedding│
│ [64 dims]   │      │ [64 dims]    │
└─────────────┘       └─────────────┘
       │                     │
       └──────────┬──────────┘
                  ▼
            Dot Product
         (Recommendation Score)
```

The PyTorch service handles neural approaches. The two-tower architecture learns separate embeddings for users and items:

```python
class TwoTowerModel(pl.LightningModule):
    def __init__(self, config: ModelConfig):
        super().__init__()
        self.user_tower = FeatureLayer(config.user_features)
        self.item_tower = FeatureLayer(config.item_features)
        
    def forward(self, batch):
        user_emb = self.user_tower(batch['user_features'])
        item_emb = self.item_tower(batch['item_features'])
        return torch.mm(user_emb, item_emb.t())  # Dot product
    
    def training_step(self, batch, batch_idx):
        scores = self(batch)
        loss = F.binary_cross_entropy_with_logits(scores, batch['targets'])
        return loss
```

The model trains on implicit feedback (playtime > 0 = positive, else negative) and can incorporate rich features from the graph database.

## Docker Orchestration: 7 Services, One Command

The entire system runs with `docker-compose up`:

```yaml
services:
  postgres:    # Raw data storage
  neo4j:       # Graph database with GDS plugins
  pg_persistor: # Data ingestion
  to_graph:    # ETL pipeline  
  model:       # Algorithm training
  api:         # FastAPI serving
  pytorch:     # Deep learning experiments
```

Each service is independently scalable. Need more API throughput? Scale the API service. Training large embeddings? Scale the model service.

## When to Use What: Decision Framework

**Content-Based** excels with:
- Cold start users (no interaction history)
- Explainable recommendations ("Because you liked RPGs")
- Rich item metadata (genres, developers, tags)

**Collaborative Filtering** works best with:
- Dense interaction matrices
- Users with established preferences
- Implicit feedback signals (playtime, purchases)

**FastRP Embeddings** shine for:
- Cross-domain recommendations (games → friends → groups)
- Scalable similarity computation
- Multi-entity recommendation spaces

**Deep Learning** handles:
- Complex feature interactions
- Large-scale datasets (millions of users)
- Rich side information (user demographics, item features)

The system lets you A/B test these approaches against real user behaviour, not synthetic benchmarks.

## Real-World Impact

This architecture powers recommendations for a Steam dataset with:
- 50M+ user-game interactions
- 2M+ unique games
- 200K+ active users
- 15+ different algorithms

Response times stay under 100ms because similarities are pre-computed and cached in Neo4j. The graph database becomes both the feature store and the serving layer.

## What's Next?

The modular design makes extending the system straightforward:
- Add new algorithms by implementing the `Model` interface
- Incorporate new data sources through the ETL pipeline
- Scale individual services based on demand
- A/B test recommendation strategies in production

Most importantly, you can compare approaches fairly—same data, same evaluation metrics, same serving infrastructure. This is how you move from recommendation research to recommendation systems.