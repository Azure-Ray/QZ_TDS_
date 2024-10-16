import boto3

def lambda_handler(event, context):
    client = boto3.client('rds', region_name='ap-east-1')

    try:
        response = client.start_export_task(
            ExportTaskIdentifier='your-export-task-id',
            SourceArn='arn:aws:rds:ap-east-1:your-account-id:cluster:your-cluster-id',
            S3BucketName='your-s3-bucket-name',
            IamRoleArn='arn:aws:iam::your-account-id:role/your-role',
            KmsKeyId='your-kms-key-id',  # 如果需要加密，填写KMS密钥
            S3Prefix='your/s3/prefix'  # 可选：S3存储的文件前缀
        )
        print(f"Export started: {response['ExportTaskIdentifier']}")
    except Exception as e:
        print(f"Error starting export task: {str(e)}")
        return {
            'statusCode': 500,
            'body': f"Error starting export task: {str(e)}"
        }

    return {
        'statusCode': 200,
        'body': f"Export task started successfully"
    }
