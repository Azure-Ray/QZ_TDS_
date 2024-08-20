# NLB for MSK instance
module "msk_nlb_proxy" {
  source  = "../../modules/shared_modules/nlb"
  environment = var.environment
  name_prefix = "msk-nlb"
  vpc_id = var.internal_vpc_id
  subnet_ids = var.internal_subnet_ids

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

# MSK Service Endpoint in VPC IDMz
module "msk_idmz_01_vpce" {
  source  = "../../modules/shared_modules/vpc-endpoint-service"
  vpc_id  = var.idmz_vpc_id
  nlb_arns = [module.msk_nlb_proxy.nlb_arn]
  tags = merge(var.hsbc_generic_tags, { "dataclassification" = "internal" })
  whitelisted_principal = var.msk_idmz_01_whitelisted_principal
}

# Security Group for MSK Endpoint VPC
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

# Route Tables for Peering Connection
module "vpc_peering_routes" {
  source            = "../../modules/shared_modules/vpc-peering-routes"
  source_vpc_id     = var.internal_vpc_id
  destination_vpc_id = var.idmz_vpc_id
  route_table_ids   = var.internal_route_table_ids

  routes = [
    {
      destination_cidr_block = var.msk_vpc_cidr_block
      target                 = var.peering_connection_id
    }
  ]
}
