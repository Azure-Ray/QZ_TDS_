Data Synchronization: The system iterates through all data in the mmm sn table from the last job execution to the current time, checking for updates. If updates are found, they are synchronized to the sss PD table.

Issue: This process only occurs when data is updated. If only the approval status of a specific group is updated without affecting the overall CC status, then the sss PD table will not be updated. Consequently, it cannot refresh to the latest data for all CRs.
Status Update: Iterates through all data from the last job to the current time in the sss PD table and calls the sn API to refresh the latest approval status.

Issue: Due to limitations in step 1, this step might not be able to acquire the latest data for all CRs.
Data Grouping: Iterates through all data in the sss PD table, grouping all ERs by group name.

Issue: High volume of data and processing, resulting in low efficiency.
Data Update and Email Preparation: Iterates through our maintained groups, fetching the portion of CRs grouped by group name, then updates the associated tables required for bulk approval web display and organizes data needed for email sending.

Email Sending: After all updates, sends emails to notify relevant personnel.

Optimized Logic:
Optimized Data Synchronization: Maintain the original step, iterating through all data in the mmm sn table from the last job to the current time, checking for updates, and synchronizing them to the sss PD table.

Optimized Status Update: Iterates through all data from the last job to the current time in the sss PD table and calls the sn API to refresh the latest approval status.

Optimized Data Grouping: Only iterates through data in the sss PD table from the past 15 days, grouping all CRs by our maintained groups.

3.5 New Step - Status Refresh: Iterates through the data processed in step 3, and calls an internal API to update the latest approval status information, reassembling the data.

Optimized Data Update and Email Preparation: Iterates through our maintained groups, fetching the portion of CRs grouped by group name, updates the associated tables required for bulk approval web display, and organizes data needed for sending emails.

Optimized Email Sending: Remains unchanged. After all updates, sends emails to notify relevant personnel.

Through these optimizations, our goal is to enhance the precision and efficiency of data processing, reduce the volume of ineffective or outdated data processed, and ensure that all relevant personnel receive updates on the latest statuses in a timely manner.
