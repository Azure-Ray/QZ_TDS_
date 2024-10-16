import boto3
import datetime
from dateutil import tz

def lambda_handler(event, context):
    client = boto3.client('rds', region_name='ap-east-1')
    s3_bucket_arn = 'arn:aws:s3:::xxxxa'
    db_cluster_identifier = '111-dev-postgresql'
    
    current_date = datetime.datetime.now(tz=tz.tzutc()).replace(tzinfo=None)
    snapshot_identifier = f"db_cluster_identifier-{current_date.strftime('%Y-%m-%d-%H-%M')}"
    
    # 1. Create a new snapshot first
    try:
        response = client.create_db_cluster_snapshot(
            DBClusterIdentifier=db_cluster_identifier,
            DBClusterSnapshotIdentifier=snapshot_identifier
        )
        snapshot_info = {
            'SnapshotId': response['DBClusterSnapshot']['DBClusterSnapshotIdentifier'],
            'Status': response['DBClusterSnapshot']['Status'],
            'CreationTime': response['DBClusterSnapshot']['SnapshotCreateTime'].isoformat() if response['DBClusterSnapshot']['SnapshotCreateTime'] else 'Not available'
        }
    except Exception as e:
        print(f"Error creating snapshot: {str(e)}")
        return {
            'statusCode': 500,
            'body': f"Error creating snapshot: {str(e)}"
        }

    # 2. Export the snapshot to S3
    try:
        export_response = client.start_export_task(
            ExportTaskIdentifier=f"export-task-{snapshot_identifier}",
            SourceArn=f"arn:aws:rds:ap-east-1:your-account-id:cluster-snapshot:{snapshot_identifier}",
            S3BucketName='xxxxa',  # Ensure this is the correct bucket name
            IamRoleArn='arn:aws:iam::your-account-id:role/service-role/export-task-role',  # Ensure you provide an IAM role ARN with permissions
            KmsKeyId='your-kms-key-id',  # Optional, if encryption is required
        )
        print(f"Export started: {export_response['ExportTaskIdentifier']}")
    except Exception as e:
        print(f"Error starting export task: {str(e)}")
        return {
            'statusCode': 500,
            'body': f"Error exporting snapshot to S3: {str(e)}"
        }

    # 3. Delete old snapshots older than 6 months
    six_months_ago = current_date - datetime.timedelta(days=180)
    try:
        snapshots = client.describe_db_cluster_snapshots(DBClusterIdentifier=db_cluster_identifier)['DBClusterSnapshots']
        for snapshot in snapshots:
            creation_date = snapshot['SnapshotCreateTime'].replace(tzinfo=None)
            if creation_date < six_months_ago:
                client.delete_db_cluster_snapshot(DBClusterSnapshotIdentifier=snapshot['DBClusterSnapshotIdentifier'])
                print(f"Deleted snapshot: {snapshot['DBClusterSnapshotIdentifier']}")
    except Exception as e:
        print(f"Error deleting old snapshots: {str(e)}")
        return {
            'statusCode': 500,
            'body': f"Error deleting old snapshots: {str(e)}"
        }

    return {
        'statusCode': 200,
        'body': snapshot_info
    }
