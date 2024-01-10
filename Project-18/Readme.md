# INFRASTRUCTURE AUTOMATION WITH IAC USING TERRAFORM (PART 2)

This is a countinuation of Project-17

In this Project, we will continue creating the resources for the AWS setup. The resources to be created include:

- 4 Private subnets
- 1 Internet Gateway
- 1 NAT Gateway
- 1 Elastic IP
- 2 Route tables
- IAM roles
- Security Groups
- Target Group for Nginx, WordPress and Tooling
- Certificate from AWS certificate manager
- External Application Load Balancer and Internal Application Load Balancer.
- Launch template for Bastion, Tooling, Nginx and WordPress
- Auto Scaling Group (ASG) for Bastion, Tooling, Nginx and WordPress
- Elastic Filesystem
- Relational Database (RDS)

# CREATE 4 PRIVATE SUBNETS AND TAGGING

We will create 4 subnets by updating the main.tf with the following code.

resource "aws_subnet" "private" {
  count                   = var.preferred_number_of_private_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_private_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index + 2)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  tags = merge(
    var.tags,
    {
      Name = format("%s-PrivateSubnet-%s", var.name, count.index)
    },
  )

}

We add the "+ 2" to the code for the count.index in the private subnets so that it doesnt overlap with the public subnets created.

Then update the vars.tf with the following for the private subnets indicating the number of subnts to be created - in our case we need to create 4 subnets.


variable "preferred_number_of_private_subnets" {
  type = 4
  description = "Number of private subnets"
}

Update our main.tf code with the following. Each section of the codes for the private and public subnets should be updated with this

tags = merge(
    var.tags,
    {
      Name = format("PrivateSubnet-%s", count.index)
    } 
  )

![Snipe 20](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/97f5e2d7-a924-4f5a-9390-717ccc5128dd)

Then update the vars.tf with the following

variable "tags" {
  description = "A mapping of tags to assign to all resources."
  type        = map(string)
  default     = {}
}

![Snipe 21](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/cfff1c6a-7b76-4a90-a5f4-6caa48dffcf0)

And the terraform.tfvars with the tags

tags = {
  Owner-Email     = "miracleanunobi80@gmail.com"
  Managed-By      = "terraform"
  Billing-Account = "153600809351"
} 

![Snipe 22](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a7f12f95-d5e4-4261-9e36-21e4b21945eb)

So our codes now looks like this -

For the main.tf

provider "aws" {
  region = var.region
}

 Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = var.vpc_cidr
  enable_dns_support             = var.enable_dns_support
  enable_dns_hostnames           = var.enable_dns_hostnames

  tags = merge(
    var.tags,
    {
      Name = format("%s-VPC", var.name)
    }
  )
}

 Get list of availability zones
data "aws_availability_zones" "available" {
  state = "available"
}

 Create public subnets
resource "aws_subnet" "public" {
  count                   = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  tags = merge(
    var.tags,
    {
      Name = format("%s-PublicSubnet-%s", var.name, count.index)
    },
  )
}

 Create private subnets
resource "aws_subnet" "private" {
  count                   = var.preferred_number_of_private_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_private_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index + 2)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  tags = merge(
    var.tags,
    {
      Name = format("%s-PrivateSubnet-%s", var.name, count.index)
    },
  )

}

The %s takes the interpolated value of var.name while the second %s takes the value of the count.index.

For the vars.tf

variable "region" {
  default = "us-east-1"
}

variable "vpc_cidr" {
  default = "172.16.0.0/16"
}

variable "enable_dns_support" {
  default = "true"
}

variable "enable_dns_hostnames" {
  default = "true"
}

variable "preferred_number_of_public_subnets" {
  type        = number
  description = "Number of public subnets"
}

variable "preferred_number_of_private_subnets" {
  type        = number
  description = "Number of private subnets"
}

variable "tags" {
  description = "A mapping of tags to assign to all resources"
  type        = map(string)
  default     = {}

}

For the terraform.tfvars

region = "us-east-1"

vpc_cidr = "10.0.0.0/16"

enable_dns_support = "true"

enable_dns_hostnames = "true"

