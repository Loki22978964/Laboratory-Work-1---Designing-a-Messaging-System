🧪 Laboratory Work 1: Variant 4 — Group ChatFocus: Scaling delivery logic & Per-recipient status tracking.🧱 Part 1 — Component DiagramTo scale group messaging, we utilize the Fan-out pattern. This allows the primary API to respond quickly to the sender while background workers handle the distribution of the message to every group member.Фрагмент кодаgraph TD
    Client[Client App] --> API[API Gateway]
    API --> AuthService[Auth Service]
    API --> RoomService[Room/Group Service]
    API --> MsgService[Message Service]
    
    RoomService --> RoomDB[(Group Metadata DB)]
    MsgService --> MsgDB[(Message Store)]
    
    MsgService --> Queue{Delivery Queue}
    Queue --> FanOut[Fan-out Worker]
    
    FanOut --> RoomService : "Get members list"
    FanOut --> StatusDB[(Delivery Status DB)]
    FanOut --> PushService[Push/WebSocket Service]
    
    PushService --> Recipient[Recipient Clients]
🔁 Part 2 — Sequence DiagramScenario: User A sends a message to a group. The system confirms receipt and then asynchronously distributes it to other participants (User B and others).Фрагмент кодаsequenceDiagram
    participant A as User A (Sender)
    participant API as API Gateway
    participant MS as Message Service
    participant Q as Message Queue
    participant FO as Fan-out Worker
    participant RS as Room Service
    participant B as User B (Recipient)

    A->>API: POST /groups/{id}/messages
    API->>MS: saveMessage(groupId, content)
    MS-->>API: 202 Accepted (MsgID: 101)
    API-->>A: Confirm Sent (Success)
    
    Note over MS, Q: Asynchronous Fan-out Process
    MS->>Q: Enqueue Task (MsgID: 101, GroupID: 5)
    
    Q->>FO: Process Message 101
    FO->>RS: getGroupMembers(GroupID: 5)
    RS-->>FO: List: [User B, User C, User D...]
    
    loop For each group member
        FO->>FO: createDeliveryRecord(MsgID, MemberID, status='pending')
        FO->>B: Deliver via WebSocket/Push
    end
🔄 Part 3 — State DiagramObject: GroupMessageStatus (Message state relative to a specific recipient).In a group chat, we track the message state for each participant individually to determine who has received or read the text.Фрагмент кодаstateDiagram-v2
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
📚 Part 4 — ADR (Architecture Decision Record)ADR-004: Using Fan-out on Write for Group MessagingStatusAcceptedContextIn a group chat system, a single message must be delivered to multiple participants (ranging from 2 to 10,000+). We need to ensure:Low latency for the sender.The ability to track delivered/read status for each group participant individually.DecisionWe will implement a Fan-out on Write strategy using a message queue:The Message Service only persists the original message and enqueues a "fan-out" task.The Fan-out Worker consumes this task, retrieves the group member list, and creates individual delivery records for each member.Statuses are stored in a NoSQL database (e.g., DynamoDB or Cassandra) using a composite key message_id + user_id to allow high-frequency writes and easy horizontal scaling.AlternativesFan-out on Read: Messages are not actively pushed; clients "pull" updates when entering the chat.Downside: Difficult to implement real-time push notifications and per-user read receipts.Synchronous Fan-out: Distribution happens during the sender's HTTP request.Downside: For large groups (e.g., 500+ members), the sender's request would hang or timeout.ConsequencesPerformance: The sender receives an immediate confirmation.Reliability: If a worker fails, the task remains in the queue for retry.Data Volume: The number of status records grows linearly ($O(N)$), where $N$ is the number of participants.Complexity: Requires additional infrastructure for queues (RabbitMQ/Kafka) and workers.
