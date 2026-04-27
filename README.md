# Nebius RAG Evaluation: FinanceBench

This repository contains a comprehensive pipeline for building and evaluating a Retrieval-Augmented Generation (RAG) system tailored for financial documents. The project leverages the **FinanceBench** dataset to assess the system's ability to answer complex financial questions using 10-K and 10-Q filings.

## 🚀 Overview

The pipeline is designed to demonstrate the transition from a "Naive" LLM approach to a robust RAG system. It includes document processing, vector indexing with FAISS, and an automated evaluation framework to measure performance and guide optimization cycles.

## 📊 Dataset & Source Documents

To run this project, you will need access to the following resources:

1.  **FinanceBench Dataset**: Available on [Hugging Face](https://huggingface.co/datasets/PatronusAI/financebench). This dataset contains 10,231 questions and answers based on public financial filings, including ground-truth evidence strings and page numbers.
2.  **Source PDFs**: The original 10-K and 10-Q filings are hosted in the [PatronusAI FinanceBench GitHub repository](https://github.com/patronus-ai/financebench/tree/main/pdfs).
3.  **Google Colab**: You can also view and run the project directly on [Google Colab](https://colab.research.google.com/drive/1dYsq8bXIZIGBto17LB2OS_Wo7MXNBdEd).

The pipeline is configured to automatically fetch the dataset and download the required PDFs. If you are running this locally, ensure you have an internet connection and the necessary API keys configured (see Technology Stack).

## 🛠️ Technology Stack

- **LLM**: `meta-llama/Llama-3.3-70B-Instruct` (via [Nebius Token Factory](https://tokenfactory.nebius.com))
- **Embeddings**: `BAAI/bge-small-en-v1.5`
- **Vector Store**: [FAISS](https://github.com/facebookresearch/faiss)
- **Framework**: LangChain & LangChain-Community
- **Data**: FinanceBench (PatronusAI)
- **Storage**: PDF filings cached locally; FAISS indices persisted on disk.

---

## 🏗️ RAG Build Process

The construction of the RAG system follows a modular approach:

### 1. Data Acquisition & Preprocessing
- **Dataset**: Loads the `PatronusAI/financebench` dataset from Hugging Face.
- **Document Retrieval**: Automatically downloads relevant financial PDFs (10-K, 10-Q) from GitHub based on the dataset's document links.
- **Parsing**: Uses `PyPDFLoader` to extract text from PDF pages while preserving metadata such as `doc_name`, `company`, `doc_period`, and `page_number`.

### 2. Chunking Strategy
Documents are split into smaller segments to fit within the embedding model's context window and improve retrieval precision.
- **Tool**: `RecursiveCharacterTextSplitter`
- **Baseline Configuration**: Chunk size of **1000 characters** with a **150-character overlap**.
- **Experimental Variations**: The project explores multiple chunk sizes (300, 500, 1500, 2000) to find the optimal balance between granularity and context.

### 3. Vector Indexing (FAISS)
- **Embeddings**: Text chunks are converted into dense vectors using the `BAAI/bge-small-en-v1.5` model.
- **Index Building**: A FAISS vector store is built for efficient similarity-based retrieval.
- **Persistence**: Indices are saved in the `indices/` directory (e.g., `faiss_chunk1000/`), allowing for fast loading without re-embedding.

---

## 🔍 Evaluation Process

A core focus of this project is the **automated evaluation** of the RAG pipeline. We use "LLM-as-a-judge" alongside deterministic retrieval metrics.

### 📊 Key Metrics
1.  **Correctness**: An LLM judge compares the RAG-generated answer against the ground truth from FinanceBench, providing a binary (1/0) score.
2.  **Faithfulness**: An LLM judge verifies if the generated answer is strictly supported by the retrieved context chunks (detecting hallucinations).
3.  **Retrieval Page Hit@k**: A deterministic metric that checks if the ground-truth evidence page (annotated in FinanceBench) is present in the top-k retrieved chunks (evaluated for $k \in \{1, 3, 5\}$).

### 🔄 Optimization Cycles
The evaluation framework supports iterative improvement:
- **Baseline**: Initial run with chunk size 1000 and top-k=5.
- **Experiment 1**: Increasing top-k (e.g., from 5 to 8).
- **Experiment 2**: Varying chunk sizes to observe the impact on `Page Hit@k` and `Correctness`.
- **Observations**: Results are exported to Excel files in the `outputs/` directory for detailed analysis and comparison.

---

## 📂 Repository Structure

- `rag_eval.ipynb`: The main notebook containing the full implementation and experiments.
- `rag_eval.py`: Python script version of the RAG pipeline.
- `indices/`: Persisted FAISS indices for different chunking configurations.
- `pdfs/`: Local cache of downloaded financial documents.
- `outputs/`: Evaluation results and improvement cycle logs (Excel format).
- `cache/`: Cached cleaned datasets and metadata.

---

## 📈 Results Summary

Initial evaluations indicated that **retrieval quality** is the primary bottleneck. While the generator (`Llama-3.3-70B`) is highly faithful to the provided context, the system often struggles to retrieve the exact page containing the supporting evidence. Future improvements involve hybrid search (combining keyword and semantic search) and reranking.
emantic search) and reranking.
