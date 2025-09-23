# Acquisitions - Dockerized Node.js Application with Neon Database

This repository contains a Node.js Express application that uses Neon Database with different setups for development and production environments.

## 🏗️ Architecture Overview

- **Development**: Uses Neon Local proxy for local database development with ephemeral branches
- **Production**: Connects directly to Neon Cloud Database
- **Framework**: Express.js with Drizzle ORM
- **Database**: PostgreSQL via Neon Database
- **Containerization**: Docker with multi-stage builds

## 📋 Prerequisites

Before you begin, ensure you have the following installed:

- [Docker](https://docs.docker.com/get-docker/) (version 20.10+)
- [Docker Compose](https://docs.docker.com/compose/install/) (version 2.0+)
- [Neon Account](https://neon.tech) with a project set up
- [Neon API Key](https://neon.com/docs/manage/api-keys)

## 🛠️ Environment Setup

### 1. Neon Database Configuration

1. **Get your Neon credentials:**
   - Visit [Neon Console](https://console.neon.tech)
   - Navigate to your project
   - Get your `Project ID` from Project Settings → General
   - Create an API key from Account Settings → Developer Settings → API Keys

2. **Create environment files:**

Create a `.env.local` file in your project root (this file is ignored by git):

```bash
# .env.local
NEON_API_KEY=your_neon_api_key_here
NEON_PROJECT_ID=your_neon_project_id_here
PARENT_BRANCH_ID=br_your_main_branch_id_here
ARCJET_KEY=your_arcjet_key_here

# Optional: Set to false if you want to persist branches across container restarts
DELETE_BRANCH=true
```

## 🚀 Development Environment

The development environment uses **Neon Local**, which creates a local proxy to your Neon database and can automatically create ephemeral branches.

### Starting Development Environment

1. **Start all services:**
   ```bash
   docker-compose -f docker-compose.dev.yml up -d
   ```

2. **Start with logs:**
   ```bash
   docker-compose -f docker-compose.dev.yml up
   ```

3. **Start with Drizzle Studio (optional):**
   ```bash
   docker-compose -f docker-compose.dev.yml --profile studio up -d
   ```

### Development Features

- **Hot Reload**: Source code changes are automatically reflected
- **Ephemeral Branches**: Each container startup creates a fresh database branch
- **Database Management**: Optional Drizzle Studio at http://localhost:4983
- **Neon Local Proxy**: Accessible at `localhost:5432`
- **Application**: Accessible at http://localhost:3000

### Development Commands

```bash
# View logs
docker-compose -f docker-compose.dev.yml logs -f

# Stop services
docker-compose -f docker-compose.dev.yml down

# Rebuild and restart
docker-compose -f docker-compose.dev.yml up -d --build

# Run database migrations
docker-compose -f docker-compose.dev.yml exec app npm run db:migrate

# Access application container
docker-compose -f docker-compose.dev.yml exec app sh
```

### Development Database Connection

Your application connects to:
```
postgres://neon:npg@neon-local:5432/neondb?sslmode=require
```

This connection string:
- Points to the Neon Local proxy container
- Uses the standard Neon credentials (`neon:npg`)
- Connects via SSL (required)
- Uses your default database name

## 🌐 Production Environment

The production environment connects directly to your Neon Cloud database without any local proxy.

### Production Setup

1. **Set environment variables** (on your deployment platform):
   ```bash
   export DATABASE_URL="postgresql://username:password@ep-xyz-pooler.region.aws.neon.tech/dbname?sslmode=require"
   export ARCJET_KEY="your_arcjet_key"
   export LOG_LEVEL="info"
   ```

2. **Deploy with Docker Compose:**
   ```bash
   docker-compose -f docker-compose.prod.yml up -d
   ```

3. **With additional services (nginx, redis):**
   ```bash
   docker-compose -f docker-compose.prod.yml --profile nginx --profile redis up -d
   ```

### Production Features

- **Security Hardened**: Non-root user, read-only filesystem, security options
- **Resource Limits**: CPU and memory constraints
- **Health Checks**: Automatic health monitoring
- **Logging**: Structured production logging
- **Optional Services**: Nginx reverse proxy, Redis cache, log aggregation

### Production Services

| Service | Port | Description |
|---------|------|-------------|
| App | 3000 | Main application |
| Nginx | 80, 443 | Reverse proxy (optional) |
| Redis | 6379 | Cache/sessions (optional) |
| Fluentd | 24224 | Log aggregation (optional) |

### Production Commands

```bash
# Deploy production
docker-compose -f docker-compose.prod.yml up -d

# Scale application
docker-compose -f docker-compose.prod.yml up -d --scale app=3

# View logs
docker-compose -f docker-compose.prod.yml logs -f app

# Health check
curl http://localhost:3000/health

# Stop production
docker-compose -f docker-compose.prod.yml down
```

## 🔧 Configuration Details

### Environment Variables

| Variable | Development | Production | Description |
|----------|-------------|------------|-------------|
| `NODE_ENV` | `development` | `production` | Application environment |
| `DATABASE_URL` | Neon Local | Neon Cloud | Database connection string |
| `PORT` | `3000` | `3000` | Application port |
| `LOG_LEVEL` | `debug` | `info` | Logging level |
| `NEON_API_KEY` | Required | Not needed | Neon API key for local proxy |
| `NEON_PROJECT_ID` | Required | Not needed | Neon project ID |
| `PARENT_BRANCH_ID` | Optional | Not needed | Branch for ephemeral branches |

### Database Connections

#### Development (Neon Local)
```javascript
// Standard Postgres driver
const connectionString = "postgres://neon:npg@neon-local:5432/neondb?sslmode=require";

// For JavaScript applications, you might need:
const config = {
  connectionString,
  ssl: {
    rejectUnauthorized: false
  }
};
```

#### Production (Neon Cloud)
```javascript
// Direct Neon Cloud connection
const connectionString = process.env.DATABASE_URL;
// Example: "postgresql://user:pass@ep-xyz-pooler.region.aws.neon.tech/db?sslmode=require"
```

## 📁 Project Structure

```
acquisitions/
├── src/                          # Application source code
├── drizzle/                      # Database migrations
├── logs/                         # Application logs (gitignored)
├── .neon_local/                  # Neon Local metadata (gitignored)
├── Dockerfile                    # Multi-stage Docker build
├── docker-compose.dev.yml        # Development environment
├── docker-compose.prod.yml       # Production environment
├── .env.development              # Development env template
├── .env.production               # Production env template
├── .env.local                    # Local secrets (gitignored)
├── .dockerignore                 # Docker ignore rules
├── .gitignore                    # Git ignore rules
├── drizzle.config.js             # Drizzle ORM configuration
└── package.json                  # Node.js dependencies
```

## 🚨 Troubleshooting

### Common Issues

1. **Neon Local container fails to start:**
   ```bash
   # Check if environment variables are set
   echo $NEON_API_KEY
   echo $NEON_PROJECT_ID
   
   # View container logs
   docker-compose -f docker-compose.dev.yml logs neon-local
   ```

2. **Application can't connect to database:**
   ```bash
   # Check network connectivity
   docker-compose -f docker-compose.dev.yml exec app ping neon-local
   
   # Verify database is ready
   docker-compose -f docker-compose.dev.yml exec neon-local pg_isready -h localhost -p 5432
   ```

3. **Permission issues in production:**
   ```bash
   # Check if running as non-root user
   docker-compose -f docker-compose.prod.yml exec app whoami
   
   # Check file permissions
   docker-compose -f docker-compose.prod.yml exec app ls -la
   ```

### Development Tips

- **Fresh database branch**: Stop and start containers to get a new ephemeral branch
- **Persist branches**: Set `DELETE_BRANCH=false` in your `.env.local`
- **Debug mode**: Check application logs for detailed debugging information
- **Database inspection**: Use Drizzle Studio to inspect your database schema and data

### Production Deployment

For production deployment, consider:

1. **Container Registry**: Push your built image to a registry (Docker Hub, AWS ECR, etc.)
2. **Orchestration**: Use Docker Swarm, Kubernetes, or managed container services
3. **Load Balancing**: Configure nginx or external load balancers
4. **Monitoring**: Set up application and infrastructure monitoring
5. **Backup Strategy**: Configure database backups in Neon Console
6. **SSL/TLS**: Configure HTTPS certificates for production domains

## 📚 Additional Resources

- [Neon Documentation](https://neon.com/docs)
- [Neon Local Documentation](https://neon.com/docs/local/neon-local)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Drizzle ORM Documentation](https://orm.drizzle.team/)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test in development environment
5. Submit a pull request

## 📄 License

This project is licensed under the ISC License - see the package.json file for details.

---

**Happy coding!** 🚀

For questions or support, please check the troubleshooting section above or create an issue in this repository.