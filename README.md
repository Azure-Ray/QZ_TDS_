Expected Results for Upgrading Aurora PostgreSQL from 12.12 to 12.17

Objective: Upgrade Aurora PostgreSQL from version 12.12 to 12.17 with minimal downtime, ensuring data integrity and no loss of connectivity.

Post-Upgrade Tests:

Connectivity: Confirm all client applications can connect without issues.
Performance Metrics:
CPU Utilization: Below 70% during peak loads.
Database Connections: Below 85% of the maximum limit.
Disk Space Utilization: Under 80%.
Read/Write Latency: Read under 10ms, Write under 15ms.
Error Rates: Less than 0.1% for queries and transactions.
Conclusion: Successful upgrade is indicated by meeting the above connectivity and performance metrics, ensuring the database operates efficiently and reliably at the new version without negatively impacting application performance.