preferred_number_of_public_subnets = 2

preferred_number_of_private_subnets = 4

tags = {
  Owner-Email     = "miracleanunobi80@gmail.com"
  Managed-By      = "terraform"
  Billing-Account = "153600809351"
} 


Now we run

$ terraform init

$ terraform plan

![Snipe 23](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f006a917-6092-458c-8eb2-ada6b6287cbe)

# Create Internet Gateway

Create an Internet Gateway in a separate Terraform file internet_gateway.tf.

resource "aws_internet_gateway" "ig" {
  vpc_id = aws_vpc.main.id
  tags = merge(
    var.tags,
    {
      Name = format("%s-%s-%s!", var.name, aws_vpc.main.id, "IG")
    },
  )

}

![Snipe 24](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/263fe178-afd0-4c50-b89c-63ca25b9bca3)

# Create NAT Gateway

We need to create an Elastic IP for the NAT Gateway before creating the NAT Gateway.

Create a file natgateway.tf and add the following code to create the Elastic IP and the NAT Gateway.

resource "aws_eip" "nat_eip" {
  vpc        = true
  depends_on = [aws_internet_gateway.ig]

  tags = merge(
    var.tags,
    {
      Name = format("%s-EIP", var.name)
    },
  )
}

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat_eip.id
  subnet_id     = element(aws_subnet.public.*.id, 0)
  depends_on    = [aws_internet_gateway.ig]

  tags = merge(
    var.tags,
    {
      Name = format("%s-Nat", var.name)
    },
  )
}

![Snipe 25](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a3c78118-095e-428b-b7e6-67e11deb8333)

# AWS Routes

Create a file called route_tables.tf and use it to create routes for both public and private subnets.

 create private route table
resource "aws_route_table" "private-rtb" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-Private-Route-Table", var.name)
    },
  )
}

 associate all private subnets to the private route table
resource "aws_route_table_association" "private-subnets-assoc" {
  count          = length(aws_subnet.private[*].id)
  subnet_id      = element(aws_subnet.private[*].id, count.index)
  route_table_id = aws_route_table.private-rtb.id
}

 create route table for the public subnets
resource "aws_route_table" "public-rtb" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-Public-Route-Table", var.name)
    },
  )
}

 create route for the public route table and attach the internet gateway
resource "aws_route" "public-rtb-route" {
  route_table_id         = aws_route_table.public-rtb.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.ig.id
}

 associate all public subnets to the public route table
resource "aws_route_table_association" "public-subnets-assoc" {
  count          = length(aws_subnet.public[*].id)
  subnet_id      = element(aws_subnet.public[*].id, count.index)
  route_table_id = aws_route_table.public-rtb.id
}

Now if you run

$ terraform validate

$ terraform plan

$ terraform apply

This will add the following resources to AWS in multi-az set up:

- Our vpc
- 2 Public subnets
- 4 Private subnets
- 1 Internet Gateway
- 1 NAT Gateway
- 1 Elastic IP
- 2 Route tables (private and public)

![Snipe 26](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/955d482d-988e-4f20-a828-07cade843f71)

![Snipe 27](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/66d28276-b9f7-4424-921f-0e8d8bacc3eb)

If everything is ok we run

$ terraform apply

![Snipe 28](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a199c777-46aa-43ed-beaf-bff211dbb904)

![Snipe 29](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/ce2cd766-b121-49b8-ac92-2e498e3770c5)

![Snipe 30](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/2d317314-e6be-4a03-9928-1d392f818ab7)

# AWS Identity and Access Management

# IAM and Roles

We want to pass an IAM role our EC2 instances to give them access to some specific resources, so we need to do the following:

# Create AssumeRole

Assume Role uses Security Token Service (STS) API that returns a set of temporary security credentials that you can use to access AWS resources that you might not normally have access to. These temporary credentials consist of an access key ID, a secret access key, and a security token. Typically, you use AssumeRole within your account or for cross-account access.

Add the following code to a new file named roles.tf

