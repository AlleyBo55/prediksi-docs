# Scaling Roadmap: Phase 1 - Database Layer (MongoDB)

**Goal:** Scale from single MongoDB instance to production-ready cluster supporting 10K-50K concurrent users

**Timeline:** 1-2 weeks  
**Priority:** CRITICAL (prevents platform crashes)  
**Cost:** ~$50-150/month

---

## Prerequisites

- [ ] **Backup current database**
  - Export all collections: `mongodump --uri="mongodb://localhost:27017/predixi"`
  - Store backup in S3/Google Cloud Storage
  - Test restore procedure
  - **Time:** 30 minutes

---

## Step 1: MongoDB Atlas Setup (Day 1)

### 1.1 Create MongoDB Atlas Account
- [ ] Sign up at [mongodb.com/cloud/atlas](https://www.mongodb.com/cloud/atlas)
- [ ] Choose cloud provider (AWS recommended for Solana compatibility)
- [ ] Select region closest to your users (e.g., Singapore for SEA, US-East for global)
- [ ] **Time:** 15 minutes

### 1.2 Create Production Cluster
- [ ] Create new cluster with these specs:
  - **Tier:** M10 (2GB RAM, 10GB storage)
  - **Replication:** 3-node replica set (automatic)
  - **Backup:** Enable continuous backup (point-in-time recovery)
  - **Auto-scaling:** Enable storage auto-scaling
- [ ] **Cost:** ~$57/month
- [ ] **Time:** 20 minutes (cluster provisioning)

### 1.3 Configure Security
- [ ] Network Access:
  - Add your current IP for migration
  - Add application server IPs (or 0.0.0.0/0 temporarily)
  - Plan: Use VPC peering for production
- [ ] Database Access:
  - Create new user: `predixi_app` with `readWrite` role
  - Generate strong password (save in password manager)
  - Enable SCRAM authentication
- [ ] **Time:** 15 minutes

### 1.4 Migrate Data
- [ ] Get connection string from Atlas (looks like: `mongodb+srv://...`)
- [ ] Restore backup to Atlas:
  ```bash
  mongorestore --uri="mongodb+srv://predixi_app:<password>@cluster.mongodb.net/predixi" dump/
  ```
- [ ] Verify data integrity:
  - Check document counts match
  - Test critical queries (user login, market listing)
- [ ] **Time:** 1-2 hours (depending on data size)

---

## Step 2: Optimize Connection Pooling (Day 1-2)

### 2.1 Update MongoDB Connection Code
- [ ] Open `api-backend/cmd/api/main.go`
- [ ] Modify `connectMongoDB` function:
  ```go
  func connectMongoDB(uri string) (*mongo.Client, error) {
      ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
      defer cancel()

      client, err := mongo.Connect(ctx, options.Client().
          ApplyURI(uri).
          SetMaxPoolSize(200).        // Max connections
          SetMinPoolSize(10).         // Keep warm connections
          SetMaxConnIdleTime(30*time.Second).
          SetServerSelectionTimeout(5*time.Second).
          SetRetryWrites(true).       // Auto-retry failed writes
          SetRetryReads(true))        // Auto-retry failed reads

      if err != nil {
          return nil, err
      }

      if err := client.Ping(ctx, nil); err != nil {
          return nil, err
      }

      return client, nil
  }
  ```
- [ ] **Time:** 10 minutes

### 2.2 Update Environment Variables
- [ ] Update `.env`:
  ```env
  MONGO_URI=mongodb+srv://predixi_app:<password>@cluster.mongodb.net/?retryWrites=true&w=majority
  MONGO_DATABASE=predixi
  ```
- [ ] **Time:** 5 minutes

### 2.3 Test Connection
- [ ] Run application locally with new connection
- [ ] Verify all CRUD operations work
- [ ] Check Atlas metrics dashboard for connection count
- [ ] **Time:** 30 minutes

---

## Step 3: Add Database Indexes (Day 2)

### 3.1 Analyze Slow Queries
- [ ] Enable MongoDB profiling in Atlas:
  - Go to Cluster → Metrics → Profiler
  - Set threshold: 100ms
- [ ] Run application under load (use Apache Bench or k6)
- [ ] Identify top 5 slowest queries
- [ ] **Time:** 1 hour

### 3.2 Create Critical Indexes
- [ ] Connect to Atlas using MongoDB Compass or CLI
- [ ] Create indexes for hot queries:

**Users Collection:**
```javascript
db.users.createIndex({ "google_id": 1 }, { unique: true })
db.users.createIndex({ "email": 1 })
db.users.createIndex({ "created_at": -1 })
```

**Markets Collection:**
```javascript
db.markets.createIndex({ "status": 1, "created_at": -1 })
db.markets.createIndex({ "category": 1, "status": 1 })
db.markets.createIndex({ "end_time": 1 })
db.markets.createIndex({ "creator_id": 1 })
db.markets.createIndex({ "volume": -1 }) // For trending
```

**Bets Collection:**
```javascript
db.bets.createIndex({ "user_id": 1, "created_at": -1 })
db.bets.createIndex({ "market_id": 1, "created_at": -1 })
db.bets.createIndex({ "user_id": 1, "status": 1 })
db.bets.createIndex({ "market_id": 1, "outcome": 1 })
```

**Transactions Collection:**
```javascript
db.transactions.createIndex({ "user_id": 1, "created_at": -1 })
db.transactions.createIndex({ "order_id": 1 }, { unique: true })
db.transactions.createIndex({ "status": 1, "created_at": -1 })
```

**Comments Collection:**
```javascript
db.comments.createIndex({ "market_id": 1, "created_at": -1 })
db.comments.createIndex({ "user_id": 1 })
```

- [ ] **Time:** 30 minutes

### 3.3 Verify Index Performance
- [ ] Run `explain()` on slow queries to confirm index usage
- [ ] Check Atlas Performance Advisor for recommendations
- [ ] Benchmark query times (should be <50ms for indexed queries)
- [ ] **Time:** 30 minutes

---

## Step 4: Configure Read Preferences (Day 3)

### 4.1 Implement Read/Write Splitting
- [ ] Create new file: `api-backend/internal/database/mongo.go`
  ```go
  package database

  import (
      "go.mongodb.org/mongo-driver/mongo"
      "go.mongodb.org/mongo-driver/mongo/options"
      "go.mongodb.org/mongo-driver/mongo/readpref"
  )

  // GetWriteCollection returns collection with primary read preference
  func GetWriteCollection(db *mongo.Database, name string) *mongo.Collection {
      opts := options.Collection().SetReadPreference(readpref.Primary())
      return db.Collection(name, opts)
  }

  // GetReadCollection returns collection with secondary preferred
  func GetReadCollection(db *mongo.Database, name string) *mongo.Collection {
      opts := options.Collection().SetReadPreference(readpref.SecondaryPreferred())
      return db.Collection(name, opts)
  }
  ```
- [ ] **Time:** 15 minutes

### 4.2 Update Repository Pattern
- [ ] Modify repositories to use read/write collections:
  - **Writes:** User creation, bet placement, market creation → Primary
  - **Reads:** Market listing, leaderboards, analytics → Secondary
- [ ] Example for `MarketRepository`:
  ```go
  func (r *MarketRepository) List() {
      coll := database.GetReadCollection(r.db, "markets") // Read from replica
      // ... query logic
  }

  func (r *MarketRepository) Create() {
      coll := database.GetWriteCollection(r.db, "markets") // Write to primary
      // ... insert logic
  }
  ```
- [ ] **Time:** 2 hours

---

## Step 5: Add Monitoring & Alerts (Day 3-4)

### 5.1 Configure Atlas Alerts
- [ ] In Atlas Dashboard → Alerts:
  - **CPU Usage** > 80% → Email + Slack
  - **Connections** > 150 → Email
  - **Disk Usage** > 75% → Email + Slack
  - **Replication Lag** > 10s → Email
  - **Query Execution Time** > 1000ms → Email
- [ ] Add team email and Slack webhook
- [ ] **Time:** 30 minutes

### 5.2 Add Application-Level Monitoring
- [ ] Install MongoDB driver metrics:
  ```bash
  go get go.mongodb.org/mongo-driver/event
  ```
- [ ] Add connection pool monitoring in `main.go`:
  ```go
  poolMonitor := &event.PoolMonitor{
      Event: func(evt *event.PoolEvent) {
          logger.Info("MongoDB pool event",
              zap.String("type", evt.Type),
              zap.Int64("connections", evt.ConnectionCount))
      },
  }

  client, err := mongo.Connect(ctx, options.Client().
      ApplyURI(uri).
      SetPoolMonitor(poolMonitor))
  ```
- [ ] **Time:** 1 hour

### 5.3 Create Health Check Endpoint
- [ ] Update `/health` endpoint to check MongoDB:
  ```go
  router.GET("/health", func(c *gin.Context) {
      ctx, cancel := context.WithTimeout(c, 2*time.Second)
      defer cancel()
      
      if err := mongoClient.Ping(ctx, nil); err != nil {
          c.JSON(503, gin.H{"status": "unhealthy", "mongo": "down"})
          return
      }
      
      c.JSON(200, gin.H{"status": "healthy", "mongo": "up"})
  })
  ```
- [ ] **Time:** 15 minutes

---

## Step 6: Load Testing (Day 4-5)

### 6.1 Setup Load Testing Tool
- [ ] Install k6: `brew install k6` (Mac) or download from k6.io
- [ ] Create test script: `api-backend/tests/load/markets.js`
  ```javascript
  import http from 'k6/http';
  import { check, sleep } from 'k6';

  export let options = {
      stages: [
          { duration: '2m', target: 100 },   // Ramp up to 100 users
          { duration: '5m', target: 100 },   // Stay at 100 users
          { duration: '2m', target: 500 },   // Ramp up to 500 users
          { duration: '5m', target: 500 },   // Stay at 500 users
          { duration: '2m', target: 0 },     // Ramp down
      ],
  };

  export default function () {
      let res = http.get('https://api.predixi.com/api/v1/markets');
      check(res, { 'status is 200': (r) => r.status === 200 });
      sleep(1);
  }
  ```
- [ ] **Time:** 30 minutes

### 6.2 Run Load Tests
- [ ] Test 1: 100 concurrent users (baseline)
- [ ] Test 2: 500 concurrent users (target load)
- [ ] Test 3: 1000 concurrent users (stress test)
- [ ] Monitor Atlas metrics during tests:
  - CPU usage should stay <70%
  - Query time should stay <100ms
  - Connection count should stay <150
- [ ] **Time:** 3 hours

### 6.3 Document Results
- [ ] Create `LOAD_TEST_RESULTS.md` with:
  - Requests per second achieved
  - Average response time
  - Error rate
  - Database metrics (CPU, memory, connections)
  - Bottlenecks identified
- [ ] **Time:** 1 hour

---

## Step 7: Deployment & Rollback Plan (Day 5)

### 7.1 Prepare Deployment
- [ ] Update production `.env` with Atlas connection string
- [ ] Test application with Atlas in staging environment
- [ ] Prepare rollback script (switch back to old MongoDB URI)
- [ ] **Time:** 1 hour

### 7.2 Deploy to Production
- [ ] Schedule maintenance window (low-traffic time)
- [ ] Deploy new code with Atlas connection
- [ ] Monitor for 30 minutes:
  - Check error logs
  - Verify Atlas connection count
  - Test critical user flows (login, bet placement)
- [ ] **Time:** 2 hours

### 7.3 Post-Deployment Validation
- [ ] Run smoke tests on production
- [ ] Check Atlas Performance Advisor for issues
- [ ] Monitor for 24 hours
- [ ] Document any issues in incident log
- [ ] **Time:** 1 hour + 24h monitoring

---

## Success Metrics

After completion, you should achieve:

- ✅ **Uptime:** 99.9% (no single point of failure)
- ✅ **Capacity:** Support 10K-50K concurrent users
- ✅ **Performance:** <100ms query response time (p95)
- ✅ **Scalability:** Auto-scaling storage, manual scaling compute
- ✅ **Reliability:** Automatic failover in <30 seconds
- ✅ **Monitoring:** Real-time alerts for issues

---

## Cost Breakdown

| Item | Monthly Cost |
|------|--------------|
| MongoDB Atlas M10 | $57 |
| Backup storage (10GB) | $2.50 |
| Data transfer | ~$5 |
| **Total** | **~$65/month** |

**ROI:** Prevents downtime that could cost $1000+/hour in lost revenue and reputation.

---

## Rollback Plan

If issues occur:

1. **Immediate:** Switch `.env` back to old MongoDB URI
2. **Deploy:** Push rollback commit
3. **Verify:** Check application connects to old database
4. **Investigate:** Review Atlas logs and metrics
5. **Time to rollback:** <5 minutes

---

## Next Steps (Phase 2)

After Phase 1 is stable:
- [ ] Redis clustering (see `SCALING_ROADMAP_PHASE2.md`)
- [ ] Application horizontal scaling
- [ ] Load balancer setup
- [ ] WebSocket scaling

---

## Questions & Support

- **MongoDB Atlas Docs:** https://docs.atlas.mongodb.com/
- **Connection String Format:** https://docs.mongodb.com/manual/reference/connection-string/
- **Performance Best Practices:** https://docs.mongodb.com/manual/administration/analyzing-mongodb-performance/

**Estimated Total Time:** 40-50 hours (1-2 weeks with testing)
