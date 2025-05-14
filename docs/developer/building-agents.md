# Developer Guide: Building Custom Agents

## Introduction

This guide provides detailed instructions for developers to create custom agents that connect to the Autonomous Agent Platform. Whether you're building a simple scheduled task agent or a complex AI-driven agent, this guide will walk you through the process.

## Agent Architecture Overview

Agents in our platform follow a standardized architecture:

```
┌──────────────────────────────────┐
│           Agent                  │
├──────────────────────────────────┤
│ ┌────────────┐   ┌────────────┐  │
│ │ Core Logic │   │ Config     │  │
│ └────────────┘   └────────────┘  │
│                                  │
│ ┌────────────┐   ┌────────────┐  │
│ │ Platform   │   │ Execution  │  │
│ │ Connector  │   │ Engine     │  │
│ └────────────┘   └────────────┘  │
│                                  │
│ ┌────────────┐   ┌────────────┐  │
│ │ Logging    │   │ Error      │  │
│ │ Module     │   │ Handling   │  │
│ └────────────┘   └────────────┘  │
└──────────────────────────────────┘
```

### Key Components

1. **Core Logic**: The actual functionality of your agent
2. **Platform Connector**: Handles communication with the Agent Platform
3. **Execution Engine**: Manages the agent's execution lifecycle
4. **Config Module**: Loads and validates agent configuration
5. **Logging Module**: Standardized logging and telemetry
6. **Error Handling**: Standard error capture and reporting

## Getting Started

### Prerequisites

- Node.js (v18+)
- npm or yarn
- An agent configuration in the platform (obtain API key)

### Quickstart with Agent SDK

1. Create a new Node.js project:

```bash
mkdir my-custom-agent
cd my-custom-agent
npm init -y
```

2. Install the Agent SDK:

```bash
npm install @agent-platform/sdk
```

3. Create an agent file (index.js):

```javascript
const { Agent } = require('@agent-platform/sdk');

// Create and configure the agent
const agent = new Agent({
  apiKey: process.env.AGENT_API_KEY,
  platformUrl: process.env.PLATFORM_URL || 'https://api.agentplatform.example.com',
  name: 'My Custom Agent'
});

// Define the agent's execution logic
agent.setExecutionHandler(async (context) => {
  // Get configuration from context
  const { config, executionId } = context;
  
  // Log the execution start
  agent.logger.info('Starting execution', { executionId });
  
  try {
    // Implement your agent's business logic here
    const result = await performTask(config);
    
    // Return success
    return {
      status: 'success',
      data: result
    };
  } catch (error) {
    // Log and return error
    agent.logger.error('Execution failed', { error: error.message });
    return {
      status: 'error',
      error: error.message
    };
  }
});

// Example task function
async function performTask(config) {
  // This is where you would implement your specific agent functionality
  return {
    message: 'Task completed successfully',
    timestamp: new Date().toISOString()
  };
}

// Start the agent
agent.start();
```

4. Create a .env file:

```
AGENT_API_KEY=your_api_key_from_platform
PLATFORM_URL=https://api.agentplatform.example.com
```

5. Run your agent:

```bash
node -r dotenv/config index.js
```

## Agent Communication Methods

The platform supports two main methods for agent communication:

### 1. API-based Agents

API-based agents actively communicate with the platform by:

- Sending heartbeats at regular intervals
- Reporting execution results
- Fetching configuration updates

**Example: Implementing a heartbeat**

```javascript
// Send heartbeat every 30 seconds
setInterval(async () => {
  try {
    await agent.sendHeartbeat({ 
      status: 'online',
      metrics: {
        memory: process.memoryUsage().heapUsed / 1024 / 1024,
        cpu: getCpuUsage()
      }
    });
  } catch (error) {
    console.error('Failed to send heartbeat:', error);
  }
}, 30000);
```

### 2. Webhook-based Agents

Webhook-based agents receive HTTP requests from the platform:

- The platform sends HTTP requests to trigger agent execution
- Agents respond with execution results
- Useful for serverless or event-driven architectures

**Example: Express webhook handler**

```javascript
const express = require('express');
const { WebhookAgent } = require('@agent-platform/sdk');

const app = express();
app.use(express.json());

const agent = new WebhookAgent({
  apiKey: process.env.AGENT_API_KEY
});

app.post('/webhook/execute', agent.webhookAuthMiddleware, async (req, res) => {
  const { executionId, config } = req.body;
  
  try {
    // Implement your agent logic
    const result = await performTask(config);
    
    // Report success back to platform
    await agent.reportExecution({
      executionId,
      status: 'success',
      data: result
    });
    
    res.json({ status: 'success' });
  } catch (error) {
    // Report error back to platform
    await agent.reportExecution({
      executionId,
      status: 'error',
      error: error.message
    });
    
    res.status(500).json({ status: 'error', message: error.message });
  }
});

app.listen(3000, () => {
  console.log('Agent webhook server running on port 3000');
});
```

