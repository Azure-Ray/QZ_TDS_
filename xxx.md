感谢你耐心的解释，抱歉之前的混乱。根据你的要求，我将重新整理整个Terraform配置，确保正确配置VPC、MSK、NLB、目标组和VPC Peering，并使用IP方式连接MSK和NLB。以下是完整的、经过纠正的Terraform代码：

### 文件结构
```
/msk-terraform
  |-- main.tf
  |-- msk.tf
  |-- nlb.tf
  |-- security_group.tf
  |-- peering_routes.tf
  |-- secrets_manager.tf
  |-- variables.tf
/env_var
  |-- dev.tfvars
```

### 1. `main.tf` (主文件)
```hcl
# 引入各个模块配置
module "msk_cluster" {
  source  = "./msk.tf"
}

module "msk_security_group" {
  source  = "./security_group.tf"
}

module "msk_nlb_proxy" {
  source  = "./nlb.tf"
}

module "vpc_peering_routes" {
  source  = "./peering_routes.tf"
}

module "msk_secrets_manager" {
  source  = "./secrets_manager.tf"
}
```

### 2. `msk.tf` (MSK集群配置)
```hcl
# 创建MSK集群
resource "aws_msk_cluster" "msk_cluster" {
  cluster_name           = "example-msk-cluster"
  kafka_version          = "2.8.0"
  number_of_broker_nodes = 3

  broker_node_group_info {
    instance_type  = "kafka.m5.large"
    client_subnets = var.idmz_subnet_ids  # MSK部署在IDMz VPC中
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
  name           = "example-configuration"
  kafka_versions = ["2.8.0"]
  server_properties = <<EOF
auto.create.topics.enable = true
delete.topic.enable = true
EOF
}
```

### 3. `nlb.tf` (NLB配置)
```hcl
# 创建NLB并配置目标组
resource "aws_lb" "msk_nlb" {
  name               = "msk-nlb"
  load_balancer_type = "network"
  internal           = true
  security_groups    = [module.msk_security_group.security_group_id]  # 直接引用安全组
  subnets            = var.internal_subnet_ids  # NLB部署在Internal VPC中
}

resource "aws_lb_target_group" "msk_target_group" {
  name     = "msk-target-group"
  port     = 9092
  protocol = "TCP"
  vpc_id   = var.internal_vpc_id
  target_type = "ip"

  health_check {
    enabled             = true
    interval            = 30
    protocol            = "TCP"
    port                = "9092"
  }
}

resource "aws_lb_listener" "msk_listener" {
  load_balancer_arn = aws_lb.msk_nlb.arn
  port              = 9092
  protocol          = "TCP"
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.msk_target_group.arn
  }
}

# 获取MSK集群的节点IP地址
data "aws_msk_cluster" "msk" {
  cluster_name = aws_msk_cluster.msk_cluster.cluster_name
}

# 配置NLB目标组的目标IP
resource "aws_lb_target_group_attachment" "msk_targets" {
  count            = length(data.aws_msk_cluster.msk.broker_node_group_info[0].client_subnet_ips)
  target_group_arn = aws_lb_target_group.msk_target_group.arn
  target_id        = element(flatten([for broker in data.aws_msk_cluster.msk.broker_node_group_info: broker.client_subnet_ips]), count.index)
  port             = 9092
}
```

### 4. `security_group.tf` (安全组配置)
```hcl
# 创建MSK的安全组，允许从内部VPC访问
resource "aws_security_group" "msk_internal_01_sg" {
  name        = "msk-internal-sg"
  description = "Security group for MSK cluster"
  vpc_id      = var.idmz_vpc_id  # 安全组应用于IDMz VPC中的MSK集群

  ingress {
    from_port   = 9092
    to_port     = 9092
    protocol    = "tcp"
    cidr_blocks = [var.internal_nonnroutable_vpc_cidr_block]  # 允许来自Internal VPC的流量
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
```

### 5. `peering_routes.tf` (VPC Peering路由配置)
```hcl
# 为MSK和NLB之间的Peering连接设置路由
module "vpc_peering_routes" {
  source            = "../../modules/shared_modules/vpc-peering-routes"
  source_vpc_id     = var.internal_vpc_id
  destination_vpc_id = var.idmz_vpc_id
  route_table_ids   = var.internal_route_table_ids

  routes = [
    {
      destination_cidr_block = var.idmz_vpc_cidr_block
      target                 = var.peering_connection_id
    },
    {
      destination_cidr_block = var.internal_nonnroutable_vpc_cidr_block
      target                 = var.peering_connection_id
    }
  ]
}
```

### 6. `secrets_manager.tf` (Secrets Manager配置)
```hcl
# 使用现有的KMS密钥创建用于存储MSK密码的Secrets Manager
resource "aws_secretsmanager_secret" "msk_secret" {
  name        = "msk_password"
  description = "MSK cluster password"

  # 使用现有的KMS密钥进行加密
  kms_key_id = var.existing_kms_key_id
}

# 将密码存储到Secrets Manager中
resource "aws_secretsmanager_secret_version" "msk_secret_value" {
  secret_id     = aws_secretsmanager_secret.msk_secret.id
  secret_string = var.msk_password  # 密码从变量中获取，并存储在Secrets Manager中
}
```

### 7. `variables.tf` (变量声明)
```hcl
variable "internal_vpc_id" {
  description = "ID of the internal VPC where the NLB will be deployed"
  type        = string
}

variable "internal_subnet_ids" {
  description = "List of subnet IDs in the internal VPC where the NLB will be deployed"
  type        = list(string)
}

variable "internal_nonnroutable_vpc_cidr_block" {
  description = "CIDR block of the internal VPC for routing"
  type        = string
}

variable "idmz_vpc_id" {
  description = "ID of the IDMz VPC where the MSK cluster will be deployed"
  type        = string
}

variable "idmz_subnet_ids" {
  description = "List of subnet IDs in the IDMz VPC where the MSK cluster will be deployed"
  type        = list(string)
}

variable "existing_kms_key_id" {
  description = "ARN of the existing KMS key used to encrypt the MSK password in Secrets Manager"
  type        = string
}

variable "msk_password" {
  description = "MSK cluster password to be stored in Secrets Manager"
  type        = string
}

variable "internal_route_table_ids" {
  description = "List of route table IDs in the internal VPC"
  type        = list(string)
}

variable "peering_connection_id" {
  description = "ID of the VPC peering connection between the internal and IDMz VPCs"
  type        = string
}

variable "idmz_vpc_cidr_block" {
  description = "CIDR block of the IDMz VPC"
  type        = string
}
```

### 8. `dev.tfvars` (变量文件)
```hcl
# MSK和NLB配置变量
internal_vpc_id        = "vpc-xxxxxxxx"
internal_subnet_ids    = ["subnet-xxxxxx", "subnet-xxxxxx", "subnet-xxxxxx"]
internal_nonnroutable_vpc_cidr_block = "10.0.0.0/16"

idmz_vpc_id            = "vpc-yyyyyyyy"
idmz_subnet_ids        = ["subnet-yyyyyy", "subnet-yyyyyy", "subnet-yyyyyy"]
idmz_vpc_c

