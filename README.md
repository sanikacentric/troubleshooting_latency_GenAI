# Troubleshooting Latency in GenAI Chatbots

A comprehensive guide and implementation for diagnosing and resolving latency bottlenecks in production GenAI chatbot systems on Google Cloud Platform. This repository includes Terraform infrastructure-as-code for GCP, a Flask-based chatbot API with Redis caching, async Pub/Sub embedding pipelines, query normalization, and FAISS vector search — all optimized for low-latency inference.

## Description

High latency is one of the most common pain points when deploying LLM-based chatbots at scale. This repository demonstrates a production-grade architecture and implementation that tackles latency at every layer: infrastructure provisioning via Terraform, response caching with Redis (Cloud Memorystore), asynchronous embedding generation via Cloud Pub/Sub, semantic similarity caching with FAISS, query normalization to maximize cache hits, and Cloud Run auto-scaling for horizontal scaling.

## Features

- **Terraform Infrastructure as Code** — Fully automated GCP infrastructure provisioning using Terraform, including Cloud Run, Redis, Pub/Sub, and networking resources
- - **Redis Response Caching** — Cache LLM responses in Cloud Memorystore (Redis) with query hash keys to avoid redundant model calls
  - - **Query Normalization** — Lowercase, punctuation removal, and stopword filtering to increase cache hit rates for semantically identical queries
    - - **Async Embedding Pipeline** — Publish embedding requests to Cloud Pub/Sub for non-blocking background processing, reducing API response latency
      - - **FAISS Vector Search** — In-memory semantic similarity search using FAISS to find cached answers for similar (not identical) queries
        - - **Vertex AI Embeddings** — Use `textembedding-gecko@003` for generating high-quality semantic embeddings
          - - **Cloud Run Deployment** — Containerized Flask app deployed on Cloud Run with auto-scaling for handling variable traffic
            - - **Latency Profiling** — Instrumented code paths to measure time spent at each stage (cache lookup, embedding, LLM call, vector search)
             
              - ## Architecture
             
              - ```
                User Query
                    ↓
                [Query Normalization]
                  - Lowercase, remove punctuation, filter stopwords
                    ↓
                [Redis Cache Lookup] (Cloud Memorystore)
                  ├── HIT → Return cached response (< 5ms)
                  └── MISS → Continue
                    ↓
                [FAISS Semantic Cache Lookup]
                  ├── Similar query found (cosine > threshold) → Return cached response
                  └── Not found → Continue
                    ↓
                [Pub/Sub Embedding Request] (async)
                    ↓
                [Vertex AI Inference]
                  - textembedding-gecko@003 + LLM
                    ↓
                [Cache Response in Redis + FAISS]
                    ↓
                Return Response to User
                ```

                ## Latency Optimization Strategies

                | Strategy | Latency Savings | Implementation |
                |---|---|---|
                | Redis Exact Cache Hit | Up to 500ms | Hash-based response caching |
                | FAISS Semantic Cache | Up to 400ms | Vector similarity search |
                | Query Normalization | Indirect (higher cache hit rate) | Text preprocessing |
                | Async Embedding | 100–200ms | Pub/Sub background processing |
                | Cloud Run Warm Instances | 0ms cold start | Min-instances configuration |

                ## Tech Stack

                | Component | Technology |
                |---|---|
                | Infrastructure | Terraform |
                | Web Framework | Flask |
                | Caching | Redis / Google Cloud Memorystore |
                | Vector Search | FAISS |
                | Embeddings | Vertex AI textembedding-gecko@003 |
                | Async Messaging | Google Cloud Pub/Sub |
                | Compute | Google Cloud Run |
                | Container Registry | Google Container Registry (GCR) |
                | Language | Python 3 |
                | Notebook | Jupyter (Google Colab) |

                ## Repository Structure

                ```
                troubleshooting_latency_GenAI/
                ├── troubleshooting_chatbot.ipynb    # Main notebook with all code
                │   ├── infrastructure/main.tf       # Terraform GCP infrastructure
                │   ├── application/main.py          # Flask API with caching & latency fixes
                │   └── Deployment & testing guide
                ```

                ## Setup Instructions

                ### Prerequisites

                - Python 3.8+
                - - Terraform 1.0+
                  - - Google Cloud SDK (`gcloud`)
                    - - Docker (for building the container)
                      - - A GCP project with the following APIs enabled:
                        -   - Cloud Run API
                            -   - Cloud Memorystore (Redis) API
                                -   - Vertex AI API
                                    -   - Cloud Pub/Sub API
                                        -   - Container Registry API
                                         
                                            - ### Step 1: Provision Infrastructure with Terraform
                                         
                                            - ```bash
                                              # Navigate to the infrastructure section from the notebook
                                              # Set your GCP project
                                              export TF_VAR_project_id=your-project-id
                                              export TF_VAR_region=us-central1

                                              # Initialize and apply
                                              terraform init
                                              terraform plan
                                              terraform apply
                                              ```

                                              ### Step 2: Build and Push the Container

                                              ```bash
                                              # Build the Docker image
                                              docker build -t gcr.io/$PROJECT_ID/chatbot-api:latest .

                                              # Push to Container Registry
                                              docker push gcr.io/$PROJECT_ID/chatbot-api:latest
                                              ```

                                              ### Step 3: Install Python Dependencies

                                              ```bash
                                              pip install flask google-cloud-aiplatform google-cloud-pubsub redis faiss-cpu numpy jupyter
                                              ```

                                              ### Step 4: Run the Notebook

                                              ```bash
                                              jupyter notebook troubleshooting_chatbot.ipynb
                                              ```

                                              ### Environment Variables

                                              | Variable | Description |
                                              |---|---|
                                              | `GOOGLE_CLOUD_PROJECT` | Your GCP project ID |
                                              | `REDIS_HOST` | Redis / Memorystore host IP |
                                              | `REDIS_PORT` | Redis port (default: 6379) |
                                              | `EMBEDDING_MODEL_ID` | Vertex AI embedding model |
                                              | `EMBEDDING_TOPIC_ID` | Pub/Sub topic for async embeddings |

                                              ## Diagnosing Latency Issues

                                              Common latency problems and their fixes demonstrated in this repository:

                                              **High P99 Latency**: Usually caused by cold starts on Cloud Run — fix with min-instances and warm-up requests.

                                              **Cache Miss Rate > 30%**: Often due to minor query variations — fix with query normalization (lowercase, stopword removal) before hashing.

                                              **Embedding Generation Bottleneck**: Synchronous embedding calls block the response — fix with async Pub/Sub pipeline.

                                              **Repeated Similar Queries**: Exact cache misses for semantically identical questions — fix with FAISS semantic similarity caching.

                                              ## License

                                              Open source — for educational and cloud architecture demonstration purposes.
