# Podman Migration Guide

This repository has been migrated to be fully compatible with Podman Compose.

## Major Changes

### 1. **Replaced Ollama with vLLM**
- **Removed**: Ollama services (ollama-cpu, ollama-gpu, ollama-gpu-amd)
- **Added**: vLLM services with CPU and GPU profiles
  - `vllm-cpu`: CPU-only inference service
  - `vllm-gpu`: GPU-accelerated inference service with NVIDIA support
- **Default Model**: `facebook/opt-125m` (lightweight model for testing)
- **API Endpoint**: http://vllm:8000 (OpenAI-compatible API)

### 2. **Volume Mounts → Folder Paths**
All named volumes have been replaced with local folder paths:
```
./data/n8n          → n8n storage
./data/postgres     → PostgreSQL database
./data/pgadmin      → pgAdmin configuration
./data/qdrant       → Qdrant vector database
./data/openwebui    → Open WebUI data
./data/vllm         → vLLM model cache
./data/jira         → Jira data
```

### 3. **Environment Variables Inline**
All environment variables are now directly in `docker-compose.yml`:
- **Database credentials**: root/password
- **n8n secrets**: Embedded in service definition
- **pgAdmin credentials**: admin@admin.com/password
- **Jira settings**: Placeholder values (update as needed)

### 4. **Added pgAdmin Service**
New database management interface:
- **URL**: http://localhost:5050
- **Login**: admin@admin.com / password
- **Pre-configured**: PostgreSQL connection with credentials
- **Configuration**:
  - `./pgadmin/servers.json`: Server definitions
  - `./pgadmin/pgpassfile`: Saved passwords

### 5. **Custom n8n Dockerfile**
n8n now builds from `Dockerfile.n8n` with additional capabilities:
- Python 3 runtime
- Git support
- Python libraries:
  - `tree-sitter` & `tree-sitter-languages`
  - `transformers`
  - `nomic[local]`

### 6. **Updated Images**
All services use the latest suitable images:
- **PostgreSQL**: `postgres:17-alpine` (latest stable)
- **pgAdmin**: `dpage/pgadmin4:latest`
- **n8n**: Built from `n8nio/n8n:latest` + Python
- **vLLM**: `vllm/vllm-openai:latest`
- **Qdrant**: `qdrant/qdrant:latest`
- **Open WebUI**: `ghcr.io/open-webui/open-webui:main`
- **Alpine**: `alpine:3.20`
- **Jira**: `cptactionhank/atlassian-jira:latest`
- **MCP Server**: `ghcr.io/sooperset/mcp-atlassian:latest`

## Running with Podman

### Start Services (CPU Profile)
```bash
podman-compose --profile cpu up -d
```

### Start Services (GPU Profile)
```bash
podman-compose --profile gpu up -d
```

### Build n8n Image
```bash
podman-compose build n8n
```

### View Logs
```bash
podman-compose logs -f [service-name]
```

### Stop Services
```bash
podman-compose down
```

## Service Endpoints

| Service | URL | Default Credentials |
|---------|-----|---------------------|
| n8n | http://localhost:5678 | (Configure on first run) |
| pgAdmin | http://localhost:5050 | admin@admin.com / password |
| Open WebUI | http://localhost:3000 | (From imported DB) |
| Qdrant | http://localhost:6333 | No auth |
| vLLM API | http://localhost:8000 | No auth |
| Jira | http://localhost:8080 | (Configure on first run) |
| MCP Server | http://localhost:5000 | No auth |

## vLLM Configuration

### Changing the Model
Edit the `command` section in docker-compose.yml:
```yaml
command:
  - "--model"
  - "your-model-name"  # e.g., "meta-llama/Llama-2-7b-hf"
  - "--host"
  - "0.0.0.0"
  - "--port"
  - "8000"
```

### Using Hugging Face Models
Set your token in the environment or docker-compose.yml:
```yaml
environment:
  - HUGGING_FACE_HUB_TOKEN=your_token_here
```

### GPU Memory Configuration
For GPU profile, adjust shared memory:
```yaml
shm_size: 8gb  # Increase for larger models
```

## pgAdmin Access

1. Navigate to http://localhost:5050
2. Login with `admin@admin.com` / `password`
3. PostgreSQL server is pre-configured:
   - **Host**: postgres
   - **Port**: 5432
   - **Database**: n8n
   - **Username**: root
   - **Password**: Saved automatically

## Migration Notes

### From Docker to Podman
1. Install podman-compose: `pip install podman-compose`
2. Ensure Podman is running: `podman info`
3. For GPU support, configure nvidia-container-toolkit for Podman

### Data Persistence
All data is stored in `./data/` directories:
- **Backup**: Simply copy the `data/` folder
- **Reset**: Delete `data/` folder to start fresh
- **Git**: `data/` folder is ignored in `.gitignore`

### Environment Variables
The `.env` file is no longer required but kept for backward compatibility.
Update credentials directly in `docker-compose.yml` instead.

## Troubleshooting

### Port Conflicts
If ports are in use, edit the port mappings in docker-compose.yml:
```yaml
ports:
  - "NEW_PORT:CONTAINER_PORT"
```

### Permission Issues
Ensure data directories have correct permissions:
```bash
chmod -R 755 data/
```

### vLLM Out of Memory
- **CPU**: Reduce model size or increase swap
- **GPU**: Use smaller model or increase `shm_size`

### pgAdmin Connection Failed
Check PostgreSQL is healthy:
```bash
podman-compose ps postgres
```

Wait for health check to pass before accessing pgAdmin.

## Security Considerations

⚠️ **Default credentials are insecure!** Update these in production:
- PostgreSQL: `POSTGRES_USER` and `POSTGRES_PASSWORD`
- pgAdmin: `PGADMIN_DEFAULT_EMAIL` and `PGADMIN_DEFAULT_PASSWORD`
- n8n: `N8N_ENCRYPTION_KEY` and `N8N_USER_MANAGEMENT_JWT_SECRET`
- Jira: Configure admin credentials on first run

## Additional Resources

- [Podman Documentation](https://docs.podman.io/)
- [vLLM Documentation](https://docs.vllm.ai/)
- [n8n Documentation](https://docs.n8n.io/)
- [pgAdmin Documentation](https://www.pgadmin.org/docs/)
