1. main.tf (主文件)
hcl
复制代码
# 调用所有配置模块
module "msk_nlb_proxy" {
  source  = "./nlb.tf"
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
2. nlb.tf (NLB配置)
hcl
复制代码
# 创建用于MSK的NLB
module "msk_nlb_proxy" {
  source  = "../../modules/shared_modules/nlb"
  environment = var.environment
  name_prefix = "msk-nlb"
  vpc_id = var.internal_vpc_id
  subnet_ids = var.internal_subnet_ids

  # 目标组映射配置，使用IP方式连接到MSK
  target_mapping = [
    {
      from_port = "9092"
      to_port   = "9092"
      protocol  = "TCP"
      ip_target = true
      targets   = data.dns_a_record_set.msk_addrs
    }
  ]
}
3. security_group.tf (安全组配置)
hcl
复制代码
# 为MSK设置安全组，允许从内部VPC访问
module "msk_internal_01_sg" {
  source  = "../../modules/shared_modules/security-group"
  vpc_id  = var.internal_vpc_id

  cidr_based_ingress = [
    {
      from_port = "9092"
      to_port   = "9092"
      protocol  = "tcp"
      cidr_blocks = var.internal_nonnroutable_vpc_cidr_block
    }
  ]
}
4. secrets_manager.tf (Secrets Manager配置)
hcl
复制代码
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
  secret_string = var.msk_password
}
5. peering_routes.tf (VPC Peering路由配置)
hcl
复制代码
# 为MSK和NLB之间的Peering连接设置路由
module "vpc_peering_routes" {
  source            = "../../modules/shared_modules/vpc-peering-routes"
  source_vpc_id     = var.internal_vpc_id
  destination_vpc_id = var.idmz_vpc_id
  route_table_ids   = var.internal_route_table_ids

  routes = [
    {
      destination_cidr_block = var.msk_vpc_cidr_block
      target                 = var.peering_connection_id
    },
    {
      destination_cidr_block = var.internal_nonnroutable_vpc_cidr_block
      target                 = var.peering_connection_id
    }
  ]
}
6. variables.tf (变量声明)
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
7. env_var/dev.tfvars (变量文件)
hcl
复制代码
# MSK NLB配置变量
internal_vpc_id        = "vpc-xxxxxxxx"
internal_subnet_ids    = ["subnet-xxxxxx", "subnet-xxxxxx"]
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
模块化: 每个主要组件（NLB、安全组、Secrets Manager、Peering Routes）都放在了单独的.tf文件中，方便管理和调试。

主文件: main.tf 文件汇总了所有模块，可以作为统一入口，方便调用不同的配置模块。

变量声明: variables.tf 用于声明所有需要的变量，这些变量通过 dev.tfvars 进行定义。
