# Agents & Tools — How to add an Agent or Tool

This guide shows how to add a simple agent and integrate a new tool into the framework.

Agent responsibilities
- Connect to RabbitMQ and subscribe to a specific queue.
- Deserialize incoming tasks and run assigned tools.
- Emit result messages and metrics (latency, success/failure).
- Respect timeouts and provide graceful shutdown.

Add a new tool (Python SDK example)
1. Create a module under `tools/` e.g. `tools/my_tool.py`:

```python
from framework.sdk import BaseTool

class MyTool(BaseTool):
    def run(self, input_data):
        # Implement tool logic here
        return {"result": "processed: " + str(input_data)}
```

2. Register tool in config.yaml under `tools` list:
```yaml
tools:
  - name: my_tool
    module: tools.my_tool:MyTool
```

3. Use tool from an agent:
```python
from framework.sdk import ToolLoader

tool = ToolLoader.load('my_tool', config)
result = tool.run({"payload": "hello"})
```

Create a simple agent
1. Minimal agent skeleton:
```python
import pika
import json
from framework.sdk import ToolLoader

def on_message(ch, method, properties, body):
    task = json.loads(body)
    tool = ToolLoader.load(task['tool'])
    result = tool.run(task['payload'])
    # send result or ack
    ch.basic_ack(delivery_tag=method.delivery_tag)

def main():
    # connect to RabbitMQ
    connection = pika.BlockingConnection(...)
    channel = connection.channel()
    channel.basic_consume(queue='tasks.my_tool', on_message_callback=on_message)
    channel.start_consuming()

if __name__ == '__main__':
    main()
```

Testing agents locally
- Start RabbitMQ: `docker run -d --name ai-rabbit -p 5672:5672 -p 15672:15672 rabbitmq:3-management`
- Start your agent module and use `curl` to submit a task to the orchestrator.

Metrics & Tracing
- Agents should use OpenTelemetry instrumentation for traces and expose Prometheus metrics for scraping.
- Exporters and Prometheus port configuration live in `config.yaml`.

Security recommendations
- Validate and sanitize input to tools to avoid command injection.
- Run untrusted code in sandboxed processes or containers.
