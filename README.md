# Docker Kit

My collection of Docker Compose files and configurations for various applications and services I use, either for personal projects or for learning purposes. This repository focuses on production-ready, well-documented containerized solutions.

## Structure

```
docker-kit/
├── README.md                    # This file
└── compose-files/              # Docker Compose configurations
    ├── caddy/                   # Caddy web server configuration
    │   ├── docker-compose.yaml   # Caddy service definition
    │   ├── Caddyfile             # Caddy configuration file
    │   └── README.md             # Caddy documentation
    └── ...
```

## Quick Start

### General Usage

1. **Clone the repository**:
   ```bash
   git clone https://github.com/glazk0/docker-kit.git
   cd docker-kit
   ```

2. **Navigate to the desired service**:
   ```bash
   cd compose-files/caddy  # Example with Caddy
   ```

3. **Configure environment variables**:
   ```bash
   cp .env.example .env    # Copy example environment file
   nano .env               # Edit with your values
   ```

4. **Ensure Docker is installed**:
   ```bash
   docker --version
   docker compose --version  # Note: Modern Docker uses 'docker compose'
   ```

5. **Start the service**:
   ```bash
   docker compose up -d
   ```

### Service-Specific Setup

Each service directory contains its own detailed README with:
- ✅ Prerequisites and requirements
- ⚙️ Configuration options
- 🚀 Deployment instructions
- 🔧 Troubleshooting guides
- 📊 Monitoring and maintenance

Visit the service's README for complete setup instructions.

## Features

- **Production Ready**: All configurations are tested and suitable for production use
- **Security Focused**: Implements security best practices and hardening
- **Well Documented**: Each service includes comprehensive documentation
- **Environment Driven**: Flexible configuration through environment variables
- **Network Isolation**: Proper Docker networking for service communication
- **Persistent Storage**: Appropriate volume configurations for data persistence
- **Certificate Management**: Support for existing VPS certificates or automatic Let's Encrypt
- **Git Security**: Comprehensive `.gitignore` prevents accidental commit of sensitive data

## Security & Best Practices

- **Environment Protection**: `.env` files are automatically ignored by Git
- **Certificate Security**: SSL certificates and private keys are never committed
- **Read-only Mounts**: Sensitive files mounted read-only in containers
- **Proper Permissions**: Documentation includes proper file permission guidelines
- **Secrets Management**: Clear separation of public configuration and sensitive data

## Prerequisites

- Docker (version 20.10 or later)
- Docker Compose (version 2.0 or later)
- Basic understanding of Docker networking and volumes

## Network Architecture

Services are designed to work together using Docker networks:
- External networks for inter-service communication
- Internal networks for service isolation
- Proper port exposure for external access

## Contributing

Contributions are welcome! When adding new services:

1. Create a new directory under `compose-files/`
2. Include a complete `docker-compose.yaml`
3. Add `.env.example` with required variables
4. Write comprehensive documentation in `README.md`
5. Follow existing patterns for consistency
6. Test thoroughly before submitting

### Contribution Guidelines

- Use official Docker images when available
- Implement security best practices
- Include health checks where applicable
- Document all configuration options
- Provide troubleshooting information

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

