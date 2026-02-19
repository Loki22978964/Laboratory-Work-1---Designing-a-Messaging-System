# Laboratory-Work-1---Designing-a-Messaging-System

Part 1 — Component Diagram (30%)
Variant 4 — Group Chat

Focus: scaling delivery logic

📌 Task

Create a Component Diagram that shows:

system components

their responsibilities

interactions between them

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

Sends messages to group chats

Receives messages from other participants

Displays message delivery and read status

Interactions:

Communicates with Backend API via HTTP or WebSocket

Receives messages through WebSocket or Push notifications

Backend API

Responsibilities:

Serves as an entry point to the system

Validates incoming requests

Routes messaging operations to Message Service

Interactions:

Receives requests from Client

Sends commands to Message Service

Message Service

Responsibilities:

Implements core group chat logic

Handles fan-out strategy for multiple recipients

Manages per-recipient delivery status

Publishes message delivery events

Interactions:

Receives requests from Backend API

Persists messages and metadata in Database

Sends delivery events to Queue

Database

Responsibilities:

Stores chat messages

Stores group membership information

Stores per-user delivery and read status

Interactions:

Accessed by Message Service only

Delivery Mechanism (Queue / WebSocket / Push)

Responsibilities:

Processes asynchronous delivery events

Delivers messages to online users via WebSocket

Sends push notifications to offline users

Improves scalability and fault tolerance

Interactions:

Consumes events from Queue

Sends messages to Clients

🔄 Interaction Summary

Client sends a message to a group chat

Backend API forwards the request to Message Service

Message Service stores the message and delivery metadata

Delivery events are published to a Queue

Delivery mechanism asynchronously delivers messages to all recipients

✅ Conclusion

This component diagram represents a scalable group chat system where messages are delivered to multiple recipients with separate delivery status per user. The use of an asynchronous delivery mechanism ensures high performance and scalability for group messaging scenarios.

Якщо хочеш, наступним кроком можемо:

зробити Sequence Diagram (Part 2)

або скоротити це до ідеальної “екзаменаційної” версії на 10–12 рядків 👌
