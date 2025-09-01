# 🧠 Django Supportal – AI-Powered Business Support Chat APIs for django projects

**Django Supportal** is an intelligent, AI-powered customer support system built with **Django**, **Django Channels**, and **OpenAI API**.  
It provides APIs for businesses to upload their internal documents, and a smart assistant will handle customer inquiries via live chat – powered by a Retrieval-Augmented Generation (RAG) system.

---

## Contribution

I’d be really happy to see you join the development of this project!
Whether it’s sharing ideas, reporting bugs, or writing some code — your contributions are truly appreciated ❤️

---

## 🚀 Features

- ✅ Real-time chat via **Django Channels (WebSockets)**
- 📎 Businesses can upload **PDF, DOCX, or TXT documents**
- 🤖 Uses **OpenAI GPT models** to provide intelligent responses
- 📚 Implements **RAG (Retrieval-Augmented Generation)** to process custom business knowledge
- 🔒 Secured communication and Redis-based event layer

---

## 🧠 How it Works (RAG Architecture)

Supportal uses a **Retrieval-Augmented Generation (RAG)** approach to enable AI to answer business-specific questions:

1. **Document Upload:**  
   Businesses upload documents such as FAQs, product guides, manuals, or policies.

2. **Chunking & Embedding:**  
   Uploaded documents are:
   - Split into smaller text chunks
   - Converted into **vector embeddings** using OpenAI's `text-embedding` models

3. **Vector Storage:**  
   Embeddings are stored in a **vector database** (like FAISS) for fast similarity search.

4. **Chat Inference:**
   - When a customer sends a message, it's embedded and compared against stored chunks.
   - The most relevant chunks are selected as **context**.
   - The context is fed into OpenAI's **chat completion API** along with the user's question.
   - A tailored, relevant answer is generated based on actual business documents.

> This allows Supportal to **answer domain-specific questions accurately**, beyond what a generic AI model can do.

---

## 🛠️ Tech Stack

- **Backend:** Django + Django Channels
- **Realtime Layer:** Redis (via `channels_redis`)
- **AI Engine:** OpenAI API (GPT + Embeddings)
- **Vector DB:** FAISS (in-memory vector search)

---

## 📦 Getting Started

### 🔧 Prerequisites

- Django
- Channels
- Celery
- Redis
- OpenAI API key

### 🧪 Installation

### 🧪 Installation

#### 1. Install the package

```bash
pip install django-supportal
```

#### 2. Add to your Django project

Add `django_supportal` to your `INSTALLED_APPS` in `settings.py`:

```python
INSTALLED_APPS = [
    # ... other apps
    'django_supportal',
    'channels',
    'rest_framework',
]
```

#### 3. Include URLs in your main `urls.py`

```python
from django.urls import path, include

urlpatterns = [
    # ... your other URL patterns
    path('supportal/', include('django_supportal.urls')),
]
```

#### 4. Configure settings

Add the following configuration to your `settings.py`:

```python
# Supportal Configuration
SUPPORTAL_SETTINGS = {
    "OPENAI_API_KEY": "your-openai-api-key-here",
    "OPENAI_MODEL": "gpt-3.5-turbo",  # or "gpt-4"
    "OPENAI_EMBEDDING_MODEL": "text-embedding-ada-002",
    "MAX_TOKENS": 1000,
    "TEMPERATURE": 0.7,
    "CHUNK_SIZE": 1000,
    "CHUNK_OVERLAP": 200,
    "TOP_K_RESULTS": 5,
    "VECTOR_DB_PATH": "vector_db/",
    "ALLOWED_FILE_TYPES": ["pdf", "docx", "txt"],
    "MAX_FILE_SIZE": 10 * 1024 * 1024,  # 10MB
    "REDIS_URL": "redis://localhost:6379/0",
    "CELERY_BROKER_URL": "redis://localhost:6379/0",
    "ENABLE_LOGGING": True,
    "LOG_LEVEL": "INFO",
}

# Channel Layers (for WebSocket support)
CHANNEL_LAYERS = {
    "default": {
        "BACKEND": "channels.layers.InMemoryChannelLayer",
    },
}

# Celery Configuration
CELERY_BROKER_URL = SUPPORTAL_SETTINGS["CELERY_BROKER_URL"]
CELERY_RESULT_BACKEND = SUPPORTAL_SETTINGS["CELERY_BROKER_URL"]
CELERY_ACCEPT_CONTENT = ["json"]
CELERY_TASK_SERIALIZER = "json"
CELERY_RESULT_SERIALIZER = "json"
CELERY_TIMEZONE = "UTC"
```

#### 5. Run migrations

```bash
python manage.py migrate
```

#### 6. Start required services

Make sure you have Redis running:

```bash
# Install Redis (if not already installed)
# macOS: brew install redis
# Ubuntu: sudo apt-get install redis-server

# Start Redis
redis-server
```

#### 7. Start Celery worker (in a separate terminal)

```bash
celery -A your_project_name worker --loglevel=info
```

#### 8. Run your Django server

```bash
python manage.py runserver
```

### 🎯 Available URLs

After installation, the following URLs will be available:

- **Admin Dashboard:** `http://localhost:8000/supportal/admin-dashboard/`
- **API Endpoints:** `http://localhost:8000/supportal/api/`
  - Businesses: `/supportal/api/businesses/`
  - Documents: `/supportal/api/documents/`
  - Chat Sessions: `/supportal/api/chat-sessions/`
- **Chat Interface:** `http://localhost:8000/supportal/chats/{business_id}/{session_id}/`
- **Health Check:** `http://localhost:8000/supportal/health/`

### 🔧 Environment Variables

For production, use environment variables:

```bash
export OPENAI_API_KEY="your-openai-api-key"
export REDIS_URL="redis://your-redis-host:6379/0"
export CELERY_BROKER_URL="redis://your-redis-host:6379/0"
```

## 📄 License
This project is licensed under the **MIT License** – see the [LICENSE](./LICENSE) file for details.