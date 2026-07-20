# RAG_Prep

A Jupyter Notebook for experimenting with Retrieval Augmented Generation (RAG) using `.jats` XML article files

## Introduction

This repository is intended as a starting point for introductory courses or workshops mainly for users without a strong technical background. It provides a lightweight framework for exploring the basic concepts of Retrieval Augmented Generation (RAG). Since the other repositories in this account focus on scientific journal articles, this notebook is also designed to work mainly with `.jats` XML documents. 

Although many tutorials already explain the principles of RAG very intuitively (see for example  https://huggingface.co/blog/ngxson/make-your-own-rag), this notebook is intended to provide a simple environment for experimenting with larger collections of documents and the complete RAG workflow.

## Features

The Jupyter Notebook makes it possible to experiment with the complete preparation pipeline required for a simple RAG system:

- collecting and importing text documents (in this case primarily `.jats XML` files)
- parsing and extracting metadata
- splitting documents into text chunks
- creating embedding vectors
- storing embeddings together with metadata in a vector database
- answering questions using RAG

Several parameters can be configured in the `config.yaml` file.

The notebook has been tested with:
- `qwen3-embedding:4b` (generating embeddings)
- `mistral:7b-instruct` (chat model)
- `chromadb 1.5.9` (vector database)
- `jupyterlab 4.5.7`
- `Python 3.13.5`

Note: If you choose a different embedding model, please keep in mind that the vector dimensions may differ. It is therefore advisable to go through the entire process again when changing embedding models (see also `config.yaml`).

## Source files

To keep the example simple, the notebook reads article files from a local directory (see the `Collect Source Files` cell). However, `.jats` files can easily be harvested from many scientific publishing platforms.

The XML files are parsed using the `BeautifulSoup` library, assuming they are well-formed (which is unfortunately not always the case in practice).

In the current version, only the article title and DOI are extracted and stored as metadata. Additional metadata can easily be added if required.

## Chunking and metadata

Each imported source document is assigned a UUID, which is stored together with the file name and source directory in `metadataProject.json`.

The article text is then divided into smaller chunks using the `langchain_text_splitters` library (see the `Create Chunks` cell). Chunk size and overlap can be adjusted in `config.yaml`.

Each chunk receives the metadata required by the vector database:
- `ids` (a unique chunk identifier consisting of the article UUID with an incrementing suffix)
- `documents` (the text chunk itself)
- `embeddings` (the embedding vector)

[Caution: If ChromaDB encounters missing embeddings, it will automatically generate them. This is intentionally avoided here, since the embeddings will be created later. Therefore a dummy vector with the correct dimensionality (specified in `config.yaml`) is temporarily inserted in this current step.]

- `metadatas` – additional information for each chunk, including:
    - the article UUID
    - the article DOI (if available)
    - the article section (e.g. extracted from `<sec>` elements)
    - a section-specific chunk identifier

Each chunk, together with its metadata and embedding placeholder, is stored as a single entry in the ChromaDB collection.

## Embedding generation

The `Create Embeddings` cell generates an embedding vector for every document chunk stored in the database. The generated embeddings replace the previously inserted placeholder vectors.

## Retrieval and content generation

The final notebook section allows users to experiment with:
- different user questions
- retrieval settings
- prompt construction
- the retrieved context supplied to the language model

This makes it easy to observe how retrieval influences the generated answers.

## Additional scripts

### Scripts for interacting with the database

This section contains several predefined database queries. The scripts simplify the inspection of the ChromaDB collection, provide an illustrative insight into the database and give an impression of how to interact with ChromaDB.

### Clean up

Since each import in step 1 generates new UUIDs, repeated runs create obsolete data and database entries. The cleanup script removes files from previous runs that are no longer needed and refreshes the database by deleting all entries.

## Acknowledgements

Parts of this notebook were developed with the assistance of ChatGPT and Ollama language models. These tools contributed to tasks including XML parsing, integration with ChromaDB, prompt design, and implementation of the RAG pipeline. They also provided useful suggestions for workflow design and configuration.

In this respect this repository also demonstrates how AI support enables users with limited programming experience to explore and experiment with retrieval-based applications.

This work also benefited from the WiNoDa Knowledge Lab, which provided the computing environment used to run the notebook, process larger document collections, and experiment with several pre-installed language models.

