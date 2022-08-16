provider "aws" {
}

data "aws_availability_zones" "available" {
		  state = "available"
		}

data "aws_region" "current" {}




resource "aws_vpc" "VPC" {
  cidr_block = "10.0.0.0/16"
  enable_dns_support = true
  enable_dns_hostnames = true
}



resource "aws_subnet" "PrivateSubnet1" {
  cidr_block = "10.0.0.0/24"
  map_public_ip_on_launch = false
  vpc_id = aws_vpc.VPC.id
  availability_zone = data.aws_availability_zones.available.names[0]

  tags = {
    Name = "Private Subnet AZ A"
  }
}



resource "aws_subnet" "PrivateSubnet2" {
  cidr_block = "10.0.1.0/24"
  map_public_ip_on_launch = false
  vpc_id = aws_vpc.VPC.id
  availability_zone = data.aws_availability_zones.available.names[1]

  tags = {
    Name = "Private Subnet AZ B"
  }
}

resource "aws_route_table" "RouteTablePrivate1" {
  vpc_id = aws_vpc.VPC.id

  tags = {
    Name = "Private Route Table A"
  }
}

resource "aws_route_table_association" "AssociationForRouteTablePrivate10" {
  subnet_id = aws_subnet.PrivateSubnet1.id
  route_table_id = aws_route_table.RouteTablePrivate1.id
}



resource "aws_route_table" "RouteTablePrivate2" {
  vpc_id = aws_vpc.VPC.id

  tags = {
    Name = "Private Route Table B"
  }
}

resource "aws_route_table_association" "AssociationForRouteTablePrivate20" {
  subnet_id = aws_subnet.PrivateSubnet2.id
  route_table_id = aws_route_table.RouteTablePrivate2.id
}





resource "aws_vpc_endpoint" "VPCEndpoint" {
  vpc_id = aws_vpc.VPC.id
  service_name = "com.amazonaws.${data.aws_region.current.name}.s3"
  vpc_endpoint_type = "Gateway"
  route_table_ids = [undefined]
  policy = <<POLICY
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "*"
      ],
      "Resource": [
        "*"
      ]
    }
  ]
}
POLICY
}



resource "aws_vpc_endpoint" "VPCEndpoint" {
  vpc_id = aws_vpc.VPC.id
  service_name = "com.amazonaws.${data.aws_region.current.name}.dynamodb"
  vpc_endpoint_type = "Gateway"
  route_table_ids = [undefined]
  policy = <<POLICY
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "*"
      ],
      "Resource": [
        "*"
      ]
    }
  ]
}
POLICY
}



resource "aws_vpc_endpoint" "VPCEndpoint" {
  vpc_id = aws_vpc.VPC.id
  service_name = "com.amazonaws.${data.aws_region.current.name}.ec2"
  vpc_endpoint_type = "Interface"
  subnet_ids = [undefined]
  security_group_ids = [ aws_security_group.SgForVPCEndpoint.id ]
}

resource "aws_security_group" "SgForVPCEndpoint" {
  name = "SgForVPCEndpoint"
  description = "Security Group for VPC Endpoint"
  vpc_id = aws_vpc.VPC.id

  ingress {
    from_port = 443
    to_port = 443
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Allow HTTPS traffic to VPC Endpoint"
  }

  egress {
    from_port = 0
    to_port = 0
    protocol = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

}