resource "aws_iam_role" "ec2_instance_role" {
name = "ec2_instance_role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Sid    = ""
        Principal = {
          Service = "ec2.amazonaws.com"
        }
      },
    ]
  })

  tags = merge(
    var.tags,
    {
      Name = "aws assume role"
    },
  )
}

![Snipe 31](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/187e53b5-e27d-4f68-9247-f40ed8a2eb5f)

In this code we are creating AssumeRole with AssumeRole policy. It grants to an entity - in our case it is an EC2, permissions to assume the role.

Create IAM policy for this role.

This is where we need to define a required policy (i.e., permissions) according to our requirements. For example, allowing an IAM role to perform action describe applied to EC2 instances:

resource "aws_iam_policy" "policy" {
  name        = "ec2_instance_policy"
  description = "A test policy"
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = [
          "ec2:Describe*",
        ]
        Effect   = "Allow"
        Resource = "*"
      },
    ]

  })

  tags = merge(
    var.tags,
    {
      Name =  "aws assume policy"
    },
  )

}

Attach the Policy to the IAM Role

This is where, we will be attaching the policy which we created above, to the role we created in the first step.

resource "aws_iam_role_policy_attachment" "test-attach" {
  role       = aws_iam_role.ec2_instance_role.name
  policy_arn = aws_iam_policy.policy.arn
}

Create an Instance Profile_ and interpolate the IAM Role

resource "aws_iam_instance_profile" "ip" {
  name = "aws_instance_profile_test"
  role =  aws_iam_role.ec2_instance_role.name
}


# CREATE SECURITY GROUPS

Create a file and name it secgrp.tf, copy and paste the code below

# security group for alb, to allow acess from any where for HTTP and HTTPS traffic
resource "aws_security_group" "ext-lb-sg" {
  name        = "ext-lb-sg"
  vpc_id      = aws_vpc.main.id
  description = "Allow TLS inbound traffic"

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTPS"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }


  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "ext-lb-sg"
    },
  )

}



# security group for bastion, to allow access into the bastion host from you IP
resource "aws_security_group" "bastion_sg" {
  name        = "bastion_sg"
  vpc_id      = aws_vpc.main.id
  description = "Allow incoming HTTP connections."

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "Bastion-SG"
    },
  )
}

#security group for nginx reverse proxy, to allow access only from the external load balancer and bastion instance
resource "aws_security_group" "nginx-sg" {
  name   = "nginx-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "nginx-SG"
    },
  )
}

resource "aws_security_group_rule" "inbound-nginx-http" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.ext-lb-sg.id
  security_group_id        = aws_security_group.nginx-sg.id
}

resource "aws_security_group_rule" "inbound-bastion-ssh" {
  type                     = "ingress"
  from_port                = 22
  to_port                  = 22
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.nginx-sg.id
}

# security group for ialb, to have acces only from nginx reverser proxy server
resource "aws_security_group" "int-lb-sg" {
  name   = "my-lb-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "int-lb-sg"
    },
  )

}

resource "aws_security_group_rule" "inbound-ilb-https" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.nginx-sg.id
  security_group_id        = aws_security_group.int-lb-sg.id
}

# security group for webservers, to have access only from the internal load balancer and bastion instance
resource "aws_security_group" "webserver-sg" {
  name   = "webserver-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "webserver-sg"
    },
  )

}

resource "aws_security_group_rule" "inbound-web-https" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.int-lb-sg.id
  security_group_id        = aws_security_group.webserver-sg.id
}

resource "aws_security_group_rule" "inbound-web-ssh" {
  type                     = "ingress"
  from_port                = 22
  to_port                  = 22
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.webserver-sg.id
}

# security group for datalayer to alow traffic from websever on nfs and mysql port and bastiopn host on mysql port
resource "aws_security_group" "datalayer-sg" {
  name   = "datalayer-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "datalayer-sg"
    },
  )
}

resource "aws_security_group_rule" "inbound-nfs-port" {
  type                     = "ingress"
  from_port                = 2049
  to_port                  = 2049
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.webserver-sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}

resource "aws_security_group_rule" "inbound-mysql-bastion" {
  type                     = "ingress"
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}

