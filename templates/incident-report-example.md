# [INC-042] Service Degradation: Database Connection Pool Exhaustion

## Summary
The API service experienced intermittent 503 Service Unavailable errors for 45 minutes, affecting approximately 30% of global users.

## Timeline (UTC)
- **14:20** - Automated Grafana alert triggered: "High API Error Rate".
- **14:25** - L2 Support Engineer initiated log analysis via Splunk.
- **14:40** - Root cause identified as database connection leak in the Auth-Service.
- **14:50** - Connection pool parameters updated and service restarted.
- **15:05** - Error rates returned to baseline; monitoring confirmed stability.

## Investigation and Root Cause Analysis (RCA)
- Log analysis revealed repeated "Could not get JDBC Connection; nested exception is HikariPool-1 - Connection is not available" errors.
- System metrics showed that the maximum number of concurrent database connections (100) was reached, while CPU and Memory usage remained within normal limits.
- **Root Cause:** A recent deployment in the Auth-Service introduced a code path where database connections were not properly closed during exception handling, leading to a gradual leak and eventual exhaustion of the pool.

## Resolution
1. Temporary increase of the `maximum-pool-size` from 100 to 150 in the service configuration.
2. Executed rolling restart of the `auth-service` pods to clear stale connections.
3. Verified active connection count using `SHOW PROCESSLIST` on the MySQL instance.

## Prevention and Follow-up Actions
- **Monitoring:** Implement a new alert for "Database Connection Usage > 80%" to provide early warning.
- **Engineering:** Escalated to L3/Development to implement an urgent fix for the connection leak in the exception handling logic.
- **Documentation:** Updated the "API Troubleshooting Guide" with steps to monitor HikariCP metrics.