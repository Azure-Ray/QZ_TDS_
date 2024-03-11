Objective: Ensure a safe rollback to the snapshot version and update the EC2 target group if the Aurora PostgreSQL database upgrade from 12.12 to 12.17 fails.

Expected Outcome:

Rollback Execution: Automatically revert the database to the latest snapshot taken before the upgrade attempt.
Target Group Adjustment: Successfully modify the EC2 target group settings to redirect traffic to instances running the stable database version, ensuring no disruption in service.
Verification: Confirm that the database functions as expected post-rollback, with full connectivity and without performance degradation.
Notification: Generate an alert to the database administration team detailing the rollback and the necessity for target group adjustment, prompting further investigation.
2. Expected Result for Upgrade Failure and Rollback to Version 12.12

Objective: Seamlessly revert to version 12.12 using a snapshot if the upgrade to 12.17 encounters issues.

Expected Outcome:

Immediate Rollback: Initiate an automated rollback to the pre-upgrade snapshot of version 12.12 upon detecting the failure.
Connectivity and Functionality: Ensure the database retains full connectivity and operational capabilities, matching performance levels prior to the upgrade attempt.
Error Analysis: Log detailed error information for analysis, aiding in diagnosing the failure cause.
Notification: Alert the database team about the rollback, including failure details for future upgrade planning.