## Agent Configuration

### Configuration Schema

Define a JSON schema for your agent's configuration to enable automatic validation:

```javascript
const configSchema = {
  type: 'object',
  required: ['targetUrl', 'interval'],
  properties: {
    targetUrl: {
      type: 'string',
      format: 'uri'
    },
    interval: {
      type: 'number',
      minimum: 5,
      description: 'Interval in seconds'
    },
    headers: {
      type: 'object',
      additionalProperties: {
        type: 'string'
      }
    }
  }
};

// Register schema with the agent
agent.setConfigSchema(configSchema);
```

### Accessing Configuration

Configuration provided by the platform is available in the execution context:

```javascript
agent.setExecutionHandler(async (context) => {
  const { targetUrl, interval, headers } = context.config;
  
  // Use configuration values
  const response = await fetch(targetUrl, { headers });
  
  // Process response...
});
```

## Error Handling and Logging

### Structured Logging

The SDK provides a structured logger that integrates with the platform:

```javascript
// Different log levels
agent.logger.info('Processing item', { itemId: '123' });
agent.logger.warn('Rate limit approaching', { remaining: 10 });
agent.logger.error('Failed to process item', { 
  itemId: '123',
  error: error.message,
  stack: error.stack
});

// Log with execution context
agent.logger.info('Task completed', { 
  executionId: context.executionId,
  duration: 1250
});
```

### Error Handling Best Practices

1. **Catch and report errors properly**:

```javascript
try {
  // Risky operation
} catch (error) {
  agent.logger.error('Operation failed', { 
    error: error.message,
    operation: 'data-fetch'
  });
  
  // Determine if error is retryable
  if (isRetryableError(error)) {
    throw error; // Platform will retry
  } else {
    return { 
      status: 'error',
      error: error.message,
      retryable: false
    };
  }
}
```

2. **Implement circuit breakers for external dependencies**:

```javascript
const { CircuitBreaker } = require('@agent-platform/sdk/utils');

const apiBreaker = new CircuitBreaker({
  name: 'external-api',
  failureThreshold: 3,
  resetTimeout: 30000
});

agent.setExecutionHandler(async (context) => {
  try {
    const result = await apiBreaker.execute(async () => {
      return await callExternalApi();
    });
    
    return { status: 'success', data: result };
  } catch (error) {
    if (error.name === 'CircuitBreakerError') {
      agent.logger.error('Circuit open, skipping API call', { 
        circuitName: error.circuit 
      });
    }
    throw error;
  }
});
```

## Advanced Features

### Reporting Custom Metrics

Report metrics to the platform for monitoring:

```javascript
await agent.reportMetrics({
  execution_time: 1250, // milliseconds
  items_processed: 50,
  memory_usage: process.memoryUsage().heapUsed / 1024 / 1024,
  success_rate: 0.98
});
```

### State Management

For agents that need to maintain state between executions:

```javascript
// Save state
await agent.setState({
  lastProcessedId: '12345',
  processedCount: 1000,
  lastRunTimestamp: new Date().toISOString()
});

// Load state in a later execution
const state = await agent.getState();
const { lastProcessedId } = state || { lastProcessedId: null };
```

### Parallel Processing

For handling large workloads:

```javascript
const { parallelMap } = require('@agent-platform/sdk/utils');

agent.setExecutionHandler(async (context) => {
  const items = await fetchItemsToProcess();
  
  // Process up to 5 items concurrently
  const results = await parallelMap(items, processItem, 5);
  
  return {
    status: 'success',
    processed: results.length,
    failures: results.filter(r => r.error).length
  };
});

async function processItem(item) {
  try {
    // Process single item
    return { id: item.id, success: true };
  } catch (error) {
    return { id: item.id, error: error.message };
  }
}
```

## Debugging and Testing

### Local Testing

1. **Mock platform responses**:

```javascript
const { AgentMock } = require('@agent-platform/sdk/testing');

// Create a mock agent platform for testing
const mockPlatform = new AgentMock({
  // Mock configuration to use during tests
  config: {
    targetUrl: 'https://test-api.example.com',
    interval: 10
  }
});

// Use the mock in your tests
const agent = new Agent({
  apiKey: 'test-key',
  platformClient: mockPlatform
});

// Run test execution
const result = await agent.testExecution();
```

2. **Run with debug logging**:

```bash
DEBUG=agent:* node -r dotenv/config index.js
```

### Unit Testing

Example using Jest:

