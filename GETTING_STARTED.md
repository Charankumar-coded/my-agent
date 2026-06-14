# Getting Started Guide - Professional AI Agent Platform

## Overview

This AI Agent Platform provides a sophisticated interface for strategic task analysis, planning, and execution guidance. It includes both a professional frontend interface and a Python backend for extensibility.

## Quick Start (Frontend Only - No Backend)

### 1. Simple HTTP Server

#### Using Python:
```bash
cd /path/to/my-agent
python -m http.server 8000
```

Then visit: **http://localhost:8000**

#### Using Node.js:
```bash
npm install -g http-server
cd /path/to/my-agent
http-server . -p 8000
```

#### Using PHP:
```bash
cd /path/to/my-agent
php -S localhost:8000
```

### 2. Start Using
1. Open the application in your browser
2. Enter a professional task or use quick prompts
3. Click "Run Agent" to see the analysis flow
4. Review insights across 4 phases: Think → Plan → Execute → Deliver

## Backend Setup (Optional - For Live AI Responses)

### Prerequisites
- Python 3.9+
- pip or conda

### Installation Steps

#### 1. Clone or download the project
```bash
cd my-agent
```

#### 2. Create Python Virtual Environment
```bash
# Using venv
python -m venv venv

# Activate virtual environment
# On Windows:
venv\Scripts\activate
# On macOS/Linux:
source venv/bin/activate
```

#### 3. Install Dependencies
```bash
pip install -r requirements.txt
```

#### 4. Run Backend Server
```bash
python app.py
```

Expected output:
```
INFO:     Started server process [12345]
INFO:     Uvicorn running on http://0.0.0.0:8001
INFO:     Application startup complete
```

#### 5. Configure Frontend for Backend

Edit `index.html` and modify the `runAgent()` function to call the backend:

Replace this section in JavaScript:
```javascript
// PHASE 1: THINK
thinkResult = buildDemoResponse('think', task);
```

With:
```javascript
// PHASE 1: THINK
const thinkResp = await fetch('http://localhost:8001/analyze', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({task, phase: 'think'})
});
const thinkData = await thinkResp.json();
thinkResult = thinkData.response;
```

Repeat for 'plan', 'execute', and 'deliver' phases.

### API Documentation

Once backend is running, view interactive API docs:
- **Swagger UI**: http://localhost:8001/docs
- **ReDoc**: http://localhost:8001/redoc

### Docker Deployment

#### Using Docker (Recommended)
```bash
# Build image
docker build -t ai-agent-backend .

# Run container
docker run -p 8001:8001 ai-agent-backend
```

#### Using Docker Compose (Frontend + Backend)
```bash
# Start both services
docker-compose up

# Frontend: http://localhost:8000
# Backend: http://localhost:8001
# API Docs: http://localhost:8001/docs
```

## Architecture

```
┌─────────────────────────────────────┐
│   Frontend (HTML/CSS/JavaScript)    │
│   - Professional UI                  │
│   - Real-time streaming             │
│   - Responsive design               │
└──────────────┬──────────────────────┘
               │ HTTP/REST
               ▼
┌─────────────────────────────────────┐
│   Backend (FastAPI/Python)          │
│   - Task analysis                   │
│   - Category detection              │
│   - Response generation             │
└─────────────────────────────────────┘
```

## Task Categories Supported

### 1. **FastAPI & REST APIs**
   - Production-grade API development
   - Authentication and security
   - Database integration
   - Monitoring and observability

### 2. **Machine Learning Ops**
   - Model versioning and registry
   - A/B testing frameworks
   - Performance monitoring
   - Deployment strategies

### 3. **Data Pipelines**
   - Real-time ETL design
   - Event streaming architecture
   - Data quality validation
   - Scalability patterns

### 4. **Microservices Architecture**
   - Service design and boundaries
   - Inter-service communication
   - Kubernetes deployment
   - Distributed tracing

### 5. **Automated Testing**
   - Test strategy and organization
   - CI/CD integration
   - Coverage tracking
   - Performance benchmarks

## Advanced Features

### 1. Integration with AI Models

To use GPT-4, Claude, or other LLMs:

```python
# In app.py, modify generate_think_response():
import openai

def generate_think_response(task: str, category: str) -> str:
    client = openai.OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
    response = client.chat.completions.create(
        model="gpt-4",
        messages=[{
            "role": "user",
            "content": f"Analyze this task: {task}"
        }],
        temperature=0.7
    )
    return response.choices[0].message.content
```

### 2. Database Persistence

Add SQLAlchemy to store analysis history:

```python
from sqlalchemy import create_engine, Column, String, DateTime
from sqlalchemy.orm import declarative_base

Base = declarative_base()

class Analysis(Base):
    __tablename__ = "analyses"
    id = Column(String, primary_key=True)
    task = Column(String)
    category = Column(String)
    think_result = Column(String)
    plan_result = Column(String)
    created_at = Column(DateTime)
```

### 3. Rate Limiting & Caching

```python
from slowapi import Limiter
from slowapi.util import get_remote_address
from functools import lru_cache

limiter = Limiter(key_func=get_remote_address)

@app.post("/analyze")
@limiter.limit("30/minute")
async def analyze(request: TaskRequest):
    # Your endpoint code
    pass
```

## Troubleshooting

### Backend won't start
```bash
# Check Python version
python --version  # Should be 3.9+

# Check if port 8001 is in use
lsof -i :8001  # macOS/Linux
netstat -ano | findstr :8001  # Windows

# Kill process and retry
kill -9 <PID>  # macOS/Linux
taskkill /PID <PID> /F  # Windows
```

### CORS errors in frontend
- Update `allow_origins` in app.py
- Ensure backend is running before frontend
- Check browser console for exact error

### Slow responses
- Increase timeout in frontend (edit `typeInto()` function)
- Check backend logs for performance issues
- Consider adding response caching

## Performance Optimization

### Frontend
- Responses stream at 4-6ms per character
- Adjust speed in `typeInto()` function
- Debounce input for better UX

### Backend
- Add caching layer (Redis)
- Implement response pooling
- Use async operations for I/O

## Production Deployment

### AWS EC2/ECS
```bash
# Build and push to ECR
aws ecr get-login-password | docker login --username AWS --password-stdin <account>.dkr.ecr.us-east-1.amazonaws.com
docker tag ai-agent-backend:latest <account>.dkr.ecr.us-east-1.amazonaws.com/ai-agent:latest
docker push <account>.dkr.ecr.us-east-1.amazonaws.com/ai-agent:latest
```

### Kubernetes Deployment
```bash
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ai-agent
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ai-agent
  template:
    metadata:
      labels:
        app: ai-agent
    spec:
      containers:
      - name: agent
        image: ai-agent-backend:latest
        ports:
        - containerPort: 8001
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
EOF
```

## Support & Resources

- **Documentation**: See README.md
- **API Reference**: http://localhost:8001/docs (when backend running)
- **Issues**: Check logs with `docker logs <container-id>`
- **Examples**: Review task prompts in the quick prompts section

## Next Steps

1. ✅ Start the frontend
2. ✅ Test with quick prompts
3. ✅ (Optional) Setup backend
4. ✅ (Optional) Integrate with AI model
5. ✅ Deploy to production

---

**Version**: 2.0  
**Last Updated**: 2024-12-14  
**Professional AI Agent Platform**
