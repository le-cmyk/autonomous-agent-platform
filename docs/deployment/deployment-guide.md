# Deployment Guide

## Overview

This guide provides instructions for deploying the Autonomous Agent Platform in various environments. The platform is designed to be deployed using Docker and Kubernetes for production environments, with simpler options for development and testing.

## Prerequisites

- Docker and Docker Compose
- Kubernetes cluster (for production deployments)
- PostgreSQL database (v14+)
- Redis (v6+)
- InfluxDB (v2.0+)
- Node.js v18+ (for development environments)
- npm v8+ (for development environments)

## Deployment Options

### 1. Docker Compose (Development & Small Deployments)

#### Setup

1. Clone the repository:
   ```bash
   git clone https://github.com/organization/autonomous-agent-platform.git
   cd autonomous-agent-platform
   ```

2. Create environment file:
   ```bash
   cp .env.example .env
   ```

3. Configure environment variables in `.env`:
   ```
   # Database Configuration
   POSTGRES_USER=postgres
   POSTGRES_PASSWORD=your_secure_password
   POSTGRES_DB=agent_platform
   
   # Redis Configuration
   REDIS_URL=redis://redis:6379
   
   # InfluxDB Configuration
   INFLUXDB_URL=http://influxdb:8086
   INFLUXDB_TOKEN=your_influxdb_token
   INFLUXDB_ORG=your_org
   INFLUXDB_BUCKET=agent_metrics
   
   # Authentication
   JWT_SECRET=generate_a_secure_random_string
   JWT_EXPIRATION=3600
   
   # API Configuration
   API_PORT=3000
   API_BASE_URL=http://localhost:3000
   
   # Frontend Configuration
   FRONTEND_URL=http://localhost:8080
   ```

4. Start the services:
   ```bash
   docker-compose up -d
   ```

5. Initialize the database:
   ```bash
   docker-compose exec backend npm run migrate:up
   docker-compose exec backend npm run seed:dev
   ```

6. Access the application:
   - Frontend: http://localhost:8080
   - API: http://localhost:3000
   - API Documentation: http://localhost:3000/docs

### 2. Kubernetes (Production)

#### Infrastructure Setup

1. Create Kubernetes namespace:
   ```bash
   kubectl create namespace agent-platform
   ```

2. Create ConfigMaps and Secrets:
   ```bash
   # Create ConfigMap for non-sensitive configuration
   kubectl create configmap agent-platform-config \
     --from-literal=API_PORT=3000 \
     --from-literal=INFLUXDB_ORG=agent_platform \
     --from-literal=INFLUXDB_BUCKET=agent_metrics \
     --namespace agent-platform
   
   # Create Secrets for sensitive configuration
   kubectl create secret generic agent-platform-secrets \
     --from-literal=POSTGRES_USER=postgres \
     --from-literal=POSTGRES_PASSWORD=your_secure_password \
     --from-literal=POSTGRES_DB=agent_platform \
     --from-literal=REDIS_URL=redis://redis-service:6379 \
     --from-literal=INFLUXDB_URL=http://influxdb-service:8086 \
     --from-literal=INFLUXDB_TOKEN=your_influxdb_token \
     --from-literal=JWT_SECRET=your_secure_jwt_secret \
     --namespace agent-platform
   ```

3. Apply Kubernetes manifests:
   ```bash
   kubectl apply -f k8s/database/
   kubectl apply -f k8s/redis/
   kubectl apply -f k8s/influxdb/
   kubectl apply -f k8s/backend/
   kubectl apply -f k8s/frontend/
   kubectl apply -f k8s/ingress/
   ```

4. Run database migrations:
   ```bash
   kubectl exec -it $(kubectl get pods -l app=agent-platform-backend -n agent-platform -o jsonpath='{.items[0].metadata.name}') -n agent-platform -- npm run migrate:up
   ```

5. Access the application:
   - Get the external IP or domain configured in your ingress:
     ```bash
     kubectl get ingress -n agent-platform
     ```

### 3. AWS Deployment

#### Using AWS ECS + RDS + ElastiCache

1. Create AWS resources:
   - RDS PostgreSQL instance
   - ElastiCache Redis cluster
   - ECR repositories for images
   - ECS cluster
   - Application Load Balancer
   - VPC with public and private subnets

2. Build and push Docker images to ECR:
   ```bash
   aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com
   
   # Build and push backend
   docker build -t $AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/agent-platform-backend:latest ./backend
   docker push $AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/agent-platform-backend:latest
   
   # Build and push frontend
   docker build -t $AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/agent-platform-frontend:latest ./frontend
   docker push $AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/agent-platform-frontend:latest
   ```

3. Create task definitions and services:
   ```bash
   aws ecs register-task-definition --cli-input-json file://aws/task-definitions/backend.json
   aws ecs register-task-definition --cli-input-json file://aws/task-definitions/frontend.json
   
   aws ecs create-service --cli-input-json file://aws/services/backend-service.json
   aws ecs create-service --cli-input-json file://aws/services/frontend-service.json
   ```

4. Run database migrations:
   ```bash
   aws ecs run-task --cluster agent-platform --task-definition agent-platform-migrations --network-configuration "awsvpcConfiguration={subnets=[$PRIVATE_SUBNET],securityGroups=[$SECURITY_GROUP],assignPublicIp=DISABLED}"
   ```

