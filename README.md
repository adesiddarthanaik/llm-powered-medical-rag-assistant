# 🏥 Medical Chatbot — RAG-Powered Healthcare Assistant

<div align="center">

![Python](https://img.shields.io/badge/Python-3.10-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/Flask-3.1.1-000000?style=for-the-badge&logo=flask&logoColor=white)
![LangChain](https://img.shields.io/badge/LangChain-0.3.26-1C3C3C?style=for-the-badge&logo=langchain&logoColor=white)
![Pinecone](https://img.shields.io/badge/Pinecone-Vector%20DB-00A0DC?style=for-the-badge&logo=pinecone&logoColor=white)
![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4o-412991?style=for-the-badge&logo=openai&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-EC2%20%7C%20ECR-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

**An AI-powered healthcare assistant that answers medical questions from trusted literature — grounded, concise, and hallucination-resistant.**

[Features](#-key-features) · [Architecture](#-system-architecture) · [Setup](#-local-setup) · [Deployment](#-aws-cicd-deployment) · [Usage](#-usage)

</div>

---

## 📌 Overview

Medical Chatbot is a **Retrieval-Augmented Generation (RAG)** application that delivers reliable medical insights, disease diagnostics, and treatment information — instantly, without requiring a physical appointment.

Rather than relying on an LLM's pretrained knowledge (which can hallucinate), the system grounds every answer in **The Gale Encyclopedia of Medicine (2nd Edition)** — a 637-page verified healthcare reference. Questions are answered using only retrieved, relevant chunks from this source, keeping responses accurate, concise, and trustworthy.

> ⚠️ **Disclaimer**: This chatbot is an informational assistant. It does not replace professional medical advice, diagnosis, or treatment. Always consult a qualified healthcare provider.

---

## ✨ Key Features

| Feature | Description |
|---|---|
| 🧠 **RAG Pipeline** | Answers grounded in retrieved document chunks — prevents LLM hallucinations |
| 🔍 **Semantic Search** | Cosine similarity search over 384-dim embeddings via Pinecone |
| 📚 **Open-Source Embeddings** | `sentence-transformers/all-MiniLM-L6-v2` — no paid embedding API required |
| ✂️ **Context-Grounded Prompting** | System prompt enforces 3-sentence brevity and source fidelity |
| 💬 **Web Chat Interface** | Lightweight Flask frontend with real-time chat at `/` |
| ➕ **Dynamic Ingestion** | Append new documents to Pinecone index without full rebuilds |
| 🚀 **Production CI/CD** | Docker → ECR → EC2 via GitHub Actions self-hosted runners |

---

## 🏗️ System Architecture

### RAG Pipeline

```
User Query
    │
    ▼
┌─────────────────────────────────────────┐
│              Flask App (app.py)         │
│         Route: POST /get                │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│         LangChain Retrieval Chain       │
│   create_retrieval_chain()              │
│   create_stuff_documents_chain()        │
└────────┬──────────────────┬────────────┘
         │                  │
         ▼                  ▼
┌─────────────────┐  ┌──────────────────────┐
│  Pinecone Index │  │  ChatOpenAI (GPT-4o) │
│  medical-chatbot│  │  Temperature: 0.4    │
│  dim=384        │  │  Max context: k=3    │
│  metric=cosine  │  │  chunks retrieved    │
└────────┬────────┘  └──────────┬───────────┘
         │                      │
         └──────────┬───────────┘
                    │
                    ▼
         ┌──────────────────┐
         │   Final Answer   │
         │  (≤ 3 sentences) │
         └──────────────────┘
```

### CI/CD Pipeline

```
git push → main
      │
      ▼
┌─────────────────────────────────────────────────┐
│              GitHub Actions (cicd.yaml)         │
│                                                 │
│  [CI Job — GitHub-Hosted Runner]                │
│  1. Configure AWS Credentials                   │
│  2. Login to AWS ECR                            │
│  3. docker build -t medical-chatbot .           │
│  4. docker push → ECR Repository                │
│                                                 │
│  [CD Job — EC2 Self-Hosted Runner]              │
│  5. docker pull latest image from ECR           │
│  6. docker stop + rm existing container         │
│  7. docker run -p 8080:8080 (new container)     │
└─────────────────────────────────────────────────┘
      │
      ▼
 App live at http://<EC2-PUBLIC-IP>:8080
```

---

## 🛠️ Tech Stack

### Core Frameworks & Libraries

| Layer | Technology | Version |
|---|---|---|
| Language | Python | `3.10` |
| Web Framework | Flask | `3.1.1` |
| AI Orchestration | LangChain | `0.3.26` |
| LangChain Community | langchain-community | `0.3.26` |
| LangChain OpenAI | langchain-openai | `0.3.24` |
| LangChain Pinecone | langchain-pinecone | `0.2.8` |
| Embedding Model | sentence-transformers | `4.1.0` |
| PDF Parser | pypdf | `5.6.1` |
| Env Management | python-dotenv | `1.1.0` |

### Infrastructure

| Component | Service |
|---|---|
| LLM | OpenAI `gpt-4o` |
| Vector Database | Pinecone (Serverless, AWS `us-east-1`, Cosine) |
| Containerization | Docker (`python:3.10-slim`) |
| Container Registry | AWS ECR |
| Compute | AWS EC2 (Ubuntu, port `8080`) |
| CI/CD | GitHub Actions + EC2 Self-Hosted Runner |

---

## 📁 Directory Structure

```
build-a-complete-medical-chatbot/
│
├── .github/
│   └── workflows/
│       └── cicd.yaml              # GitHub Actions CI/CD workflow
│
├── data/
│   └── Medical_Book.pdf           # Gale Encyclopedia of Medicine (637 pages)
│
├── research/
│   └── trials.ipynb               # Experimental notebook for pipeline testing
│
├── src/
│   ├── __init__.py                # Python package initializer
│   ├── helper.py                  # ETL: load, filter, chunk, embed
│   └── prompt.py                  # System prompt template
│
├── static/
│   └── style.css                  # Chat UI stylesheet
│
├── templates/
│   └── chat.html                  # Web chat interface
│
├── .env                           # Local secrets (git-ignored)
├── .gitignore
├── app.py                         # Flask entry point + RAG chain
├── Dockerfile                     # Container build config
├── requirements.txt               # Pinned dependencies
├── setup.py                       # Local package setup
└── store_index.py                 # One-time vector store initialization
```

---

## 📂 File Breakdown

<details>
<summary><strong>src/helper.py</strong> — ETL Utility Functions</summary>

```python
load_pdf_files(data_path)
# Loads all PDFs from a directory using DirectoryLoader + PyPDFLoader

filter_to_minimal_docs(extracted_data)
# Strips heavy PDF metadata, retains only source + page_content

text_split(minimal_docs)
# Splits text using RecursiveCharacterTextSplitter
# chunk_size=500, chunk_overlap=20

download_embedding()
# Loads HuggingFace all-MiniLM-L6-v2 (384-dimensional embeddings)
```

</details>

<details>
<summary><strong>src/prompt.py</strong> — System Prompt Template</summary>

```python
system_prompt = (
    "You are an assistant for question-answering tasks. "
    "Use the following pieces of retrieved context to answer "
    "the question. If you don't know the answer, say that you "
    "don't know. Use three sentences maximum and keep the "
    "answer concise.\n\n"
    "{context}"
)
```

</details>

<details>
<summary><strong>store_index.py</strong> — One-Time Vector DB Indexing</summary>

Run this **once** to build the Pinecone index:
1. Loads PDFs from `./data/`
2. Filters metadata and creates text chunks
3. Downloads the embedding model
4. Creates Pinecone index: `medical-chatbot` | `dim=384` | `metric=cosine` | `cloud=AWS` | `region=us-east-1`
5. Populates index via `PineconeVectorStore.from_documents()`

</details>

<details>
<summary><strong>app.py</strong> — Flask Application + RAG Chain</summary>

- Loads `.env` secrets
- Initializes Pinecone retriever (`search_type="similarity"`, `k=3`)
- Builds chain: `ChatOpenAI(gpt-4o)` → `create_stuff_documents_chain` → `create_retrieval_chain`
- `GET /` → Renders `chat.html`
- `POST /get` → Accepts user prompt, invokes RAG chain, returns response text

</details>

<details>
<summary><strong>Dockerfile</strong></summary>

```dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY . /app
RUN pip install --no-cache-dir -r requirements.txt
EXPOSE 8080
CMD ["python", "app.py"]
```

</details>

---

## 🔐 Environment Variables

Create a `.env` file in the project root:

```env
PINECONE_API_KEY=your_pinecone_api_key_here
OPENAI_API_KEY=your_openai_api_key_here
```

### GitHub Actions Secrets (for AWS Deployment)

Go to **Settings → Secrets and variables → Actions** and add:

| Secret | Description |
|---|---|
| `AWS_ACCESS_KEY_ID` | IAM user access key |
| `AWS_SECRET_ACCESS_KEY` | IAM user secret key |
| `AWS_DEFAULT_REGION` | `us-east-1` |
| `ECR_REPO_NAME` | `medical-chatbot` |
| `PINECONE_API_KEY` | Your Pinecone API key |
| `OPENAI_API_KEY` | Your OpenAI API key |

---

## 💻 Local Setup

### Prerequisites
- Python `3.10`
- Conda (recommended) or `venv`
- A [Pinecone](https://pinecone.io) account (free tier works)
- An [OpenAI](https://platform.openai.com) API key

### Step-by-Step

**1. Clone the repository**
```bash
git clone https://github.com/your-username/build-a-complete-medical-chatbot.git
cd build-a-complete-medical-chatbot
```

**2. Create and activate virtual environment**
```bash
conda create -n medbot python=3.10 -y
conda activate medbot
```

**3. Install dependencies**
```bash
pip install -r requirements.txt
```

**4. Configure environment secrets**
```bash
# Create .env in project root
echo "PINECONE_API_KEY=your_key_here" >> .env
echo "OPENAI_API_KEY=your_key_here" >> .env
```

**5. Place your medical PDF in `data/`**
```bash
# Ensure the file is present:
ls data/Medical_Book.pdf
```

**6. Initialize the vector database** *(run once)*
```bash
python store_index.py
```
> This creates the `medical-chatbot` Pinecone index and ingests all document chunks. Takes a few minutes on first run.

**7. Launch the application**
```bash
python app.py
```

Open your browser at → **`http://localhost:8080`**

---

## ☁️ AWS CI/CD Deployment

### 1. Create IAM User
- Attach policies: `AmazonEC2FullAccess` + `AmazonEC2ContainerRegistryFullAccess`
- Generate and save: **Access Key ID** and **Secret Access Key**

### 2. Create ECR Repository
- Go to AWS Console → ECR → Create repository
- Name: `medical-chatbot`
- Visibility: **Private**

### 3. Launch EC2 Instance
- AMI: **Ubuntu Server** (latest LTS)
- Instance type: `t3.medium` or higher (≥ 8 GB RAM recommended)
- Security group: Open **Custom TCP port `8080`** from `0.0.0.0/0`
- SSH into the instance and install Docker:

```bash
sudo apt-get update -y
sudo apt-get install docker.io -y
sudo usermod -aG docker ubuntu
newgrp docker
```

### 4. Configure GitHub Actions Self-Hosted Runner
- Go to **GitHub Repo → Settings → Actions → Runners → New self-hosted runner**
- Select Linux, follow the setup commands on your EC2 instance

### 5. Add GitHub Secrets
- Navigate to **Settings → Secrets and variables → Actions**
- Add all secrets listed in the [Environment Variables](#-environment-variables) section

### 6. Deploy
```bash
git add .
git commit -m "feat: deploy medical chatbot"
git push origin main
```

The GitHub Actions workflow automatically:
- Builds and pushes the Docker image to ECR
- SSHes into EC2 via the self-hosted runner
- Pulls the latest image and restarts the container

**App live at:** `http://<your-EC2-public-IP>:8080`

---

## 💬 Usage

1. Open the web interface at `http://localhost:8080` (or your EC2 URL)
2. Type a medical question in the chat input
3. The system retrieves the 3 most relevant chunks from the Pinecone index
4. GPT-4o synthesizes a concise answer (≤ 3 sentences) grounded in the retrieved context
5. If the answer isn't in the source material, the bot says so — no hallucinations

**Example questions to try:**
- *"What are the symptoms of Type 2 Diabetes?"*
- *"How is hypertension typically treated?"*
- *"What is the diagnostic procedure for appendicitis?"*

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

<div align="center">

Built with LangChain · Pinecone · OpenAI · Flask · Docker · AWS

</div>

