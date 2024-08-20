1. security_group_id 的声明
security_group_id 是从创建的安全组资源中输出的值，因此需要在 security_group.tf 文件中正确声明和输出。

修正后的 security_group.tf
hcl
复制代码
# 创建MSK的安全组，允许从内部VPC访问
resource "aws_security_group" "msk_internal_01_sg" {
  name        = "msk-internal-sg"
  description = "Security group for MSK cluster"
  vpc_id      = var.internal_vpc_id

  ingress {
    from_port   = 9092
    to_port     = 9092
    protocol    = "tcp"
    cidr_blocks = [var.internal_nonnroutable_vpc_cidr_block]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# 输出安全组ID
output "security_group_id" {
  value = aws_security_group.msk_internal_01_sg.id
}
2. broker_node_group_info 的声明和引用
在 msk.tf 中，broker_node_group_info 是 AWS MSK 集群资源的一部分。在之前的配置中，它已经声明并配置，但我会明确其使用和引用方法。

修正后的 msk.tf
hcl
复制代码
# 创建MSK集群
resource "aws_msk_cluster" "msk_cluster" {
  cluster_name           = "example-msk-cluster"
  kafka_version          = "2.8.0"
  number_of_broker_nodes = 3
  
  # broker_node_group_info 包含关于每个 broker 节点的信息
  broker_node_group_info {
    instance_type  = "kafka.m5.large"
    client_subnets = var.internal_subnet_ids
    security_groups = [module.msk_security_group.security_group_id]  # 引用已创建的安全组ID
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
3. 更新后的 main.tf
hcl
复制代码
# 引入各个模块配置
module "msk_nlb_proxy" {
  source  = "./nlb.tf"
}

module "msk_cluster" {
  source  = "./msk.tf"
}

module "msk_security_group" {
  source  = "./security_group.tf"
}

module "msk_secrets_manager" {
  source  = "./secrets_manager.tf"
}

module "vpc_peering_routes" {
  source  = "./peering_routes.tf"
}
说明
security_group_id:

现在在 security_gro
