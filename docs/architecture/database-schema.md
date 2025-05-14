# Database Schema Design

## PostgreSQL Tables

### Users and Authentication

```sql
-- Users and Authentication
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  name VARCHAR(255),
  role VARCHAR(50) NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Organizations (for multi-tenancy)
CREATE TABLE organizations (
  id UUID PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Organization Memberships
CREATE TABLE org_memberships (
  user_id UUID REFERENCES users(id),
  org_id UUID REFERENCES organizations(id),
  role VARCHAR(50) NOT NULL,
  PRIMARY KEY (user_id, org_id)
);
```

### Agent Management

```sql
-- Agent Types (predefined agent templates)
CREATE TABLE agent_types (
  id UUID PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  default_config JSONB,
  required_params JSONB,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Agent Instances
CREATE TABLE agents (
  id UUID PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  type_id UUID REFERENCES agent_types(id),
  org_id UUID REFERENCES organizations(id),
  created_by UUID REFERENCES users(id),
  api_key VARCHAR(255) UNIQUE,
  webhook_url VARCHAR(255),
  config JSONB NOT NULL,
  schedule VARCHAR(255),  -- Cron expression
  status VARCHAR(50) DEFAULT 'inactive',
  last_heartbeat TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### Monitoring and Alerts

```sql
-- Agent Execution Logs
CREATE TABLE agent_logs (
  id UUID PRIMARY KEY,
  agent_id UUID REFERENCES agents(id),
  status VARCHAR(50) NOT NULL,  -- success, error, warning
  message TEXT,
  details JSONB,
  started_at TIMESTAMP WITH TIME ZONE,
  completed_at TIMESTAMP WITH TIME ZONE,
  duration INTEGER  -- in milliseconds
);

-- Alert Configurations
CREATE TABLE alert_configs (
  id UUID PRIMARY KEY,
  agent_id UUID REFERENCES agents(id),
  name VARCHAR(255) NOT NULL,
  conditions JSONB NOT NULL,  -- e.g. {"status": "error", "consecutive_failures": 3}
  channels JSONB NOT NULL,    -- e.g. {"email": "user@example.com", "slack": "#alerts"}
  enabled BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Alert History
CREATE TABLE alerts (
  id UUID PRIMARY KEY,
  config_id UUID REFERENCES alert_configs(id),
  agent_id UUID REFERENCES agents(id),
  triggered_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  resolved_at TIMESTAMP WITH TIME ZONE,
  status VARCHAR(50) DEFAULT 'open',
  details JSONB
);
```

## Redis Data Structures

```
# Agent Status Cache
agent:{id}:status = "online"|"offline"|"error"

# Agent Metrics (recent)
agent:{id}:metrics = {cpu, memory, last_execution_time}

# Agent Execution Queue
queue:scheduled_jobs = [job1, job2, ...]

# Rate Limiting
ratelimit:api:{api_key} = count

# User Sessions
session:{session_id} = {user_data}
```

## InfluxDB (Time-Series Data)

Used to store historical performance metrics for agents:
- Execution duration
- Memory usage
- Execution status over time
- Custom metrics reported by agents

## Entity Relationship Diagram

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│    Users    │     │Organizations│     │ AgentTypes  │
├─────────────┤     ├─────────────┤     ├─────────────┤
│ id          │┐    │ id          │     │ id          │
│ email       │├────┼─OrgMembership      │ name        │
│ password    ││    │ name        │     │ description │
│ name        ││    └─────────────┘     │ default_cfg │
└─────────────┘│                        └──────┬──────┘
               │                               │
               │                               │
┌─────────────┐│    ┌─────────────┐     ┌─────┴──────┐
│ AlertConfig ││    │   Alerts    │     │   Agents   │
├─────────────┤│    ├─────────────┤     ├─────────────┤
│ id          │└────┼─agent_id    │◄────┤ id          │
│ agent_id    │┌────┼─config_id   │     │ name        │
│ name        ││    │ triggered_at│     │ type_id     │
│ conditions  ││    │ resolved_at │     │ org_id      │
│ channels    ││    │ status      │     │ api_key     │
└─────────────┘│    └─────────────┘     │ webhook     │
               │                        │ config      │
               │    ┌─────────────┐     │ schedule    │
               │    │ Agent Logs  │     │ status      │
               │    ├─────────────┤     └─────────────┘
               └────┼─agent_id    │
                    │ status      │
                    │ message     │
                    │ details     │
                    │ started_at  │
                    │ duration    │
                    └─────────────┘
```

## Data Migration Strategy

1. **Initial Schema Creation**:
   - Run migration scripts to create initial tables
   - Seed essential data like default agent types

2. **Schema Upgrades**:
   - Use versioned migrations with tools like db-migrate
   - Ensure backward compatibility during upgrades

3. **Data Backups**:
   - Automated daily backups of PostgreSQL
   - Redis persistence configuration
   - InfluxDB retention policies for historical data
