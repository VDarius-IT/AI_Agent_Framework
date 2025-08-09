# Quickstart — AI Agent Framework

This quickstart gets a minimal local instance of the Autonomous AI Agent Framework running for development and experimentation.

Prerequisites
- Git
- Python 3.10+
- Docker (for RabbitMQ and optional monitoring components)
- (Optional) kubectl / Helm if you plan to run on Kubernetes

1) Clone repository
```bash
git clone <your-repo-url>
cd AI_Agent_Framework
```

2) Create and activate a Python virtual environment
```bash
python -m venv .venv
source .venv/bin/activate   # Linux / macOS
.venv\Scripts\activate      # Windows
```

3) Install Python dependencies
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

4) Start RabbitMQ (required for message bus)
```bash
docker run -d --hostname ai-rabbit --name ai-rabbit -p 5672:5672 -p 15672:15672 rabbitmq:3-management
```
Confirm the management UI at http://localhost:15672 (guest/guest default) — change credentials for production.

5) Start the orchestrator API (FastAPI / Uvicorn)
```bash
# from repository root
uvicorn orchestrator.main:app --reload --host 0.0.0.0 --port 8000
```
API will be available at: http://localhost:8000

6) Start example agents
- Check `examples/` or `agents/` for sample agent processes. Typical start:
```bash
python -m agents.sample_agent --config config.yaml
```

7) Optional: start monitoring stack (Prometheus + Grafana)
```bash
docker-compose -f monitoring/docker-compose.yml up -d
```

8) Submit a test task
Use curl or Postman to submit a task to the orchestrator:
```bash
curl -X POST http://localhost:8000/task -H "Content-Type: application/json" -d '{"task":"summarize","payload":"Short text to summarize."}'
```

Notes & Next Steps
- Configuration is managed in `config.yaml`. Update RabbitMQ host/port and agent registrations there.
- For multi-node testing, deploy individual agents on separate hosts or Docker containers and point them at the same RabbitMQ instance.
- To run in Kubernetes, prepare images (Dockerfile present) and install the Helm chart included in `helm/`.
