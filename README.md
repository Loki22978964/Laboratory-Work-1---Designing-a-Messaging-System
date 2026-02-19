Laboratory Work 1: Designing a Messaging System
Part 1 — Component Diagram (30%)
Variant 4: Group Chat

Focus: Scaling delivery logic

📌 Task
Create a Component Diagram that illustrates:

System components

Their specific responsibilities

Interactions between them

🧩 System Components
Client (Web / Mobile)

Backend API

Message Service

Database

Delivery Mechanism (Queue / WebSocket / Push)

📊 Component Diagram (Mermaid)

graph LR
    Client[Client<br/>(Web / Mobile)]
    API[Backend API]
    MessageService[Message Service]
    DB[(Database)]
    Queue[Message Queue]
    Delivery[Delivery Mechanism<br/>(WebSocket / Push)]

    Client -->|Send / Receive messages| API
    API -->|Forward requests| MessageService
    MessageService -->|Store messages & metadata| DB
    MessageService -->|Publish delivery events| Queue
    Queue -->|Async processing| Delivery
    Delivery -->|Deliver messages| Client

    🧱 Components and Responsibilities
Client (Web / Mobile)
Responsibilities:

Sends messages to group chats.

Receives messages from other participants.

Displays message delivery and read status.

Interactions:

Communicates with Backend API via HTTP or WebSocket.

Receives messages through WebSocket or Push notifications.

Backend API
Responsibilities:

Serves as the primary entry point to the system.

Validates incoming requests and authentication.

Routes messaging operations to the Message Service.

Interactions:

Receives requests from the Client.

Sends commands to the Message Service.

Message Service
Responsibilities:

Implements core group chat logic.

Handles fan-out strategy for multiple recipients.

Manages per-recipient delivery status.

Publishes message delivery events.

Interactions:

Receives requests from the Backend API.

Persists messages and metadata in the Database.

Sends delivery events to the Queue.

Database
Responsibilities:

Stores chat messages and history.

Stores group membership information.

Stores per-user delivery and read status.

Interactions:

Accessed exclusively by the Message Service.

Delivery Mechanism (Queue / WebSocket / Push)
Responsibilities:

Processes asynchronous delivery events.

Delivers messages to online users via WebSocket.

Sends Push notifications to offline users.

Improves scalability and fault tolerance.

Interactions:

Consumes events from the Queue.

Sends messages back to the Clients.

🔄 Interaction Summary
Client sends a message to a group chat.

Backend API validates and forwards the request to the Message Service.

Message Service stores the message and delivery metadata in the Database.

Delivery events are published to a Queue.

Delivery mechanism asynchronously delivers messages to all recipients (online or offline).

✅ Conclusion
This component diagram represents a scalable group chat system where messages are delivered to multiple recipients with separate delivery status per user. The use of an asynchronous delivery mechanism ensures high performance and scalability, preventing bottlenecks during heavy group messaging traffic.
