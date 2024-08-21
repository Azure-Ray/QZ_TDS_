为了将 S3 和 DynamoDB 用于 Terraform 状态管理和锁定，您需要在 Terraform 配置中正确设置 `terraform` 块和资源创建的代码。我将完整的示例代码整合并展示给您。

### 文件结构示例

```
/msk-terraform
  |-- main.tf
  |-- msk.tf
  |-- nlb.tf
  |-- security_group.tf
  |-- peering_routes.tf
  |-- secrets_manager.tf
  |-- variables.tf
  |-- backend.tf
/env_var
  |-- dev.tfvars
```

### 1. `backend.tf` (配置 S3 和 DynamoDB 用于状态管理)

```hcl
terraform {
  backend "s3" {
    bucket         = "your-terraform-state-bucket"  # 替换为您的S3 Bucket名称
    key            = "path/to/your/terraform.tfstate"  # 状态文件在S3中的路径
    region         = "us-west-2"  # S3 Bucket所在的区域
    dynamodb_table = "your-terraform-lock-table"  # 替换为您的DynamoDB表名称
    encrypt        = true  # 启用S3状态文件的加密
  }
}
```

### 2. `main.tf` (主文件 - 包含 S3 和 DynamoDB 资源创建)

在 `main.tf` 中添加 S3 和 DynamoDB 的资源创建部分，这样它们可以先创建，再用于状态管理。

```hcl
# 创建 S3 Bucket 用于存储 Terraform 状态
resource "aws_s3_bucket" "terraform_state" {
  bucket = "your-terraform-state-bucket"  # 替换为您的S3 Bucket名称
  acl    = "private"

  versioning {
    enabled = true
  }

  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }
}

# 创建 DynamoDB 表用于状态锁定
resource "aws_dynamodb_table" "terraform_lock" {
  name         = "your-terraform-lock-table"  # 替换为您的DynamoDB表名称
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}

# 调用其他模块
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

### 3. `variables.tf` (变量声明)
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

### 4. `env_var/dev.tfvars` (变量文件)
```hcl
# Internal VPC（用于NLB）的配置变量
internal_vpc_id        = "vpc-xxxxxxxx"
internal_subnet_ids    = ["subnet-xxxxxx", "subnet-xxxxxx", "subnet-xxxxxx"]
internal_nonnroutable_vpc_cidr_block = "10.0.0.0/16"

# IDMz VPC（用于MSK）的配置变量
idmz_vpc_id            = "vpc-yyyyyyyy"
idmz_subnet_ids        = ["subnet-yyyyyy", "subnet-yyyyyy", "subnet-yyyyyy"]
idmz_vpc_cidr_block    = "10.1.0.0/16"

# Secrets Manager和KMS配置变量
msk_password           = "YourMSKPassword"
existing_kms_key_id    = "arn:aws:kms:region:account-id:key/key-id"

# VPC Peering路由配置变量
internal_route_table_ids = ["rtb-xxxxxx", "rtb-yyyyyy"]
```

### 说明

1. **`backend.tf` 文件**：用于定义 Terraform 状态管理和锁定的后端配置，指向 S3 和 DynamoDB。
   
2. **`main.tf` 文件**：主要用于定义和创建 S3 bucket 和 DynamoDB 表，确保在执行 Terraform 时首先创建这些资源。然后，模块化地引入其他资源配置。

3. **变量声明**：在 `variables.tf` 文件中声明所有必要的变量，并在 `dev.tfvars` 文件中为这些变量赋值。

### 运行顺序

1. **初始化 Terraform**：执行 `terraform init`，它将自动设置 S3 和 DynamoDB 用于状态管理和锁定。
2. **应用配置**：使用 `terraform apply` 来创建资源，并确保状态文件安全地存储在 S3 中，锁定通过 DynamoDB 实现。
3. **状态文件管理**：在执行 Terraform 命令时，状态文件会自动存储到 S3 中，并且操作期间的状态锁定由 DynamoDB 管理。

通过这些配置，你的 Terraform 状态将得到良好的管理和版本控制，确保团队协作和基础设施管理的可靠性。如果有任何进一步的问题或需要帮助，请随时告诉我！
