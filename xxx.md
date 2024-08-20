# Internal VPC（用于NLB）的配置变量
internal_vpc_id        = "vpc-xxxxxxxx"
internal_subnet_ids    = ["subnet-xxxxxx", "subnet-xxxxxx", "subnet-xxxxxx"]  # Internal VPC的子网
internal_nonnroutable_vpc_cidr_block = "10.0.0.0/16"  # Internal VPC的CIDR块

# IDMz VPC（用于MSK）的配置变量
idmz_vpc_id            = "vpc-yyyyyyyy"
idmz_subnet_ids        = ["subnet-yyyyyy", "subnet-yyyyyy", "subnet-yyyyyy"]  # IDMz VPC的子网
idmz_vpc_cidr_block    = "10.1.0.0/16"  # IDMz VPC的CIDR块

# Secrets Manager和KMS配置变量
msk_password           = "YourMSKPassword"  # MSK集群的密码
existing_kms_key_id    = "arn:aws:kms:region:account-id:key/key-id"  # 现有的KMS密钥ARN

# VPC Peering路由配置变量
internal_route_table_ids = ["rtb-xxxxxx", "rtb-yyyyyy"]  # Internal VPC中的路由表ID
peering_connection_id   = "pcx-xxxxxxxx"  # Internal和IDMz VPC之间的Peering连接ID
