# Architecture — AI Agent Framework

This document describes the main components, interactions, and deployment considerations.

Overview
- The system is microservice-based and designed for horizontal scaling.
- Core components:
  - Orchestrator (FastAPI): Central control plane that accepts tasks, applies routing policy (RL), and exposes management APIs.
  - Message Broker: RabbitMQ for async task distribution and decoupling.
  - Agents: Worker processes that subscribe to RabbitMQ queues and run tools (search, summarizer, code-runner, etc.)
  - Monitoring: OpenTelemetry instrumentation, Prometheus metrics, Grafana dashboards.
  - RL Router: Service/component that evaluates agent effectiveness and updates routing policy (PPO training loop in rl_training/).

Data flows
1. Client POSTs a task to `/task` on the Orchestrator.
2. Orchestrator validates and enqueues the task into RabbitMQ with metadata (task type, priority).
3. Agents consume tasks from relevant queues; agents may call external tools or SDK-provided utilities.
4. Agents return results via a result queue or update orchestrator via REST callback.
5. Orchestrator records traces (OpenTelemetry) and emits Prometheus metrics for latency, success rate, and throughput.

Deployment patterns
- Local development: docker-compose with RabbitMQ and optional monitoring.
- Production: Kubernetes with Deployments/StatefulSets for agents, HorizontalPodAutoscaler for scaling, and a dedicated RL training cluster.
- Security: Use mTLS for inter-service communication where possible, store secrets in Kubernetes Secrets or Vault, enforce RBAC for API endpoints.

Operational notes
- Use separate queues per agent type to avoid noisy neighbors.
- Instrument agents to include trace context for distributed tracing.
- Snapshot RL policy versions and maintain a model registry for reproducibility.
