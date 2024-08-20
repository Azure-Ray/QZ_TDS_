1. nlb.tf (调整后的NLB配置)
hcl
复制代码
# 创建MSK的NLB并配置目标组
module "msk_nlb_proxy" {
  source  = "../../modules/shared_modules/nlb"
  
  name_prefix = "msk-nlb"
  vpc_id      = var.internal_vpc_id
  subnet_ids  = var.internal_subnet_ids

  # 目标组映射配置，使用IP方式连接到MSK
  target_mapping = [
    {
      from_port = "9092"
      to_port   = "9092"
      protocol  = "TCP"
      ip_target = true
      targets   = data.aws_msk_cluster.msk.broker_nodes[*].broker_node_info.client_subnet_ips
    }
  ]
}

# 获取MSK集群的节点IP地址
data "aws_msk_cluster" "msk" {
  cluster_name = aws_msk_cluster.msk_cluster.cluster_name
}
2. msk.tf (MSK集群配置)
hcl
复制代码
# 创建MSK集群
resource "aws_msk_cluster" "msk_cluster" {
  cluster_name           = "example-msk-cluster"
  kafka_version          = "2.8.0"
  number_of_broker_nodes = 3
  broker_node_group_info {
    instance_type  = "kafka.m5.large"
    client_subnets = var.internal_subnet_ids
    security_groups = [module.msk_internal_01_sg.security_group_id]
  }

  encryption_info {
    encryption_in_transit {
      client_broker = "TLS"
      in_cluster    = true
    }
    encryption_at_rest_kms_key_arn = var.existing_kms_key_id
  }

  configuration_info {
    arn      = aws_msk_configuration.example.arn
    revision = aws_msk_configuration.example.latest_revision
  }
}

# MSK配置资源（如有需要）
resource "aws_msk_configuration" "example" {
  name      = "example-configuration"
  kafka_versions = ["2.8.0"]
  server_properties = <<EOF
auto.create.topics.enable = true
delete.topic.enable = true
EOF
}
3. variables.tf (修正后的变量声明)
hcl
复制代码
variable "internal_vpc_id" {}
variable "internal_subnet_ids" {}
variable "internal_nonnroutable_vpc_cidr_block" {}
variable "msk_password" {}
variable "existing_kms_key_id" {}
variable "idmz_vpc_id" {}
variable "internal_route_table_ids" {}
variable "peering_connection_id" {}
variable "msk_vpc_cidr_block" {}
4. dev.tfvars (变量文件)
hcl
复制代码
# MSK NLB配置变量
internal_vpc_id        = "vpc-xxxxxxxx"
internal_subnet_ids    = ["subnet-xxxxxx", "subnet-xxxxxx", "subnet-xxxxxx"] # MSK需要三个子网
internal_nonnroutable_vpc_cidr_block = "10.0.0.0/16"

# Secrets Manager配置变量
msk_password           = "YourMSKPassword"
existing_kms_key_id    = "arn:aws:kms:region:account-id:key/key-id"

# VPC Peering路由配置变量
idmz_vpc_id            = "vpc-yyyyyyyy"
internal_route_table_ids = ["rtb-xxxxxx", "rtb-yyyyyy"]
peering_connection_id   = "pcx-xxxxxxxx"
msk_vpc_cidr_block     = "10.1.0.0/16"
说明
MSK集群配置: 在 msk.tf 中创
