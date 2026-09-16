# 📲 Real-Time Serverless Chat Application

A scalable **real-time chat application** built using a fully serverless AWS architecture. The application uses **API Gateway WebSockets, AWS Lambda, DynamoDB, and IAM** to provide real-time, bi-directional messaging without managing traditional servers.

---

## 📌 Objective

The objective of this project is to build a **real-time messaging application using serverless architecture** that provides:

* Real-time communication
* Automatic scalability
* Persistent message storage
* Low infrastructure maintenance
* Cost-efficient deployment
* Secure communication between AWS services

---

## 🛠️ Technologies Used

| Technology             | Purpose                                           |
| ---------------------- | ------------------------------------------------- |
| **AWS Lambda**         | Executes backend logic without managing servers   |
| **Amazon API Gateway** | Handles REST/WebSocket API communication          |
| **Amazon DynamoDB**    | Stores WebSocket connections and chat messages    |
| **WebSockets**         | Enables real-time bi-directional communication    |
| **AWS IAM**            | Manages permissions and secure access             |
| **AWS CloudWatch**     | Monitors Lambda execution and application logs    |
| **AWS SAM**            | Optional infrastructure and deployment automation |

---

## 🏗️ Architecture

```text
                    ┌─────────────────┐
                    │   Chat Client   │
                    └────────┬────────┘
                             │
                          WebSocket
                             │
                             ▼
                ┌─────────────────────────┐
                │   API Gateway WebSocket │
                └────────────┬────────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
        ┌──────────┐   ┌──────────┐   ┌──────────┐
        │ $to connect │   │sendMessage│  │$disconnect│
        │  Lambda  │   │  Lambda   │  │  Lambda  │
        └────┬─────┘   └─────┬─────┘  └────┬─────┘
             │               │              │
             └───────────────┼──────────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │    DynamoDB     │
                    ├─────────────────┤
                    │ Connections     │
                    │ Messages        │
                    └─────────────────┘
```

---

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
└── .env
```

### Lambda Functions

* `connect.py` — Handles new WebSocket connections.
* `disconnect.py` — Removes disconnected users.
* `message.py` — Processes and broadcasts chat messages.

---

## ⚙️ Features

### 💬 Real-Time Messaging

Users can send and receive messages instantly through WebSocket connections.

### ☁️ Serverless Architecture

The application runs completely on AWS managed services without requiring EC2 or traditional backend servers.

### 📈 Automatic Scaling

AWS Lambda automatically handles increasing workloads without manually managing servers.

### 💾 Persistent Storage

Chat messages and active WebSocket connections are stored in DynamoDB.

### 🔐 Secure Access

AWS IAM policies control communication between Lambda, DynamoDB, and API Gateway.

### 💰 Cost Efficient

The serverless architecture follows a pay-per-use model and can be suitable for low-volume applications within AWS Free Tier limits.

---

# 🔧 Setup Instructions

## Step 1: Create DynamoDB Tables

### WebSocket Connections Table

Create a DynamoDB table:

```text
Table Name: WebSocketConnections
Partition Key: connectionId
Type: String
```

This table stores active WebSocket connection IDs.

### Chat Messages Table

Create another table:

```text
Table Name: ChatMessages
Partition Key: messageId
Type: String
```

Each message can be assigned a UUID as its `messageId`.

---

## Step 2: Create WebSocket API

Open **Amazon API Gateway** and create a WebSocket API.

### Route Selection Expression

```text
$request.body.action
```

Configure the following routes:

| Route         | Lambda Function | Purpose                |
| ------------- | --------------- | ---------------------- |
| `$connect`    | `connect`       | Stores new connection  |
| `$disconnect` | `disconnect`    | Removes connection     |
| `sendMessage` | `message`       | Processes messages     |
| `$default`    | Optional        | Handles unknown routes |

---

## Step 3: Configure Lambda Functions

### `connect.py`

```python
import boto3
import os

dynamodb = boto3.client("dynamodb")

def lambda_handler(event, context):

    connection_id = event["requestContext"]["connectionId"]

    dynamodb.put_item(
        TableName=os.environ["WEBSOCKET_TABLE"],
        Item={
            "connectionId": {
                "S": connection_id
            }
        }
    )

    return {
        "statusCode": 200
    }
```

Configure the following Lambda environment variable:

```text
WEBSOCKET_TABLE=WebSocketConnections
```

---

### `disconnect.py`

```python
import boto3
import os