resource "aws_security_group_rule" "inbound-mysql-webserver" {
  type                     = "ingress"
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.webserver-sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}

We use the aws_security_group_rule to refrence another security group in a security group.

# CREATE CERTIFICATE FROM AMAZON CERTIFICATE MANAGER (ACM)

Create cert.tf file and add the following code snippets to it. This entire section will create a certificate, public zone, and validate the certificate using DNS method

# The entire section create a certiface, public zone, and validate the certificate using DNS method

# Create the certificate using a wildcard for all the domains created in mytoolz
resource "aws_acm_certificate" "mirahkeyz" {
  domain_name       = "*.mirahkeyz.xyz"
  validation_method = "DNS"
}

# calling the hosted zone
data "aws_route53_zone" "mirahkeyz" {
  name         = "mirahkeyz.xyz"
  private_zone = false
}

# selecting validation method
resource "aws_route53_record" "mirahkeyz" {
  for_each = {
    for dvo in aws_acm_certificate.mirahkeyz.domain_validation_options : dvo.domain_name => {
      name   = dvo.resource_record_name
      record = dvo.resource_record_value
      type   = dvo.resource_record_type
    }
  }

  allow_overwrite = true
  name            = each.value.name
  records         = [each.value.record]
  ttl             = 60
  type            = each.value.type
  zone_id         = data.aws_route53_zone.mirahkeyz.zone_id
}

# validate the certificate through DNS method
resource "aws_acm_certificate_validation" "mirahkeyz" {
  certificate_arn         = aws_acm_certificate.mirahkeyz.arn
  validation_record_fqdns = [for record in aws_route53_record.mirahkeyz : record.fqdn]
}

# create records for tooling
resource "aws_route53_record" "tooling" {
  zone_id = data.aws_route53_zone.mirahkeyz.zone_id
  name    = "tooling.mirahkeyz.tk"
  type    = "A"

  alias {
    name                   = aws_lb.ext-lb.dns_name
    zone_id                = aws_lb.ext-lb.zone_id
    evaluate_target_health = true
  }
}

# create records for wordpress
resource "aws_route53_record" "wordpress" {
  zone_id = data.aws_route53_zone.mirahkeyz.zone_id
  name    = "wordpress.mirahkeyz.tk"
  type    = "A"

  alias {
    name                   = aws_lb.ext-lb.dns_name
    zone_id                = aws_lb.ext-lb.zone_id
    evaluate_target_health = true
  }
}

# Create an external (Internet facing) Application Load Balancer (ALB)

Create a file called alb.tf

First of all we will create the ALB, then we create the target group and lastly we will create the listener rule. We need to create an ALB to balance the traffic between the Instances:

resource "aws_lb" "ext-alb" {
  name     = "ext-alb"
  internal = false
  security_groups = [
    aws_security_group.ext-alb-sg.id,
  ]

  subnets = [
    aws_subnet.public[0].id,
    aws_subnet.public[1].id
  ]

   tags = merge(
    var.tags,
    {
      Name = "narbyd-ext-ALB"
    },
  )

  ip_address_type    = "ipv4"
  load_balancer_type = "application"
}

To inform the ALB to where to route the traffic, we need to create a Target Group to point to its targets:

resource "aws_lb_target_group" "nginx-tgt" {
  health_check {
    interval            = 10
    path                = "/healthstatus"
    protocol            = "HTTPS"
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }
  name        = "nginx-tgt"
  port        = 443
  protocol    = "HTTPS"
  target_type = "instance"
  vpc_id      = aws_vpc.narbyd-vpc.id
}

Then we will need to create a Listner for this target Group

resource "aws_lb_listener" "nginx-listner" {
  load_balancer_arn = aws_lb.ext-alb.arn
  port              = 443
  protocol          = "HTTPS"
  certificate_arn   = aws_acm_certificate_validation.narbyd-acm-v.certificate_arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.nginx-tgt.arn
  }
}

Add the following outputs to output.tf to print them on screen

output "alb_dns_name" {
  value = aws_lb.ext-alb.dns_name
}

output "alb_target_group_arn" {
  value = aws_lb_target_group.nginx-tgt.arn
}

