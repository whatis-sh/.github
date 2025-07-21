# whAtIs.sh

A modern implementation of the Unix `whatis` command using Large Language Models (LLMs). This project provides both an LLM model configuration and a REST API service that mimics the behavior of the traditional `whatis` command.

## Project Structure

```
whatis.sh/
├── llm/                    # LLM model configuration
│   ├── Modelfile          # Ollama model definition
│   └── README.md          # LLM setup instructions
├── api/                   # REST API service
│   ├── main.py           # FastAPI application
│   ├── requirements.txt  # Python dependencies
│   ├── start.sh         # Quick start script
│   ├── .gitignore       # Git ignore patterns
│   ├── ecs/             # Container deployment files
│   │   ├── Dockerfile   # Container image definition
│   │   ├── docker-compose.yml # Full stack with LLM service
│   │   └── ecs-task-definition.json # AWS ECS configuration
│   ├── serverless/      # Serverless deployment files
│   │   ├── lambda_handler.py # AWS Lambda adapter
│   │   └── serverless.yml # Serverless Framework config
│   ├── tests/           # Test files
│   │   └── test_api.py  # API test suite
│   └── README.md        # API documentation
└── README.md            # This file
```

## Quick Start

### 1. Set up the LLM Model

The LLM component uses Ollama to serve a specialized model trained to behave like the Unix `whatis` command.

```bash
# Navigate to llm directory
cd llm/

# Create the model using Ollama
ollama create whatis.sh -f Modelfile

# Start Ollama server (if not already running)
ollama serve
```

### 2. Start the API Service

The API provides HTTP endpoints that accept commands and return `whatis`-style responses.

```bash
# Navigate to api directory
cd api/

# Quick start (installs dependencies and starts server)
./start.sh

# Or manually:
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

### 3. Test the Service

```bash
# Health check
curl http://whatis.sh:2095/health

# Get usage instructions
curl http://whatis.sh:2095/

# Query a command (headless style)
curl http://whatis.sh:2095/ls
curl http://whatis.sh:2095/grep?v=true

# Query with JSON body
curl -X POST http://whatis.sh:2095/ \
  -H "Content-Type: application/json" \
  -d '{"cmd_or_func": "awk", "verbose": true}'
```

## Components

### LLM Model (`/llm`)

- **Purpose**: Provides the AI backend that generates `whatis`-style responses
- **Technology**: Ollama with llama3.2:3b base model
- **Features**:
  - Trained to respond like Unix `whatis` command
  - Supports verbose mode with `-v` flag
  - Provides command origins (bash, Python, etc.)
  - Web search capability for detailed documentation

### API Service (`/api`)

- **Purpose**: HTTP REST API that serves the LLM functionality
- **Technology**: FastAPI with Python 3.13+
- **Features**:
  - Multiple endpoint types (headless, JSON body, POST)
  - Health checks for monitoring
  - Container and serverless deployment ready
  - Comprehensive error handling and logging

## API Endpoints

| Method | Endpoint | Description | Example |
|--------|----------|-------------|---------|
| GET | `/` | Usage instructions or JSON body processing | `curl http://whatis.sh:2095/` |
| GET | `/{command}` | Headless command query | `curl http://whatis.sh:2095/ls` |
| GET | `/{command}?v=true` | Verbose headless query | `curl http://whatis.sh:2095/grep?v=true` |
| POST | `/` | JSON body command query | See examples below |
| GET | `/health` | Service health check | `curl http://whatis.sh:2095/health` |

### Example Requests

```bash
# Basic command query
curl http://whatis.sh:2095/ls

# Verbose command query  
curl http://whatis.sh:2095/sed?v=true

# JSON POST request
curl -X POST http://whatis.sh:2095/ \
  -H "Content-Type: application/json" \
  -d '{"cmd_or_func": "grep", "verbose": false}'

# JSON GET request (with body)
curl -X GET http://whatis.sh:2095/ \
  -H "Content-Type: application/json" \
  -d '{"cmd_or_func": "awk", "verbose": true}'
```

## Deployment Options

### Local Development
```bash
cd api/
./start.sh
```

### Docker Container
```bash
cd api/ecs/
docker build -t whatis-api .
docker run -p 8000:8000 whatis-api
```

### Docker Compose (Full Stack)
```bash
cd api/ecs/
docker-compose up
```

### AWS Lambda (Serverless)
```bash
cd api/serverless/
serverless deploy
```

### AWS ECS
Use the provided `ecs-task-definition.json` in the `api/ecs/` directory for container deployment.

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `LLM_BASE_URL` | URL of the LLM service | `http://localhost:11434` |
| `LLM_MODEL` | Model name to use | `whatis.sh` |

## Development

### Testing
```bash
cd api/
python -m pytest tests/test_api.py -v
```

### Dependencies
- **LLM**: Ollama, llama3.2:3b model
- **API**: FastAPI, uvicorn, httpx, pydantic
- **Optional**: Docker, AWS CLI, Serverless Framework

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

This project is open source. See individual component directories for specific details.

## Support

- **Issues**: Report bugs and feature requests via GitHub Issues
- **Documentation**: See `/api/README.md` and `/llm/README.md` for detailed component docs
- **Health Check**: Use `GET /health` endpoint to verify service status
