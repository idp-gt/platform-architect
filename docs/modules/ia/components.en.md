# AI Google Components

This section centralizes AI templates (Machine Learning, Deep Learning, Generative AI)

## Architecture
The code is modularized to allow reusability in different environments (Dev, Staging, Prod) in Cloud and On-Prem.

!!! info "Tech Stack"
    * **LlamaIndex:** It is a Python (and JS) framework used to connect language models (LLMs) like GPT with your own data (PDFs, databases, APIs, logs, etc.)
      
        **What is it for?**

        Read and load data (PDF, CSV, SQL, JSON, APIs, etc.)

        Split them into chunks

        Convert them into embeddings

        Query them intelligently using an LLM

        **In a nutshell:**

        👉 LlamaIndex takes care of preparing, organizing, and querying your data using AI.




    * **Qdrant:** It is a vector database. It is designed to store and search embeddings (numerical vectors that represent meaning).

        **What is it for?**

        Store embeddings of texts, images, audio, etc.

        Search for information by semantic similarity, not by exact words

        Scale fast searches across millions of vectors

        **In a nutshell:**
        👉 Qdrant is where vectors are saved and searched.