# Create an Internal Application Load Balancer (ALB)

The same concepts used to create the external load balancer will be used to create the internal load balancer_.

Add the code snippets inside the alb.tf file.

# create an ALB to balance the traffic between the Instances

resource "aws_lb" "ext-lb" {
  name     = "ext-lb"
  internal = false
  security_groups = [
    aws_security_group.ext-lb-sg.id,
  ]

  subnets = [
    aws_subnet.public[0].id,
    aws_subnet.public[1].id
  ]

  tags = merge(
    var.tags,
    {
      Name = "TCS-ext-lb"
    },
  )

  ip_address_type    = "ipv4"
  load_balancer_type = "application"
}


# ----------------------------
#Internal Load Balancers for webservers
#---------------------------------

resource "aws_lb" "ilb" {
  name     = "ilb"
  internal = true
  security_groups = [
    aws_security_group.int-lb-sg.id,
  ]

  subnets = [
    aws_subnet.private[0].id,
    aws_subnet.private[1].id
  ]

  tags = merge(
    var.tags,
    {
      Name = "TCS-int-lb"
    },
  )

  ip_address_type    = "ipv4"
  load_balancer_type = "application"
}





# Create Target group to point its targets

resource "aws_lb_target_group" "nginx-tgt" {
  health_check {
    interval            = 10
    path                = "/healthstatus"
    protocol            = "HTTPS"
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }
  name        = "nginx-tgt"
  port        = 443
  protocol    = "HTTPS"
  target_type = "instance"
  vpc_id      = aws_vpc.main.id
}

# --- target group  for wordpress -------

resource "aws_lb_target_group" "wordpress-tgt" {
  health_check {
    interval            = 10
    path                = "/healthstatus"
    protocol            = "HTTPS"
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }

  name        = "wordpress-tgt"
  port        = 443
  protocol    = "HTTPS"
  target_type = "instance"
  vpc_id      = aws_vpc.main.id
}

# --- target group for tooling -------

resource "aws_lb_target_group" "tooling-tgt" {
  health_check {
    interval            = 10
    path                = "/healthstatus"
    protocol            = "HTTPS"
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }

  name        = "tooling-tgt"
  port        = 443
  protocol    = "HTTPS"
  target_type = "instance"
  vpc_id      = aws_vpc.main.id
}




# create a Listner for the target Group

resource "aws_lb_listener" "nginx-listner" {
  load_balancer_arn = aws_lb.ext-lb.arn
  port              = 443
  protocol          = "HTTPS"
  certificate_arn   = aws_acm_certificate_validation.mirahkeyz.certificate_arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.nginx-tgt.arn
  }
}


# For this aspect a single listener was created for the wordpress which is default,
# A rule was created to route traffic to tooling when the host header changes

resource "aws_lb_listener" "web-listener" {
  load_balancer_arn = aws_lb.ilb.arn
  port              = 443
  protocol          = "HTTPS"
  certificate_arn   = aws_acm_certificate_validation.mirahkeyz.certificate_arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.wordpress-tgt.arn
  }
}

# listener rule for tooling target

resource "aws_lb_listener_rule" "tooling-listener" {
  listener_arn = aws_lb_listener.web-listener.arn
  priority     = 99

  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.tooling-tgt.arn
  }

  condition {
    host_header {
      values = ["tooling.mytoolz.tk"]
    }
  }
}

# CREATING AUTOSCALING GROUPS
We need to configure our ASG to be able to scale out and in the EC2 instances depending on the application traffic.

Before configuring ASG, we need to create the launch template and the the AMI needed. For now we are going to use a random AMI from AWS then in the next project we will use Packer to create AMI.

From the Architetcture, we need Auto Scaling Groups for bastion, nginx, wordpress and tooling.

We will create two files

asg-bastion-nginx.tf which will contain Launch Template and Auto scaling group for Bastion and Nginx
asg-wordpress-tooling.tf which will contain Launch Template and Austoscaling group for wordpress and tooling.
Create asg-bastion-nginx.tf and paste all the code snippet below;





































































































































































































































