```javascript
const { Agent } = require('@agent-platform/sdk');
const { AgentMock } = require('@agent-platform/sdk/testing');

describe('My Custom Agent', () => {
  let agent;
  let mockPlatform;
  
  beforeEach(() => {
    // Setup mock platform and agent for each test
    mockPlatform = new AgentMock({
      config: {
        targetUrl: 'https://test-api.example.com'
      }
    });
    
    agent = new Agent({
      apiKey: 'test-key',
      platformClient: mockPlatform,
      name: 'Test Agent'
    });
    
    // Set up your agent's execution handler
    agent.setExecutionHandler(myExecutionHandler);
  });
  
  test('should process items successfully', async () => {
    // Arrange: Set up test data and mocks
    mockFetch(200, { data: [ ... ] });
    
    // Act: Execute the agent
    const result = await agent.testExecution();
    
    // Assert: Verify the results
    expect(result.status).toBe('success');
    expect(result.data.processed).toBe(5);
    
    // Verify logs were created
    expect(mockPlatform.getLogEntries()).toContainEqual(
      expect.objectContaining({ level: 'info', message: 'Processing complete' })
    );
  });
  
  test('should handle API errors gracefully', async () => {
    // Arrange: Simulate API error
    mockFetch(500, { error: 'Server error' });
    
    // Act: Execute the agent
    const result = await agent.testExecution();
    
    // Assert: Verify error handling
    expect(result.status).toBe('error');
    expect(result.error).toContain('API returned status 500');
  });
});

// Your actual execution handler
async function myExecutionHandler(context) {
  // Implementation
}

// Mock fetch for testing
function mockFetch(status, body) {
  global.fetch = jest.fn().mockImplementation(() => 
    Promise.resolve({
      status,
      json: () => Promise.resolve(body),
      ok: status >= 200 && status < 300
    })
  );
}
```

## Security Best Practices

1. **API Key Protection**:
   - Store API keys securely (environment variables, secret management)
   - Never commit API keys to source code

2. **Input Validation**:
   - Validate all configuration parameters
   - Use the provided schema validation

3. **Secure Communications**:
   - Always use HTTPS for API calls
   - Verify SSL certificates

4. **Dependency Security**:
   - Regularly update dependencies
   - Run security audits (`npm audit`)

## Example Agents

### 1. HTTP Monitoring Agent

```javascript
const { Agent } = require('@agent-platform/sdk');
const fetch = require('node-fetch');

const agent = new Agent({
  apiKey: process.env.AGENT_API_KEY,
  name: 'HTTP Monitor'
});

agent.setConfigSchema({
  type: 'object',
  required: ['endpoints'],
  properties: {
    endpoints: {
      type: 'array',
      items: {
        type: 'object',
        required: ['url', 'name'],
        properties: {
          url: { type: 'string', format: 'uri' },
          name: { type: 'string' },
          expectedStatus: { type: 'number', default: 200 },
          timeout: { type: 'number', default: 5000 }
        }
      }
    }
  }
});

agent.setExecutionHandler(async (context) => {
  const { endpoints } = context.config;
  const results = [];

  agent.logger.info(`Monitoring ${endpoints.length} endpoints`);

  for (const endpoint of endpoints) {
    try {
      const startTime = Date.now();
      const response = await fetch(endpoint.url, { 
        timeout: endpoint.timeout
      });
      const responseTime = Date.now() - startTime;

      const success = response.status === endpoint.expectedStatus;
      
      results.push({
        name: endpoint.name,
        url: endpoint.url,
        status: response.status,
        responseTime,
        success
      });

      if (!success) {
        agent.logger.warn(`Endpoint check failed: ${endpoint.name}`, {
          expected: endpoint.expectedStatus,
          actual: response.status,
          url: endpoint.url
        });
      }
    } catch (error) {
      results.push({
        name: endpoint.name,
        url: endpoint.url,
        error: error.message,
        success: false
      });

      agent.logger.error(`Endpoint check error: ${endpoint.name}`, {
        url: endpoint.url,
        error: error.message
      });
    }
  }

  const failedChecks = results.filter(r => !r.success).length;
  
  return {
    status: failedChecks > 0 ? 'warning' : 'success',
    data: {
      results,
      summary: {
        total: endpoints.length,
        successful: endpoints.length - failedChecks,
        failed: failedChecks
      }
    }
  };
});

agent.start();
```

### 2. Data Processing Agent

