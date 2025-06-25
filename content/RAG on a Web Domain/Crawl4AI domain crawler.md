---
title: "From Crawl to Context: Building a RAG-Ready Dataset with Crawl4AI"
---

*A hands-on guide to turning any website into structured, vectorised knowledge—ready for your AI to reason over.*
![[crawl4ai_logo.png]]

Here we present a simple setup to perform a breadth-first crawl of a given web-domain, 
storing per page (meta)data such as internal and external links etc and semantically split chunks with embeddings for consumption of a RAG applications.

The code can be found [here](https://github.com/RiccardoScott1/talktomycrawl).

## Crawl4AI
[**Crawl4AI**](https://docs.crawl4ai.com/) is a lightweight, open-source web crawling framework designed specifically for AI and machine learning use cases. It simplifies the process of extracting structured content from websites, making it easy to gather high-quality text data for tasks like training models
or building RAG pipelines. 

With built-in support for filtering, rate limiting, and customisable parsing logic, Crawl4AI is ideal for developers looking to integrate clean, domain-specific data into LLM workflows.


## Implementation
Next we will walk through the prerequisites and code to run our web-domain crawler.

### Supabase
We will be storing the crawled data and embeddings to a remote [**Supabase**](https://supabase.com/) database. Supabase is a fully managed, open-source backend-as-a-service database built on top of PostgreSQL. 

To get started you'll need to:
- [Sign up](https://supabase.com/) and start a new project
- Create the `documents` and `crawled_pages` tables for the crawled data, see [[Setting up N8N and Supabase for a Domain aware RAG App]] for more details.

### Docker
We will be embedding our text chunks from web pages with a local Ollama model running on **Docker**.

To install Docker, visit [https://docs.docker.com/get-docker](https://docs.docker.com/get-docker)  and download the appropriate version for your operating system.  After installation, verify it’s working by running `docker --version` in your terminal.


### Code for crawling and embedding
Next we will set up the code for crawling all the pages on a given domain, cleaning, chunking and embedding the content and storing page data and embeddings to Supabase.

The repo is structured in the following:
```
├── requirements.txt
├── .env
├── src
    ├── embed.py
    ├── main.py
    └── sb.py
```

#### **`.env`**
We define the project's environment variables here.
For `SUPABASE_URL` and `SUPABASE_KEY` go to your Supabase project
`Project Settings`>`Data API`.

```bash
SUPABASE_URL="<YOUR_SUPABASE_PROJECT_URL>"
SUPABASE_KEY="<anon public key>"
SUPABASE_TABLE_NAME_PAGES=crawled_pages
SUPABASE_TABLE_NAME_DOCUMENTS=documents
```


#### **`main.py`**
The `main.py` script orchestrates the full web crawling and document embedding
pipeline using `crawl4ai`, Supabase, and a local embedding model. 
It asynchronously crawls a target website (e.g., `https://nexergroup.com`) using 
a configurable browser and deep crawl strategy, extracts and cleans HTML 
content, and processes the results.

Successful crawls are stored in a Supabase table `crawled_pages` and passed 
through a document embedding function for vectorisation (see `embed.py` later).
The setup enables automated content ingestion, transformation, and storage for
RAG applications.


```python
import asyncio
from crawl4ai import AsyncWebCrawler
from crawl4ai.async_configs import BrowserConfig, CrawlerRunConfig, CacheMode
from crawl4ai.deep_crawling import BFSDeepCrawlStrategy
from sb import get_client  # helper to get the Supabase client
from embed import embed_documents  # Function to embed crawled documents (see later)
from supabase import PostgrestAPIError
import os

async def main():
    url = "https://nexergroup.com"  # Target website for crawling

    # Configuration for the browser used in crawling
    browser_cfg = BrowserConfig(
        text_mode=True,  # Extract only visible text (no images/media)
    )

    # Configuration for how the crawler should run
    run_cfg = CrawlerRunConfig(
        excluded_tags=["script", "style", "form", "header", "footer", "nav"],  # Remove unwanted HTML tags
        excluded_selector="#nexer-navbar",  # Skip specific page element by CSS selector
        only_text=True,  # Extract just the text
        remove_forms=True,  # Skip form elements
        exclude_social_media_links=True,  # Don't follow social links 
        exclude_external_links=True,  # Stay within the main domain
        remove_overlay_elements=True,  # Clean overlays/popups
        magic=True,  # Let crawler auto-tune settings if needed
        simulate_user=True,  # Behave like a real user (e.g., scrolling, clicking)
        override_navigator=True,  # Mask headless browser properties
        verbose=True,  # Output crawl logs
        cache_mode=CacheMode.DISABLED,  # Disable caching of visited pages
        stream=True,  # Stream results as they're found

        # Set up depth-limited crawling strategy (BFS = breadth-first search)
        deep_crawl_strategy=BFSDeepCrawlStrategy(
            max_depth=2,  # Crawl up to 2 levels deep from the starting page
            include_external=False,  # Stay within the same domain
            # max_pages=10  # Optional: limit number of pages, good for debugging
        ),
    )

    # Initialize the asynchronous crawler with Playwright
    async with AsyncWebCrawler(
        config=browser_cfg,
        verbose=True,
        debug=True,
        use_playwright=True,  # Use Playwright for browser automation
    ) as crawler:

        # Crawl the site using provided run configuration
        async for result in await crawler.arun(
            url=url,
            config=run_cfg
        ):
            process_result(result)  # handles the crawl output (one result = one page)


...


# Entry point: runs the main crawler function in an asyncio event loop
if __name__ == "__main__":
    asyncio.run(main())
```

The function for processing the results from the crawler
- connects to supabase
- writes one row per page into the `crawled_pages` table
- calls `embed_documents` that chunks and embeds the text and writes to `documents` table , see later

```python
def process_result(result):
    """
    Process the result returned from the crawler
    """
    if result.success:
        # Convert result object into a dictionary
        result_json = result_dict(result)

        # Initialize Supabase client
        sb_client = get_client()

        try:
            # Insert the crawled data into Supabase
            table_name = os.getenv("SUPABASE_TABLE_NAME_PAGES", "crawled_pages")

            sb_client.table(table_name).insert(result_json).execute()
        except PostgrestAPIError as e:
            print(f"Error inserting into Supabase: {e}")
        
        try:
            # Generate embeddings for the document and store them using the Supabase client
            embed_documents(result_json, sb_client)
        except Exception as e:
            print(f"Error embedding documents: {e}")

        print("Data inserted and embedded successfully.")
    
    else:
        # Log any crawl failure along with the error message
        print(f"Crawl failed: {result.error_message}")
```

A helper to make a dict from the crawler result. The keys correspond to the columns we created in the Supabase table `crawled_pages`. 

```python
def result_dict(result) -> dict:
    """
    convert the result object into a dictionary
    """
    return {
        "url": result.url,
        "links": result.links,
        "metadata": result.metadata,
        "markdown": result.markdown,
        "html": result.html,
        "cleaned_html": result.cleaned_html,
    }
```

We will now look into the two modules called by the main.

#### **`sb.py`**
The `sb` module is defined as a helper to create the Supabase client:
```python 
import os
from supabase import create_client, Client

def get_client()-> Client:
    """
    This function creates a Supabase client using the URL and key from environment variables.
    """
    url: str = os.environ.get("SUPABASE_URL")
    key: str = os.environ.get("SUPABASE_KEY")
    return create_client(url, key)
```


#### **`embed.py`**
The `embed_documents` function in `embed.py`, processes and stores crawled 
web content into our Supabase vector database: 
- cleaned HTML produced by Crawl4AI 
- splits it into semantically meaningful chunks using HTML headers
- embeds each chunk using the `nomic-embed-text` model via a locally running Ollama instance.

These embeddings, along with associated metadata, are stored in the Supabase
`documents` table using LangChain’s `SupabaseVectorStore`. 
This setup enables efficient semantic search and retrieval, 
which is crucial for building RAG applications.


```python
from langchain_community.vectorstores import SupabaseVectorStore
from langchain_text_splitters import HTMLSemanticPreservingSplitter  # Preserves HTML structure while splitting
from langchain_ollama import OllamaEmbeddings  # Interface for embedding with Ollama models
from langchain.docstore.document import Document  # Document object used by LangChain
from supabase import Client

def embed_documents(result:dict, supabase_client:Client):
    """
    Splits a crawled HTML document into semantic chunks, generates embeddings using an Ollama model,
    and stores the resulting vectors in a Supabase vector store.
    """

    # Define which HTML headers to split on (semantic chunking)
    headers_to_split_on = [
        ('h1', 'header1'),
        ('h2', 'header2'),
        ('h3', 'header3'),
    ]

    # Create the text splitter with a max chunk size
    text_splitter = HTMLSemanticPreservingSplitter(
        headers_to_split_on=headers_to_split_on,
        max_chunk_size=1000
    )

    # Split the cleaned HTML into smaller semantically meaningful chunks
    docs = text_splitter.split_text(result['cleaned_html'])

    # Add metadata and unique IDs to each chunked document
    for i, doc in enumerate(docs):
        doc.metadata = {
            'metadata': result['metadata'],
            'url': result['url'],
        }
        doc.id = result['url'] + '__' + str(i)  # Unique ID for each chunk

    # Initialize the Ollama embeddings model (using nomic-embed-text)
    embeddings = OllamaEmbeddings(model="nomic-embed-text")

    # Store the embedded documents into Supabase vector store for later retrieval
    vector_store = SupabaseVectorStore.from_documents(
        docs,                         # List of chunked documents
        embeddings,                   # Embedding model
        client=supabase_client,       # Supabase client connection
        table_name="documents",       # Target table for vector storage
        query_name="match_documents", # Name of the query function for retrieval (see init sql)
    )
```

Putting it all together: with `main.py` we:
- perform a breadth first crawl of the domain we set 
- for each page of the domain we:
  - extract properties that we write into `crawled_pages` (one-page one-row)
  - chunk the extracted text using LangChain's semantically preserving `HTMLSemanticPreservingSplitter` splitter
  - embed the text chunks with Ollama's `nomic-embed-text` model
  - write chunks and some metadata to Supabase table `documents`


The `crawled_pages` holds useful extracted data such as 
- internal and external url links, text, raw html
- metadata: title, author etc
- crawl specific data: depth (crawl depth) and parent-url
- Crawl4AI also extracts [Open Graph data](https://ogp.me/)


The `documents` table will hold several rows per page, one for each chunk that got embedded.


## Run the crawl 
Last step is setting up the cralwer environment and we are ready to run the
crawl.

### Install the Python Virtual Environment
In the root directory of the repo run:
```bash
python -m venv .venv
.venv/bin/pip install --upgrade pip
.venv/bin/pip install -r requirements.txt
```

### Set up Crawl4AI

This will 
- install required Playwright browsers (Chromium, Firefox, etc.)
- perform OS-level checks (e.g., missing libs on Linux)
- confirm your environment is ready to crawl
```bash
source .venv/bin/activate 
crawl4ai-setup
```

### Get Ollama running locally 
For embedding locally we'll start a local Ollama instance and pull `nomic-embed-text` model.

```bash
docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
docker exec -it ollama ollama pull nomic-embed-text
```

To test `ollama` is working locally the following command:

```bash
curl http://localhost:11434/api/embeddings -d '{
  "model": "nomic-embed-text",
  "prompt": "The sky is blue because of Rayleigh scattering"
}'
```

should return a json with embedding:
```json
{"embedding":[0.589758,...,0.480590]}
```

### Crawl and Embed

To start the crawl, run the following from the root directory:
```bash
source .venv/bin/activate 
python src/main.py
```

## First results
If you open your Supabase project you should start seeing the `crawled_pages` 
as well as the `documents` tables being populated. 

If you alreadt set up Supabase and N8N - (see [[Setting up N8N and Supabase for a Domain aware RAG App]]) -  you can now go over to your n8n workflow, open the chat node and start asking questions about the domain you are scraping.


## **Conclusion**

With just a few lines of code and the right tooling, you've now built a complete pipeline that crawls an entire domain, semantically splits and embeds its content, and stores it in a vector database—ready for powerful, retrieval-augmented AI. Whether you're indexing your company's website or preparing content for a domain-specific chatbot, Crawl4AI + Supabase + Ollama gives you a lean, production-ready stack to turn any website into a RAG-ready knowledge base. Happy crawling!

