Mobile / Browser
      │
      ▼
CloudFront CDN
  - Static assets
  - Event pages cache
  - Cache hit → User
      │ API requests
      ▼
Application Load Balancer (ALB)
  - SSL Termination
  - Health Checks (10s)
  - Rate Limit: 200 req/IP/min
      │
      ├── Node.js API x1
      ├── Node.js API x2
      ├── Node.js API x3
      └── Node.js API xN
             │
             ├── READ
             ▼
      Redis Cluster (3 Nodes)
        - availability:event:cat (TTL 30s)
        - event:[id] (TTL 3600s)
        - seat_lock:[id] (SETNX TTL 30s)
        ← Redis SETNX lock (see CONCURRENCY.md)

             │
             ├── WRITE
             ▼
      PostgreSQL Primary
             │
             └── Replicates To
                   ├── Read Replica 1
                   └── Read Replica 2
      ← Read replicas for browsing traffic

             │
             └── PUBLISH
                   ▼
          SQS Payment Queue
          - Visibility Timeout: 20s
          - Max Receive Count: 3
          - DLQ: payment-dlq
          ← Async queue (see QUEUE.md)

                   │
                   ▼
         Payment Worker (ECS)
         1. Read SQS Message
         2. Call Razorpay
         3. Update DB
         4. Publish SNS
         5. Delete SQS Message

                   │
                   ▼
                AWS SNS
                ├── SES Email
                └── SMS