dynamodb = boto3.client("dynamodb")

def lambda_handler(event, context):

    connection_id = event["requestContext"]["connectionId"]

    dynamodb.delete_item(
        TableName=os.environ["WEBSOCKET_TABLE"],
        Key={
            "connectionId": {
                "S": connection_id
            }
        }
    )

    return {
        "statusCode": 200
    }
```

---

### `message.py`

The message Lambda should:

1. Receive the WebSocket message.
2. Extract the message content.
3. Store the message in DynamoDB.
4. Retrieve active WebSocket connections.
5. Send the message to connected clients using the API Gateway Management API.

Example request:

```json
{
  "action": "sendMessage",
  "message": "Hello everyone!"
}
```

---

# 🔐 Step 4: Configure IAM Permissions

The Lambda execution role requires permissions to interact with DynamoDB and API Gateway.

Example policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:PutItem",
        "dynamodb:DeleteItem",
        "dynamodb:Scan",
        "dynamodb:GetItem"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "execute-api:ManageConnections"
      ],
      "Resource": "*"
    }
  ]
}
```

> **Production recommendation:** Replace `"Resource": "*"` with the specific DynamoDB table ARN and API Gateway resource ARN.

---

# 🚀 Step 5: Deploy the Application

The application can be deployed through the AWS Console or AWS SAM.

If using AWS SAM:

```bash
sam build
```

Then:

```bash
sam deploy --guided
```

After deployment, AWS API Gateway will provide a WebSocket endpoint similar to:

```text
wss://<api-id>.execute-api.<region>.amazonaws.com/dev
```

---

# 🧪 Step 6: Test the WebSocket API

Install `wscat`:

```bash
npm install -g wscat
```

Connect to the WebSocket API:

```bash
wscat -c wss://<api-id>.execute-api.<region>.amazonaws.com/dev
```

Send a message:

```json
{
  "action": "sendMessage",
  "message": "Hello World!"
}
```

If multiple clients are connected, the message can be broadcast to the active connections.

---

# 🐛 Testing & Debugging

### AWS CloudWatch

Use **CloudWatch Logs** to monitor:

* Lambda execution
* Runtime errors
* Connection events
* Message processing
* DynamoDB errors
* WebSocket failures

### API Gateway

Use API Gateway's WebSocket testing functionality to simulate:

* `$connect`
* `$disconnect`
* `sendMessage`

### Error Handling

Lambda functions should handle failures using `try/except` blocks.

Example:

```python
try:
    # Application logic
    pass

except Exception as error:
    print(f"Error: {error}")

    return {
        "statusCode": 500,
        "body": "Internal server error"
    }
```

---

# 🔄 Application Flow

```text
User connects
      ↓
API Gateway WebSocket
      ↓
$connect Lambda
      ↓
Connection ID stored in DynamoDB
      ↓
User sends message
      ↓
sendMessage Lambda
      ↓
Message stored in DynamoDB
      ↓
Active connections retrieved
      ↓
API Gateway Management API
      ↓
Message broadcast to users
      ↓
User disconnects
      ↓
$disconnect Lambda
      ↓
Connection ID removed
```

---

# 📊 Advantages

* **Scalable:** Lambda automatically scales with demand.
* **Real-time:** WebSockets provide instant communication.
* **Serverless:** No EC2 or server management required.
* **Reliable:** Uses managed AWS infrastructure.
* **Persistent:** Messages can be stored in DynamoDB.
* **Cost-efficient:** Pay only for consumed AWS resources.
* **Maintainable:** Backend functionality is separated into independent Lambda functions.

---

# 🔮 Future Enhancements

* 🔐 User authentication with Amazon Cognito
* 👤 User profiles
* 🟢 Online/offline status
* ✍️ Typing indicators
* 📎 File and image sharing
* 👥 Group conversations
* 🔔 Push notifications
* ✓ Message delivery/read status
* 🔍 Chat history and search
* 🛡️ End-to-end encryption
* 📊 CloudWatch monitoring dashboard

---

# 📌 Conclusion

This project demonstrates how **AWS serverless technologies can be combined to build a scalable real-time communication system**. By using API Gateway WebSockets, AWS Lambda, and DynamoDB, the application eliminates the need for traditional backend servers while providing real-time messaging and persistent data storage.

The architecture can be extended to support authentication, group chats, notifications, media sharing, and other production-level messaging features.
