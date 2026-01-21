# 6. SmartKhata - Operations & Deployment

**Purpose**: Production deployment, monitoring, scaling, incident response  
**Audience**: DevOps, SRE, operations team  
**Consolidates**: DEPLOYMENT_GUIDE.md, OPERATIONS_RUNBOOK.md, PERFORMANCE_GUIDELINES.md

---

## 📦 Deployment Architecture

### Infrastructure Stack

**Cloud**: AWS (India region - ap-south-1)
**Database**: PostgreSQL 14 (RDS multi-AZ)
**App Server**: Node.js on ECS (Fargate)
**Mobile Hosting**: App Store, Play Store
**CDN**: CloudFront for static assets
**Email/SMS**: SendGrid, MSG91

---

## 🚀 Deployment Process

### Pre-Deployment Checklist

- [ ] All tests passing (unit + integration + E2E)
- [ ] Code review approved
- [ ] Security scan passed (no high-severity issues)
- [ ] Database migrations tested
- [ ] Staging deployment verified
- [ ] Performance benchmarks met (<100ms response)

### Deployment Steps

**1. Build Stage** (5 min)
```bash
# Build Docker image
docker build -t smartkhata:v1.2.3 .

# Push to ECR
aws ecr push smartkhata:v1.2.3
```

**2. Staging Deployment** (10 min)
```bash
# Deploy to staging ECS cluster
aws ecs update-service --cluster staging --service smartkhata-app \
  --force-new-deployment
```

**3. Smoke Tests** (5 min)
```bash
# Run health checks
npm run test:smoke

# Check API endpoints
curl https://staging-api.smartkhata.com/health
```

**4. Production Deployment** (Blue-Green, 0 downtime)
```bash
# Route 10% to new version
aws route53 weighted-routing-policy \
  --weight-old=90 --weight-new=10

# Monitor errors for 5 minutes
curl /metrics | check error_rate < 0.1%

# Route 50% to new version after 5 min
aws route53 weighted-routing-policy \
  --weight-old=50 --weight-new=50

# Route 100% to new version after 5 min
aws route53 weighted-routing-policy \
  --weight-old=0 --weight-new=100

# Keep old version ready for 1 hour rollback
```

**5. Post-Deployment** (10 min)
```bash
# Verify production metrics
check_metrics {
  error_rate < 0.05%
  p99_latency < 200ms
  database_connections < 80%
}

# Notify team
slack_post "✅ Deployment successful: v1.2.3"
```

---

## 📊 Performance Benchmarks

### API Response Times (Targets)

- GET /api/invoices: **<50ms**
- POST /api/invoices: **<100ms**
- GET /api/parties/:id/ledger: **<150ms**
- POST /api/invoices/:id/finalize: **<200ms**
- GET /api/reports/profit-loss: **<500ms**

**Testing**:
```bash
npm run test:performance
# Simulates 100 concurrent users
# Ensures p99 latency <200ms
```

---

### Mobile Performance (Targets)

- App size: **<50MB**
- Cold start: **<2 seconds**
- First invoice create: **<30 seconds** (on 4G)
- Offline sync: **<5 seconds** for 100 invoices

---

### Database Performance

- Connection pool: 20-50 connections
- Query execution: <50ms average
- Backup frequency: Daily at 2 AM IST
- Backup retention: 30 days

---

## 🔍 Monitoring & Alerting

### Key Metrics

**Application**:
```
error_rate (target: <0.1%)
response_time_p99 (target: <200ms)
uptime (target: 99.9%)
active_users (tracked)
```

**Database**:
```
connection_count (alert if >40)
slow_queries (alert if >10/min)
replication_lag (alert if >1sec)
disk_usage (alert if >80%)
```

**Infrastructure**:
```
cpu_usage (alert if >70%)
memory_usage (alert if >80%)
disk_io (alert if >80%)
network_bandwidth (tracked)
```

### Monitoring Stack

**Metrics**: Prometheus + Grafana
**Logs**: ELK Stack (Elasticsearch, Logstash, Kibana)
**Alerting**: PagerDuty
**Error Tracking**: Sentry

**Setup**:
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'smartkhata-api'
    static_configs:
      - targets: ['localhost:3001']
  
  - job_name: 'database'
    static_configs:
      - targets: ['localhost:9187']  # postgres_exporter
