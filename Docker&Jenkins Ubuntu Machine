terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region     = "ap-south-1"
  access_key = ""
  secret_key = ""
}

resource "aws_vpc" "myVpc" {
  cidr_block       = "10.0.0.0/16"
  instance_tenancy = "default"

  tags = {
    Name = "myVpc"
  }
}

resource "aws_subnet" "mySubnet" {
  vpc_id     = aws_vpc.myVpc.id
  cidr_block = "10.0.0.0/24"
  availability_zone = "ap-south-1a"

  tags = {
    Name = "mySubnet"
  }
  }

resource "aws_internet_gateway" "gw" {
  vpc_id = aws_vpc.myVpc.id

  tags = {
    Name = "my-igw"
  }
}

resource "aws_route_table" "my-rtw" {
  vpc_id = aws_vpc.myVpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.gw.id
  }
 

  tags = {
    Name = "my-rtw"
  }
}

resource "aws_security_group" "allow_tls" {
  name        = "allow_tls"
  description = "Allow TLS inbound traffic"
  vpc_id      = aws_vpc.myVpc.id

  ingress {
    description      = "TLS from VPC"
    from_port        = 80
    to_port          = 80
    protocol         = "tcp"
    cidr_blocks      = ["0.0.0.0/0"]
  }

  ingress {
    description      = "TLS from VPC"
    from_port        = 22
    to_port          = 22
    protocol         = "tcp"
    cidr_blocks      = ["0.0.0.0/0"]
  }

  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  tags = {
    Name = "allow_tls"
  }
}

resource "aws_key_pair" "kp-key" {
     key_name   = "kp-key"
     public_key = tls_private_key.rsa.public_key_openssh
    }

  resource "tls_private_key" "rsa" {
     algorithm = "RSA"
     rsa_bits  = 4096
    }

 resource "local_file" "kp-key" {
     content = tls_private_key.rsa.private_key_pem
     filename = "kp-key.pem"
    }

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]  # Canonical's AWS account ID for official Ubuntu AMIs
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-*-22.04-amd64-server-*"]  # Ubuntu 22.04 pattern for x86
  }

  filter {
    name   = "root-device-type"
    values = ["ebs"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}



resource "aws_instance" "s1" {
     ami                         = data.aws_ami.ubuntu.id
     key_name                    = aws_key_pair.kp-key.key_name
     instance_type               = "t2.micro"
     subnet_id                   =  aws_subnet.mySubnet.id
     vpc_security_group_ids      = [aws_security_group.allow_tls.id]
     associate_public_ip_address = true
     user_data = <<-EOF
        #!/bin/bash
        set -e

        # Update and install prerequisites
        apt-get update -y
        apt-get install -y apt-transport-https ca-certificates curl software-properties-common openjdk-11-jdk

        # Install Docker
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
        add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
        apt-get update -y
        apt-get install -y docker-ce
        systemctl start docker
        systemctl enable docker
        usermod -aG docker ubuntu  # Replace 'ubuntu' with the appropriate user

        # Verify Docker installation
        docker --version

        # Install Jenkins
        curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
        echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | tee /etc/apt/sources.list.d/jenkins.list
        apt-get update -y
        apt-get install -y jenkins
        systemctl start jenkins
        systemctl enable jenkins

        # Verify Jenkins installation
        systemctl status jenkins
        EOF

         tags = {
             Name = "server1"
             source = "terraform"
                }
    }
