# 1. Подготовка окружения

### подготовка aws cli 

```bash
C:\Users\Calculator>aws configure
AWS Access Key ID [****************EVPU]: ID_ID
AWS Secret Access Key [****************Nl3l]:
Default region name [eu-central-1]:
Default output format [json]: json
```


видно, что авторизация прошла успешно

```bash
C:\Users\Calculator>aws sts get-caller-identity
{
    "UserId": "AIDA...",
    "Account": "accc ID",
    "Arn": ".../user_admin_example"
}
```

команды по работе с terraform: 

`init` - подготовка рабочей директории (установка утилит, создание структуры лдокумента)

`plan` - показ плана изменений, которые собирается применить терраформ

`apply` - применение планов

`destroy` - удаления инфраструктуры

______

создаем файлы:

main.tf — для описания провайдера и ресурсов.
variables.tf — для входных переменных.
outputs.tf — для вывода результатов.

```bash
PS C:\Users\Calculator\ACI-IT\lab_05> tree /f
Структура папок тома System
Серийный номер тома: 504C-FB5F
C:.
│   README.md
│   
└───aws_proj
        main.tf
        outputs.tf
        variables.tf
```

создаем бакет

![alt text](/lab_06/photos/image.png)


`main.tf` : 

настраиваем блок для `backend` 

```hcl
terraform {
backend "s3"{
bucket = "aci-it-bucket"
key = "lab06proj/terrafotm.tfstate"
region = "us-east-1"
encrypt = "true"
}
}
```

настраиваем провайдера

```hcl

provider "aws" {
region = var.aws_reg
}

```

регион подбирается из фалйа переменных `variables.tf`:

```hcl
variable "aws_reg" {
type = string
default = "us-east-1"
}

variable "env" {
type = string
default = "dev"
}

variable {
type = string
default = "ami-0c55b159cbfafe1f0"
}
```

далее добавляем в `main` 2 новыx ресурс:

```hcl
resource "aws_instance" "web" {
ami = var.ami
instance_type = "t3.micro"
tags = {
name = "server-${var.env}"
}
}

resource "aws_s3_bucket" "bucket" {
bucket = "bucket-${var.env}"
acl = "private"
}
```

далее настраиваем выходные данные 

```hcl
output "ec2_public_ip" {
value = aws_instance.web.public_ip
}

output "s3_bucket_name" {
value = aws_s3_bucket.bucket.bucket
}
```


_________


инициализируем backend : 

```bash
aiai@PC:~/Desktop/aws-proj$ terraform init
Initializing the backend...

Successfully configured the backend "s3"! Terraform will automatically
use this backend unless the backend configuration changes.
Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Installing hashicorp/aws v6.25.0...
```


`Terraform has been successfully initialized!
`

смотрим план изменений от `terraform`:

