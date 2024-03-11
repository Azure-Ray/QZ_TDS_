Health Check Journey for Aurora PostgreSQL Database Post-Rollback to Version 12.12

Rollback Confirmation: Verify the database is operational and accessible post-rollback to version 12.12.
Connectivity Test: Execute a simple query to ensure the database responds correctly.
Performance Metrics Review: Check critical metrics such as CPU utilization, connection counts, and disk space usage to confirm they are within normal ranges.
Error Log Analysis: Scan the database logs for any immediate errors or warnings post-rollback.
Application Integration Test: Run a predefined set of application operations that interact with the database to ensure application-level functionality and data integrity.
Monitoring Setup Verification: Confirm monitoring tools and alerts are correctly configured to capture future anomalies.
Health Check Scenario for Aurora PostgreSQL Database Post-Rollback to Version 12.12

Objective: Quickly ascertain the operational status and health of the Aurora PostgreSQL database following a rollback to version 12.12.

Steps:

Connectivity: Establish a database connection and execute a 'SELECT' query.
Performance Metrics: Verify CPU utilization is below 70%, disk usage is under 80%, and the database is operating with at least 15% free connections.
Error Check: Review database logs for unusual error rates or critical warnings that could indicate problems.
Functionality Test: Perform basic CRUD operations through an application interface to ensure the database interacts correctly with the application layer.
Monitoring Check: Ensure alerting systems are active and reporting as expected, with no unresolved critical alerts.