5. Access the application via the Application Load Balancer URL

## Database Migration

### Initial Setup

1. Run the initial migration:
   ```bash
   # Development
   npm run migrate:up
   
   # Docker
   docker-compose exec backend npm run migrate:up
   
   # Kubernetes
   kubectl exec -it $(kubectl get pods -l app=agent-platform-backend -n agent-platform -o jsonpath='{.items[0].metadata.name}') -n agent-platform -- npm run migrate:up
   ```

2. Seed the database with initial data:
   ```bash
   # Development
   npm run seed:dev
   
   # Docker
   docker-compose exec backend npm run seed:dev
   
   # Kubernetes
   kubectl exec -it $(kubectl get pods -l app=agent-platform-backend -n agent-platform -o jsonpath='{.items[0].metadata.name}') -n agent-platform -- npm run seed:dev
   ```

### Schema Updates

When updating the database schema:

1. Create a new migration:
   ```bash
   # Development
   npm run migrate:create my_migration_name
   
   # Docker
   docker-compose exec backend npm run migrate:create my_migration_name
   ```

2. Edit the generated migration file in `migrations/` directory

3. Apply the migration:
   ```bash
   # Development
   npm run migrate:up
   
   # Docker
   docker-compose exec backend npm run migrate:up
   
   # Kubernetes
   kubectl exec -it $(kubectl get pods -l app=agent-platform-backend -n agent-platform -o jsonpath='{.items[0].metadata.name}') -n agent-platform -- npm run migrate:up
   ```

## Scaling Considerations

### Horizontal Scaling

The platform is designed to scale horizontally:

1. **Backend API Services**:
   - Can be scaled by increasing container replicas
   - Use load balancing to distribute traffic

2. **Frontend**:
   - Stateless and can be scaled with multiple replicas
   - Consider using a CDN for static assets

3. **Database**:
   - Consider read replicas for scaling read operations
   - Implement connection pooling
   - Shard data for very large deployments

4. **Redis**:
   - Set up Redis Cluster for distributed caching
   - Configure appropriate memory limits

### Vertical Scaling

For smaller deployments, vertical scaling can be considered:

1. **Backend Services**:
   - Increase CPU and memory resources

2. **Database**:
   - Upgrade to larger instance types
   - Optimize for available memory

## Monitoring Setup

1. **Prometheus & Grafana**:
   ```bash
   kubectl apply -f k8s/monitoring/prometheus.yaml
   kubectl apply -f k8s/monitoring/grafana.yaml
   ```

2. **Access Grafana**:
   ```bash
   kubectl port-forward svc/grafana 3000:3000 -n agent-platform
   ```
   
   Import pre-built dashboards from `monitoring/dashboards/` directory.

## Backup and Disaster Recovery

### Database Backups

1. **PostgreSQL**:
   ```bash
   # Manual backup
   pg_dump -h <host> -U <user> -d agent_platform -F c -f backup.dump
   
   # Automated backups (AWS RDS)
   # Configure through AWS Console or CLI
   ```

2. **Redis**:
   ```bash
   # Configure persistence in redis.conf
   # For Redis in Kubernetes
   kubectl apply -f k8s/redis/redis-persistence.yaml
   ```

3. **InfluxDB**:
   ```bash
   # Manual backup
   influx backup -t <token> -o <org> <path/to/backup>
   ```

### Restore Procedures

1. **PostgreSQL**:
   ```bash
   pg_restore -h <host> -U <user> -d agent_platform -c backup.dump
   ```

2. **Redis**:
   - Redis will automatically restore from the RDB and AOF files on restart

3. **InfluxDB**:
   ```bash
   influx restore -t <token> <path/to/backup>
   ```

## Security Considerations

1. **API Security**:
   - JWT tokens are used for authentication
   - HTTPS is required in production
   - API keys are securely stored and transmitted

2. **Container Security**:
   - Use non-root users in containers
   - Scan images for vulnerabilities
   - Use read-only file systems where possible

3. **Network Security**:
   - Implement proper network policies in Kubernetes
   - Use security groups in AWS to restrict access
   - Only expose necessary services to the internet

4. **Secrets Management**:
   - Use Kubernetes Secrets or AWS Secrets Manager
   - Never store secrets in code or images
   - Rotate secrets regularly

## Troubleshooting

### Common Issues

1. **Database Connection Issues**:
   ```bash
   # Check database connectivity
   kubectl exec -it <pod-name> -n agent-platform -- nc -zv <db-host> 5432
   ```

2. **Redis Connection Issues**:
   ```bash
   # Check Redis connectivity
   kubectl exec -it <pod-name> -n agent-platform -- nc -zv <redis-host> 6379
   ```

3. **API Errors**:
   ```bash
   # Check API logs
   kubectl logs <api-pod-name> -n agent-platform
   ```

4. **Frontend Issues**:
   ```bash
   # Check for JavaScript console errors
   kubectl logs <frontend-pod-name> -n agent-platform
   ```

### Log Collection

Configure centralized logging using EFK (Elasticsearch, Fluentd, Kibana) stack:

```bash
kubectl apply -f k8s/logging/
```

Access Kibana dashboard:

```bash
kubectl port-forward svc/kibana 5601:5601 -n logging
```