```bash
iai@PC:~/Desktop/aws-proj$ terraform plan

Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.web will be created
  + resource "aws_instance" "web" {
      + ami                                  = "ami-0c55b159cbfafe1f0"
      + arn                                  = (known after apply)
      + associate_public_ip_address          = (known after apply)
      + availability_zone                    = (known after apply)
      + disable_api_stop                     = (known after apply)
      + disable_api_termination              = (known after apply)
      + ebs_optimized                        = (known after apply)
      + enable_primary_ipv6                  = (known after apply)
      + force_destroy                        = false
      + get_password_data                    = false
      + host_id                              = (known after apply)
      + host_resource_group_arn              = (known after apply)
      + iam_instance_profile                 = (known after apply)
      + id                                   = (known after apply)
      + instance_initiated_shutdown_behavior = (known after apply)
      + instance_lifecycle                   = (known after apply)
      + instance_state                       = (known after apply)
      + instance_type                        = "t3.micro"
      + ipv6_address_count                   = (known after apply)
      + ipv6_addresses                       = (known after apply)
      + key_name                             = (known after apply)
      + monitoring                           = (known after apply)
      + outpost_arn                          = (known after apply)
      + password_data                        = (known after apply)
      + placement_group                      = (known after apply)
      + placement_group_id                   = (known after apply)
      + placement_partition_number           = (known after apply)
      + primary_network_interface_id         = (known after apply)
      + private_dns                          = (known after apply)
      + private_ip                           = (known after apply)
      + public_dns                           = (known after apply)
      + public_ip                            = (known after apply)
      + region                               = "us-east-1"
      + secondary_private_ips                = (known after apply)
      + security_groups                      = (known after apply)
      + source_dest_check                    = true
      + spot_instance_request_id             = (known after apply)
      + subnet_id                            = (known after apply)
      + tags                                 = {
          + "name" = "server-dev"
        }
      + tags_all                             = {
          + "name" = "server-dev"
        }
      + tenancy                              = (known after apply)
      + user_data_base64                     = (known after apply)
      + user_data_replace_on_change          = false
      + vpc_security_group_ids               = (known after apply)

      + capacity_reservation_specification (known after apply)

      + cpu_options (known after apply)

      + ebs_block_device (known after apply)

      + enclave_options (known after apply)

      + ephemeral_block_device (known after apply)

      + instance_market_options (known after apply)

      + maintenance_options (known after apply)

      + metadata_options (known after apply)

      + network_interface (known after apply)

      + primary_network_interface (known after apply)

      + private_dns_name_options (known after apply)

      + root_block_device (known after apply)
    }

  # aws_s3_bucket.bucket will be created
  + resource "aws_s3_bucket" "bucket" {
      + acceleration_status         = (known after apply)
      + acl                         = "private"
      + arn                         = (known after apply)
      + bucket                      = "bucket-dev"
      + bucket_domain_name          = (known after apply)
      + bucket_prefix               = (known after apply)
      + bucket_region               = (known after apply)
      + bucket_regional_domain_name = (known after apply)
      + force_destroy               = false
      + hosted_zone_id              = (known after apply)
      + id                          = (known after apply)
      + object_lock_enabled         = (known after apply)
      + policy                      = (known after apply)
      + region                      = "us-east-1"
      + request_payer               = (known after apply)
      + tags_all                    = (known after apply)
      + website_domain              = (known after apply)
      + website_endpoint            = (known after apply)

      + cors_rule (known after apply)

      + grant (known after apply)

      + lifecycle_rule (known after apply)

      + logging (known after apply)

      + object_lock_configuration (known after apply)

      + replication_configuration (known after apply)

      + server_side_encryption_configuration (known after apply)

      + versioning (known after apply)

      + website (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + ec2_public_ip  = (known after apply)
  + s3_bucket_name = "bucket-dev"
╷
│ Warning: Argument is deprecated
│ 
│   with aws_s3_bucket.bucket,
│   on main.tf line 24, in resource "aws_s3_bucket" "bucket":
│   24: acl = "private"
│ 
│ acl is deprecated. Use the aws_s3_bucket_acl resource instead.
```

approve-им план и смотрим результаты


неверный ami, меняем

```bash
Error: creating EC2 Instance: operation error EC2: RunInstances, https response error StatusCode: 400, RequestID: f8570f45-51bd-438d-b8dd-854b077e22c9, api error InvalidAMIID.NotFound: The image id '[ami-0c55b159cbfafe1f0]' does not exist
```

```hcl
variable "ami" {
type = string
default = "ami-0a6793a25df710b06"
}
```


```bash
Plan: 2 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + ec2_public_ip  = (known after apply)
  + s3_bucket_name = "bucket-aci-it-super-dev"
aws_s3_bucket.bucket: Creating...
aws_instance.web: Creating...
aws_s3_bucket.bucket: Creation complete after 1s [id=bucket-aci-it-super-dev]
aws_instance.web: Still creating... [00m10s elapsed]
aws_instance.web: Creation complete after 13s [id=i-063adc13194f13ae4]
╷
│ Warning: Argument is deprecated
│ 
│   with aws_s3_bucket.bucket,
│   on main.tf line 24, in resource "aws_s3_bucket" "bucket":
│   24: acl = "private"
│ 
│ acl is deprecated. Use the aws_s3_bucket_acl resource instead.
│ 
│ (and 2 more similar warnings elsewhere)
╵

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

ec2_public_ip = "63.181.83.145"
s3_bucket_name = "bucket-aci-it-super-dev"

```