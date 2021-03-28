# Terraform

## Tutorial

### 1. install terraform

아래의 hashcorp 페이지에서 최신 버전 압축파일의 다운로드링크를 복사한 뒤 `wget`명령을 통해 다운로드 받습니다.

https://www.terraform.io/downloads.html 

```bash
wget https://releases.hashicorp.com/terraform/0.14.9/terraform_0.14.9_linux_amd64.zip
```

다운로드 받은 파일을 압축 해제 합니다.

```bash
unzip terraform_0.14.9_linux_amd64.zip 
```

terraform binary를 `/usr/local/bin` 디렉터리에 복사합니다.

```
sudo mv terraform /usr/local/bin/
```

압축파일을 삭제합니다.

```
rm -rf terraform_0.14.9_linux_amd64.zip
```



### 2. terraform auto-complete

terraform 커맨드의 자동완성 기능을 활성화 시켜줍니다.

```
terraform -install-autocomplete
```



### 3. Install aws-cli

aws-cli version2를 다운로드 받습니다. -> 압축을 해제합니다 -> 설치파일을 실행합니다. 

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

압축파일 및 설치파일을 삭제합니다.

```
rm -rf aws awscliv2.zip
```



자세한 내용은 아래를 참조합니다.
https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/install-cliv2-linux.html



### 4. Configure AWS IAM

`access-key`, `secret-key`를 사용하는 Hard-coded방식이 아닌 임시 토큰으로 AWS Account에 접근하는 Assume-role 방식으로 구성 합니다.

User setting

- username: `myfit-admin`
- permission: `None`

Role setting (Administrator)

- role-name : `AdministratorRole`

- permissions : `AdministratorAccess`

- trust relationships : 

  ```
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {
          "AWS": "arn:aws:iam::242593025403:user/myfit-admin"
        },
        "Action": "sts:AssumeRole",
        "Condition": {
          "StringEquals": {
            "sts:ExternalId": "terraform-administrator"
          }
        }
      }
    ]
  }
  ```

Role setting (Poweruser)

- role-name : `PowerUserRole`

- permissions : `PowerUserAccess`

- trust relationships :

  ```
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {
          "AWS": "arn:aws:iam::242593025403:user/myfit-admin"
        },
        "Action": "sts:AssumeRole",
        "Condition": {
          "StringEquals": {
            "sts:ExternalId": "terraform-poweruser"
          }
        }
      }
    ]
  }
  ```



### 5. Configure aws-cli and credential 

`~/.aws/credentials`

```
[myfit-admin]
aws_access_key_id=[access-key]
aws_secret_access_Key=[secret-key]
```

`~/.aws/config`

```
[myfit-admin]
region=ap-northeast-2
output=json
```

`AWS_PROFILE`환경 변수를 설정하여 aws user profile을 `myfit-admin`으로 변경합니다.

```
export AWS_PROFILE=myfit-admin
```



### 6. Provision ec2 instance

`main.tf`

```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.27"
    }
  }
}

provider "aws" {
  region = "ap-northeast-2"
  assume_role {
    role_arn     = "arn:aws:iam::242593025403:role/PowerUserRole"
    session_name = "terraform-poweruser"
    external_id  = "terraform-poweruser"
  }
}

resource "aws_instance" "example" {
  ami           = "ami-830c94e3"
  instance_type = "t2.micro"

  tags = {
    Name = "ExampleInstance"
  }
}
```



### 5. initialize the directory

`main.tf` 파일이 있는 디렉터리에서 아래 명령을 수행합니다.

```bash
terraform init
```



### 6. Format and validate the configuration

tf파일의 formatting을 수행합니다. format이 HCL format으로 변경됩니다.

```
terraform fmt
```

tf파일의 validate를 수행합니다.

```
terraform validate
```



### 7. Plan and Apply

최종적으로 plan을 통해 어떤 리소스가 프로비저닝되는 지 확인합니다. 만약 변경이 일어날 시에는 변경사항에 대해서 볼 수 있습니다. cloudformation의 change-set과 같은 역할을 합니다.

```
terraform plan
```

실제로 terraform을 클라우드 환경에 적용시킵니다.

```
terraform apply
```



## Variables

변수를 미리 정의하여 사용할 수 있습니다. 중복되어 들어가는 값을 변수로 선언하여 사용하면 추후 변경이 용이합니다.

`variables.tf`

```
variable "vpc_name" {
  description = "Value of the Name tag for the VPC"
  type        = string
  default     = "jenkins"
}

variable "cidr_blocks" {
  description = "Value of the cidr_blocks for vpc and subnets"
  type        = map(any)
  default = {
    vpc              = "10.0.0.0/16"
    public-subnet-1  = "10.0.0.0/20"
    public-subnet-2  = "10.0.64.0/20"
    public-subnet-3  = "10.0.128.0/20"
    public-subnet-4  = "10.0.192.0/20"
    private-subnet-1 = "10.0.16.0/20"
    private-subnet-2 = "10.0.80.0/20"
    private-subnet-3 = "10.0.144.0/20"
    private-subnet-4 = "10.0.210.0/20"
    data-subnet-1    = "10.0.32.0/20"
    data-subnet-2    = "10.0.96.0/20"
    data-subnet-3    = "10.0.160.0/20"
    data-subnet-4    = "10.0.224.0/20"
  }
}
```

`vpc.tf`

```
resource "aws_vpc" "main" {
  cidr_block           = var.cidr_blocks.vpc
  enable_dns_hostnames = true
  tags = {
    "Name" = var.vpc_name
  }
}

resource "aws_subnet" "public-subnet-a" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.cidr_blocks.public-subnet-1
  availability_zone       = "ap-northeast-2a"
  map_public_ip_on_launch = true
  tags = {
    "Name" = "${var.vpc_name}-public-subnet-a"
  }
}

resource "aws_subnet" "public-subnet-b" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.cidr_blocks.public-subnet-2
  availability_zone       = "ap-northeast-2b"
  map_public_ip_on_launch = true
  tags = {
    "Name" = "${var.vpc_name}-public-subnet-b"
  }
}

resource "aws_subnet" "private-subnet-a" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.cidr_blocks.private-subnet-1
  availability_zone = "ap-northeast-2a"
  tags = {
    "Name" = "${var.vpc_name}-private-subnet-a"
  }
}

resource "aws_subnet" "private-subnet-b" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.cidr_blocks.private-subnet-2
  availability_zone = "ap-northeast-2b"
  tags = {
    "Name" = "${var.vpc_name}-private-subnet-b"
  }
}
```