```

### Alert Rules

```yaml
# High Error Rate
alert: HighErrorRate
condition: rate(errors_total[5m]) > 0.001
action: PagerDuty critical

# High Latency
alert: HighLatency
condition: histogram_quantile(0.99, response_time) > 0.2
action: PagerDuty warning

# Database Issues
alert: DatabaseDown
condition: pg_up{} == 0
action: PagerDuty critical

# Disk Space
alert: DiskFull
condition: node_filesystem_avail_bytes / node_filesystem_size_bytes < 0.2
action: PagerDuty warning
```

---

## 🔄 Backup & Disaster Recovery

### Backup Strategy

**Database**:
- Automated daily backup at 2 AM IST
- 30-day retention
- Multi-region replication

**Application Data**:
- All uploads in S3 with versioning
- S3 cross-region replication

**Disaster Recovery Plan**:
- **RTO** (Recovery Time Objective): <4 hours
- **RPO** (Recovery Point Objective): <1 hour

**Test Recovery**:
```bash
# Monthly disaster recovery drill
restore_database_from_backup("2026-01-01")
verify_data_integrity()
# Should complete in <30 min
```

---

## 🔐 Security Operations

### SSL/TLS Certificates

- Issued by: AWS ACM (free)
- Duration: Auto-renewed before expiry
- Minimum TLS version: 1.2

### API Security

- **Rate Limiting**: 100 req/min per IP (adjustable per plan)
- **DDoS Protection**: AWS Shield
- **WAF Rules**: AWS WAF + custom rules

---

## 📲 Mobile App Deployment

### App Store Deployment

**iOS (Apple App Store)**:
```
1. Build release APK (Xcode)
2. Create app signing certificate
3. Upload to TestFlight (beta)
4. Test for 1 week
5. Submit to App Review
6. Wait 1-2 days for approval
7. Release to production
```

**Android (Google Play Store)**:
```
1. Build release AAB (Android Studio)
2. Sign APK with keystore
3. Upload to Google Play Console
4. Create release notes
5. Submit to Play Store
6. Usually approved within 1 hour
7. Release to production
```

---

## 🚨 Incident Response

### Incident Severity Levels

**Critical (P1)**:
- System completely down
- Data loss occurred
- Security breach
- Response: <15 min

**High (P2)**:
- Major feature broken
- Performance degraded >50%
- Response: <30 min

**Medium (P3)**:
- Minor feature broken
- Performance degraded <50%
- Response: <4 hours

**Low (P4)**:
- Bug with workaround
- Response: <1 day

### Incident Response Workflow

```
1. DETECT → Automated alert fires
2. ALERT → PagerDuty notifies on-call engineer
3. ACK → Engineer joins incident channel
4. INVESTIGATE → Gather logs, metrics
5. MITIGATE → Implement emergency fix or rollback
6. RESOLVE → Deploy permanent fix
7. POSTMORTEM → Review within 24 hours
```

---

## 📋 Runbook Examples

### Website Down

**Symptoms**: `/health` returns 503
**Steps**:
1. Check CloudWatch metrics
2. Check database connection pool
3. Restart ECS tasks if database OK
4. Investigate error logs in Sentry
5. If unfixable in 5 min: Rollback to previous version

### High Memory Usage

**Symptoms**: Memory >80%
**Steps**:
1. Check for memory leaks in Sentry
2. Identify problematic endpoint
3. Increase container memory temporarily
4. Deploy fix for leak
5. Revert temporary increase

### Database Down

**Symptoms**: Database connectivity errors
**Steps**:
1. Check AWS RDS dashboard
2. Check replication status
3. If replica available: Failover to replica
4. If not: Restore from latest backup
5. Notify users of data state

---

## ✅ Launch Checklist

- [ ] Infrastructure provisioned (AWS, database, CDN)
- [ ] Monitoring configured (Prometheus, Grafana, Sentry)
- [ ] Backups configured and tested
- [ ] SSL certificates installed
- [ ] Domain DNS configured
- [ ] Mobile apps submitted to stores
- [ ] Email templates configured
- [ ] SMS gateway tested
- [ ] Incident response plan documented
- [ ] Team trained on runbooks
- [ ] Soft launch to 10% users
- [ ] Gradual rollout to 100%

---

**Related Documents**: 01_ARCHITECTURE.md, 04_SECURITY.md, 07_DEVELOPMENT.md, 08_UX.md, 09_INTEGRATIONS.md
