# Production Architecture

## [Production Architecture](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/13-module-3#production-architecture)

```text
┌────────────┐     JWT Token      ┌─────────────────────────────────────┐
│  Streamlit │ ──────────────────►│  AgentCore Runtime                  │
│  Chat App  │                    │  ┌─────────────────────────────┐    │
└─────┬──────┘                    │  │ Product Catalog Agent       │    │
      │                           │  │ (Claude Sonnet 4.6)         │    │
      │ Login                     │  │ • JWT role extraction       │    │
      ▼                           │  │ • Tool filtering by role    │    │
┌─────────────┐                   │  │ • OTEL auto-instrumentation │    │
│  Cognito    │                   │  └──────────────┬──────────────┘    │
│  User Pool  │                   └─────────────────┼──────────────────┘
│             │                                     │ SigV4 MCP
│ Groups:     │                   ┌─────────────────▼──────────────────┐
│ - customer  │                   │  AgentCore Gateway                 │
│ - admin     │                   │  ┌───────────┐  ┌───────────────┐ │
└─────────────┘                   │  │ JWT Auth  │  │ RBAC          │ │
                                  │  │ Authorizer│  │ Interceptor   │ │
                                  │  └───────────┘  └───────────────┘ │
                                  └─────────────────┬─────────────────┘
                                                    │
                                  ┌─────────────────▼─────────────────┐
                                  │  Product Tools Lambda              │
                                  │  11 tools (6 read + 5 admin)      │
                                  └─────────────────┬─────────────────┘
                                                    │
                                  ┌─────────────────▼─────────────────┐
                                  │  DynamoDB Products Table           │
                                  └───────────────────────────────────┘
```

## Architecture Flow

1. **Streamlit Chat App**

   * Provides the user interface.
   * Users authenticate using Amazon Cognito.
   * Sends a JWT token to AgentCore Runtime.

2. **Amazon Cognito**

   * Handles user authentication.
   * Users belong to roles such as:

     * `customer`
     * `admin`

3. **AgentCore Runtime**

   * Hosts the **Product Catalog Agent**.
   * Uses **Claude Sonnet 4.6**.
   * Extracts the user's role from the JWT token.
   * Filters available tools based on the user's role.
   * Uses OpenTelemetry (OTEL) for observability.

4. **AgentCore Gateway**

   * Connects the AI agent with backend tools.
   * Uses **SigV4 MCP** communication.
   * Provides:

     * JWT authentication
     * Role-Based Access Control (RBAC)

5. **Product Tools Lambda**

   * Contains **11 tools**.
   * `6` read-only tools.
   * `5` admin tools.

6. **DynamoDB Products Table**

   * Stores product catalog data.
   * Lambda tools read and update product information.

## Request Flow

```text
User
  ↓
Streamlit Chat App
  ↓
Amazon Cognito Authentication
  ↓
JWT Token
  ↓
AgentCore Runtime
  ↓
Product Catalog Agent
  ↓
AgentCore Gateway
  ↓
RBAC / Authorization
  ↓
Product Tools Lambda
  ↓
DynamoDB
```

## Key AWS Services

* Amazon Bedrock
* Amazon Bedrock AgentCore Runtime
* Amazon Bedrock AgentCore Gateway
* Amazon Cognito
* AWS Lambda
* Amazon DynamoDB
* Amazon CloudWatch
* OpenTelemetry (OTEL)
* Model Context Protocol (MCP)
