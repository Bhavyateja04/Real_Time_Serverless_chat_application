# 📲 Real-Time Serverless Chat Application

A scalable, real-time messaging application built using a **serverless AWS architecture**. It leverages Amazon API Gateway WebSockets, AWS Lambda, and Amazon DynamoDB to enable bidirectional communication, persistent message storage, and automatic scaling without managing traditional servers.

## 📌 Overview

The Real-Time Serverless Chat Application demonstrates how AWS serverless technologies can be integrated to build a lightweight, scalable communication system.

### 🎯 Objectives

- Enable real-time, bidirectional communication.
- Store chat messages persistently.
- Automatically scale with user demand.
- Minimize infrastructure management.
- Optimize costs using pay-per-use services.
- Secure interactions between AWS services using IAM.

## 🛠️ Tech Stack used

| Technology | Purpose |
|---|---|
| AWS Lambda | Executes backend logic without server management |
| Amazon API Gateway | Manages WebSocket connections and routing |
| Amazon DynamoDB | Stores connections and chat messages |
| WebSockets | Enables real-time communication |
| AWS IAM | Controls permissions and access |
| Amazon CloudWatch | Monitors logs, errors, and execution metrics |
| AWS SAM | Automates infrastructure provisioning and deployment |
| Python | Implements Lambda backend functions |

## 🏗️ System Architecture

```text
                 ┌──────────────────┐
                 │    Chat Client   │
                 └────────┬─────────┘
                          │
                    WebSocket
                          │
                          ▼
              ┌──────────────────────┐
              │ API Gateway WebSocket│
              └──────────┬───────────┘
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
        ┌─────────┐ ┌──────────┐ ┌──────────┐
        │ Connect │ │  Send    │ │Disconnect│
        │ Lambda  │ │ Message  │ │  Lambda  │
        └────┬────┘ └────┬─────┘ └────┬─────┘
             │           │            │
             └───────────┼────────────┘
                         ▼
                ┌─────────────────┐
                │ Amazon DynamoDB │
                │                 │
                │  Connections    │
                │  Chat Messages  │
                └────────┬────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │ API Gateway Management│
              │ API: Message Delivery │
              └──────────┬───────────┘
                         ▼
                  Connected Clients
```

### 🔄 Architecture Workflow

1. A client establishes a WebSocket connection through API Gateway.
2. The `$connect` Lambda function stores the connection ID in DynamoDB.
3. The client sends a message using the `sendMessage` route.
4. The message Lambda validates and stores the message.
5. Active connections are retrieved from DynamoDB.
6. The API Gateway Management API delivers the message to connected clients.
7. The `$disconnect` Lambda removes disconnected connection IDs.

## ✨ Key Features

### 💬 Real-Time Messaging
Enables clients to exchange messages through persistent WebSocket connections.

### ☁️ Serverless Backend
Uses AWS-managed services without requiring EC2 instances or dedicated backend servers.

### 📈 Automatic Scaling
AWS Lambda scales function execution according to incoming workloads, subject to service quotas.

### 💾 Persistent Storage
Stores chat messages and active WebSocket connection information in DynamoDB.

### 🔐 Access Control
Uses IAM execution roles to grant Lambda functions only the permissions they need.

### 📊 Monitoring
Uses CloudWatch logs and metrics to help identify execution failures and messaging issues.

### 💰 Pay-per-Use Infrastructure
Charges are based on service usage, making the architecture suitable for low-volume applications when usage remains within applicable free-tier limits.

## 📁 Project Structure

```text
real-time-serverless-chat-application/
│
├── lambda-functions/
│   ├── connect.py
│   ├── disconnect.py
│   └── message.py
│
├── template.yaml
├── README.md
└── .gitignore
```

| File | Description |
|---|---|
| `connect.py` | Registers new WebSocket connections |
| `disconnect.py` | Removes disconnected clients |
| `message.py` | Processes, stores, and broadcasts messages |
| `template.yaml` | Defines AWS resources and deployment configuration |
| `.gitignore` | Excludes local and sensitive files from version control |

## 🗄️ Database Design

The application uses Amazon DynamoDB to manage connections and chat history.

### 1. WebSocketConnections

| Attribute | Type | Description |
|---|---|---|
| `connectionId` | String | Unique API Gateway connection identifier |

**Partition key:** `connectionId`

### 2. ChatMessages

| Attribute | Type | Description |
|---|---|---|
| `messageId` | String | Unique message identifier |
| `connectionId` | String | Sender's connection identifier |
| `message` | String | Message content |
| `timestamp` | String or Number | Message creation time |

**Partition key:** `messageId`

> For production use, consider adding sender identity, conversation ID, and a timestamp-based sort key to support efficient chat history queries.

## ⚙️ Setup and Configuration

### Prerequisites

Ensure you have the following installed or configured:

- An AWS account.
- AWS CLI configured with appropriate credentials.
- Python 3.x.
- Node.js and npm for WebSocket testing.
- AWS SAM CLI if deploying through SAM.

### Step 1: Create DynamoDB Tables