```javascript
const { Agent } = require('@agent-platform/sdk');
const fs = require('fs/promises');
const path = require('path');
const csv = require('csv-parser');
const { createReadStream } = require('fs');
const { pipeline } = require('stream/promises');

const agent = new Agent({
  apiKey: process.env.AGENT_API_KEY,
  name: 'Data Processor'
});

agent.setConfigSchema({
  type: 'object',
  required: ['inputDir', 'outputDir', 'processingRules'],
  properties: {
    inputDir: { type: 'string' },
    outputDir: { type: 'string' },
    archiveDir: { type: 'string' },
    filePattern: { type: 'string', default: '*.csv' },
    processingRules: {
      type: 'array',
      items: {
        type: 'object',
        required: ['field', 'operation'],
        properties: {
          field: { type: 'string' },
          operation: { 
            type: 'string',
            enum: ['sum', 'average', 'count', 'normalize']
          },
          parameters: { type: 'object' }
        }
      }
    }
  }
});

agent.setExecutionHandler(async (context) => {
  const { inputDir, outputDir, archiveDir, filePattern, processingRules } = context.config;
  const results = { processed: 0, errors: 0, files: [] };
  
  try {
    // Create output directory if it doesn't exist
    await fs.mkdir(outputDir, { recursive: true });
    if (archiveDir) {
      await fs.mkdir(archiveDir, { recursive: true });
    }
    
    // Find files to process
    const files = await findFiles(inputDir, filePattern);
    agent.logger.info(`Found ${files.length} files to process`);
    
    // Process each file
    for (const file of files) {
      try {
        const outputFile = path.join(outputDir, `processed_${path.basename(file)}`);
        const data = await processFile(file, processingRules);
        
        // Write processed data
        await fs.writeFile(outputFile, JSON.stringify(data, null, 2));
        
        // Archive or delete original
        if (archiveDir) {
          const archiveFile = path.join(archiveDir, path.basename(file));
          await fs.rename(file, archiveFile);
        } else {
          await fs.unlink(file);
        }
        
        results.processed++;
        results.files.push({
          name: path.basename(file),
          status: 'success',
          records: data.length
        });
      } catch (error) {
        results.errors++;
        results.files.push({
          name: path.basename(file),
          status: 'error',
          error: error.message
        });
        
        agent.logger.error(`Failed to process file: ${file}`, { error: error.message });
      }
    }
    
    return {
      status: results.errors > 0 ? 'warning' : 'success',
      data: results
    };
  } catch (error) {
    agent.logger.error('Processing failed', { error: error.message });
    return {
      status: 'error',
      error: error.message
    };
  }
});

async function findFiles(directory, pattern) {
  const files = await fs.readdir(directory);
  const regex = new RegExp(pattern.replace('*', '.*'));
  return files
    .filter(file => regex.test(file))
    .map(file => path.join(directory, file));
}

async function processFile(filePath, rules) {
  const results = [];
  
  await new Promise((resolve, reject) => {
    createReadStream(filePath)
      .pipe(csv())
      .on('data', (data) => {
        const processed = applyRules(data, rules);
        results.push(processed);
      })
      .on('end', resolve)
      .on('error', reject);
  });
  
  return results;
}

function applyRules(data, rules) {
  const result = { ...data };
  
  for (const rule of rules) {
    const { field, operation, parameters } = rule;
    
    switch (operation) {
      case 'normalize':
        result[field] = normalize(result[field], parameters);
        break;
      // Implement other operations...
    }
  }
  
  return result;
}

function normalize(value, params = {}) {
  const { min = 0, max = 100 } = params;
  const num = parseFloat(value);
  if (isNaN(num)) return value;
  
  return Math.min(Math.max(num, min), max);
}

agent.start();
```

## Deployment Options

### Running as a Docker Container

1. Create a Dockerfile:

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

CMD ["node", "index.js"]
```

2. Build and run:

```bash
docker build -t my-custom-agent .
docker run -e AGENT_API_KEY=your_api_key my-custom-agent
```

### Running as a AWS Lambda Function

1. Modify your agent for Lambda:

```javascript
const { LambdaAgent } = require('@agent-platform/sdk/lambda');

const agent = new LambdaAgent({
  apiKey: process.env.AGENT_API_KEY
});

agent.setExecutionHandler(async (context) => {
  // Your agent logic
});

// Lambda handler
exports.handler = agent.getLambdaHandler();
```

2. Deploy to Lambda using the AWS CLI or framework of your choice

### Running as a Kubernetes Job

Create a Kubernetes job manifest:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: my-custom-agent
spec:
  schedule: "0 * * * *"  # Run hourly
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: agent
            image: my-custom-agent:latest
            env:
            - name: AGENT_API_KEY
              valueFrom:
                secretKeyRef:
                  name: agent-secrets
                  key: api-key
          restartPolicy: OnFailure
```

## Additional Resources

- [Agent SDK Reference](https://docs.agentplatform.example.com/sdk)
- [Example Agent Templates](https://github.com/agent-platform/agent-templates)
- [Community Agent Repository](https://github.com/agent-platform/community-agents)
- [Agent Development Best Practices](https://docs.agentplatform.example.com/guides/agent-best-practices)
- [Troubleshooting Guide](https://docs.agentplatform.example.com/guides/troubleshooting)
