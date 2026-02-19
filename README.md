🧪 Laboratory Work 1
Variant 4 — Group Chat

![TypeScript](https://img.shields.io/badge/TypeScript-33.05%25-blue?style=flat-square)
![PHP](https://img.shields.io/badge/PHP-27.72%25-purple?style=flat-square)
![C#](https://img.shields.io/badge/C%23-25.52%25-green?style=flat-square)

Focus: Scaling delivery logic & Per-recipient status tracking

🧱 Part 1 — Component Diagram

To scale group messaging, we utilize the Fan-out pattern.
This allows the primary API to respond quickly to the sender while background workers handle distribution to every group member.

```mermaid
graph TD
    Client[Client App] --> API[API Gateway]
    API --> AuthService[Auth Service]
    API --> RoomService[Room / Group Service]
    API --> MsgService[Message Service]
    
    RoomService --> RoomDB[(Group Metadata DB)]
    MsgService --> MsgDB[(Message Store)]
    
    MsgService --> Queue{Delivery Queue}
    Queue --> FanOut[Fan-out Worker]
    
    FanOut -->|Get members list| RoomService
    FanOut --> StatusDB[(Delivery Status DB)]
    FanOut --> PushService[Push / WebSocket Service]
    
    PushService --> Recipient[Recipient Clients]
```
    
🔁 Part 2 — Sequence Diagram
Scenario

User A sends a message to a group. The system immediately confirms receipt and then asynchronously distributes it to other participants (User B, C, D, …).

```mermaid
sequenceDiagram
    participant A as User A (Sender)
    participant Server as Messaging System
    participant B as User B (Online)
    participant C as User C (Offline)

    A->>Server: Send message to Group
    Server-->>A: OK (Message Saved)
    
    Note over Server: System finds all group members
    
    Server->>B: Push via WebSocket (Instant)
    Server->>C: Push Notification (Stored for later)
    
    Note over Server: Update delivery status for each
```

🔄 Part 3 — State Diagram
Object

GroupMessageStatus — message state relative to a specific recipient.
We track the message state per participant to determine who has received or read it.

```mermaid
stateDiagram-v2
    [*] --> Pending: Task created in Queue
    Pending --> Delivering: Attempting connection
    
    state DeliveryProcess {
        Delivering --> Delivered: Received by device
        Delivered --> Read: Opened by user
    }
    
    Delivering --> Failed: Connection timeout
    Failed --> Retrying: Exponential backoff
    Retrying --> Delivering
    
    Read --> [*]
```

📚 Part 4 — ADR (Architecture Decision Record)

### ADR-004: Fan-out on Write for Group Messaging

**Status:** Accepted

### Context
In a group chat system, a single message must be delivered to **2–10,000+** participants.

We need to ensure:
- **Low latency** for the sender
- **Per-recipient delivery/read tracking**

### Decision
We implement a **Fan-out on Write** strategy using a message queue:
- **Message Service** persists the original message and enqueues a *fan-out* task
- **Fan-out Worker** consumes the task, retrieves the group members list, and creates individual delivery records
- **Statuses** are stored in a NoSQL DB (e.g., DynamoDB, Cassandra) using a composite key *(message_id + user_id)* for scalable writes

### Alternatives
- **Fan-out on Read**: clients pull messages instead of pushing
  - Hard to implement real-time delivery and read receipts
- **Synchronous fan-out**: the sender request distributes messages
  - Timeout risk for large groups (500+ users)

### Consequences
- **Performance:** sender receives immediate confirmation
- **Reliability:** failed worker → task stays in queue for retry
- **Data volume:** status records grow linearly → $O(N)$ per message
- **Complexity:** requires queue infrastructure (RabbitMQ / Kafka) and workers