Create the following tables in the AWS Management Console:

**Table 1: `WebSocketConnections`**
- Partition key: `connectionId`
- Attribute type: String

**Table 2: `ChatMessages`**
- Partition key: `messageId`
- Attribute type: String

### Step 2: Create a WebSocket API

1. Open Amazon API Gateway.
2. Create a WebSocket API.
3. Configure the route selection expression:

```text
$request.body.action
```

4. Create the following routes:

| Route | Lambda Function | Purpose |
|---|---|---|
| `$connect` | `connect` | Registers a client |
| `$disconnect` | `disconnect` | Removes a client |
| `sendMessage` | `message` | Processes incoming messages |
| `$default` | Optional handler | Handles unmatched routes |

5. Deploy the API to a stage, such as `dev`.

### Step 3: Configure Lambda Functions

Create the three Python Lambda functions and configure their environment variables.

For connection management, set:

```text
WEBSOCKET_TABLE=WebSocketConnections
```

For message processing, set:

```text
WEBSOCKET_TABLE=WebSocketConnections
MESSAGE_TABLE=ChatMessages
```

The `connect.py` function stores the new connection ID, while `disconnect.py` removes the corresponding record. The `message.py` function should validate incoming messages, persist them, and broadcast them using the API Gateway Management API.

### Step 4: Configure IAM Permissions

Attach an IAM execution role to the Lambda functions.

Required permissions may include:

- `dynamodb:PutItem`
- `dynamodb:DeleteItem`
- `dynamodb:Scan` or appropriate query permissions
- `execute-api:ManageConnections`

Restrict DynamoDB access to the relevant table ARNs and API Gateway access to the appropriate API and stage ARN. Avoid using unrestricted `"Resource": "*"` permissions in production.

### Step 5: Deploy the Application

If using AWS SAM, build and deploy the application from the project directory:

```bash
sam build
sam deploy --guided
```

Configure the required resource names, AWS Region, IAM permissions, and deployment stage when prompted.

After deployment, obtain the WebSocket endpoint from API Gateway. It will follow a format similar to:

```text
wss://<api-id>.execute-api.<region>.amazonaws.com/dev
```

## 🧪 Testing the Application

### Install `wscat`

```bash
npm install -g wscat
```

### Connect to the WebSocket API

```bash
wscat -c wss://<api-id>.execute-api.<region>.amazonaws.com/dev
```

### Send a Message

```json
{
  "action": "sendMessage",
  "message": "Hello World!"
}
```

Connect two or more clients to verify that messages are delivered to the intended recipients.

### Expected Results

- A new connection is registered in DynamoDB.
- A valid message is stored in the messages table.
- Connected clients receive the broadcast message.
- Disconnected clients are removed from the connection table.

## 🐛 Monitoring and Debugging

### Amazon CloudWatch

Monitor the following:

- Lambda invocation logs.
- Runtime exceptions and failed invocations.
- Connection and disconnection events.
- DynamoDB permission or throughput errors.
- WebSocket message delivery failures.

### Common Issues

| Issue | Possible Cause | Solution |
|---|---|---|
| Connection failure | Incorrect endpoint or stage | Verify the deployed WebSocket URL |
| DynamoDB access denied | Missing IAM permissions | Review the Lambda execution role |
| Messages not delivered | Incorrect Management API endpoint | Configure the endpoint using the current API's domain and stage |
| Stale connections | Client disconnected unexpectedly | Handle HTTP `410 Gone` responses and remove stale connection records |
| Missing messages | Incorrect route configuration | Verify the route selection expression and Lambda integration |

## 🔮 Future Enhancements

- 🔐 User authentication using Amazon Cognito.
- 👤 User profiles and unique usernames.
- 🟢 Online/offline presence indicators.
- ✍️ Typing indicators.
- 👥 Private and group conversations.
- 📎 Image and file sharing using Amazon S3.
- 🔔 Push notifications.
- ✓ Message delivery and read receipts.
- 🔍 Chat history, pagination, and search.
- 🛡️ End-to-end encryption.
- 📊 CloudWatch dashboards and alerts.
- 🧪 Automated unit and integration tests.

## 🚀 Learning Outcomes

Through this project, developers can gain practical experience with:

- Serverless application architecture.
- WebSocket communication and event-driven processing.
- AWS Lambda integration with API Gateway.
- DynamoDB data modeling.
- IAM roles and least-privilege access.
- AWS deployment automation using SAM.
- CloudWatch monitoring and troubleshooting.

## 📌 Conclusion

The **Real-Time Serverless Chat Application** demonstrates how AWS Lambda, API Gateway WebSockets, and DynamoDB can be combined to build a scalable messaging backend with minimal infrastructure management.

Its modular architecture supports future improvements such as authenticated users, private messaging, group conversations, notifications, and enhanced message history.

The project provides a practical foundation for understanding serverless application development and real-time communication on AWS.

---

**Technologies:** Python · AWS Lambda · API Gateway WebSockets · DynamoDB · IAM · CloudWatch · AWS SAM
