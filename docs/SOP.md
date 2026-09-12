# Standard Operating Procedure: Sovereign Godot Pipeline

## 1. Service Health Verification
Run `sovereign-godot-pipeline --health` to confirm the engine responds with status `HEALTHY`.

## 2. Webhook Adapter Operation
The webhook adapter listens on port `8786`:
```bash
python3 n8n/webhook_adapter.py
```
If port 8786 is occupied, check active processes:
```bash
lsof -i :8786
```

## 3. n8n Integration Testing
Send a probe POST request:
```bash
curl -X POST http://localhost:8786/api/v1/execute \
  -H "Content-Type: application/json" \
  -d '{"action": "health_ping", "payload": {"test": true}}'
```
