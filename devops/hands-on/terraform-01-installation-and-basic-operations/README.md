# Hands-on Terraform-01 : Terraform Installation and Basic Operations

Purpose of the this hands-on training is to give students the knowledge of basic operations in Terraform.

## Learning Outcomes

At the end of the this hands-on training, students will be able to;

- Install Terraform

- Build AWS Infrastructure with Terraform

## Outline

- Part 1 - Install Terraform

- Part 2 - Build AWS Infrastructure with Terraform

- Part 3 - Use Docker with Terraform

## Part 1 - Install Terraform

- Launch an EC2 instance using the Amazon Linux 2 AMI with security group allowing SSH connections.

- Connect to your instance with SSH.

```bash
ssh -i .ssh/call-training.pem ec2-user@ec2-3-133-106-98.us-east-2.compute.amazonaws.com
```

- Update the installed packages and package cache on your instance.

```bash
$ sudo yum update -y
```

- Install yum-config-manager to manage your repositories.

```bash
$ sudo yum install -y yum-utils
```

- Use yum-config-manager to add the official HashiCorp Linux repository.

```bash
$ sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
```

- Install Terraform.

```bash
$ sudo yum -y install terraform
```

- Verify that the installation

```bash
$ terraform version
```

- list Terraform's available subcommands.

```bash
$ terraform -help
Usage: terraform [-version] [-help] <command> [args]

The available commands for execution are listed below.
The most common, useful commands are shown first, followed by
less common or more advanced commands. If you are just getting
started with Terraform, stick with the common commands. For the
other commands, please read the help and docs before usage.

Common commands:
    apply              Builds or changes infrastructure
...    
```

- Add any subcommand to terraform -help to learn more about what it does and available options.

```bash
$ terraform -help apply
```

- To enable autocomplete, run the following command and then restart your shell.

```bash
$ terraform -install-autocomplete
```

## Part 2 - Build AWS Infrastructure with Terraform

### Prerequisites

- An AWS account

- The AWS CLI installed

- Your AWS credentials configured locally.  

```bash
$ aws configure
```

> Hard-coding credentials into any Terraform configuration is not recommended, and risks secret leakage should this file ever be committed to a public version control system.

> Using aws credentials in ec2 instance is not recommended.

- So we should use iam role. 

### Create a role in IAM management console.

- Select `EC2` from `select type of trusted entity`.

- Select `AmazonEC2FullAccess` from `Attach permissions policies`.

- Name it `terraform-ec2`.

- Attach this role to your ec2 instance.

### Write configuration

- The set of files used to describe infrastructure in Terraform is known as a Terraform configuration. You'll write your first configuration now to launch a single AWS EC2 instance.

- Each configuration should be in its own directory. Create a directory for the new configuration.

```bash
$ mkdir terraform-aws-instance
```

- Change into the directory.

```bash
$ cd terraform-aws-instance
```

- Create a file named `aws-sample.tf` for the configuration code.

```json
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 2.70"
    }
  }
}

provider "aws" {
  profile = "default"
  region  = "us-east-1"
}

resource "aws_instance" "sample-resource" {
  ami           = "ami-09d95fab7fff3776c"
  instance_type = "t2.micro"
}
```

- Explain the each block via the following section.

### Terraform Block

The terraform {} block is required so Terraform knows which provider to download from the Terraform Registry. In the configuration above, the `aws` provider's source is defined as `hashicorp/aws` which is shorthand for `registry.terraform.io/hashicorp/aws`.

You can also assign a version to each provider defined in the `required_providers` block. The `version` argument is optional, but recommended. It is used to constrain the provider to a specific version or a range of versions in order to prevent downloading a new provider that may possibly contain breaking changes. If the version isn't specified, Terraform will automatically download the most recent provider during initialization.

### Providers

The `provider` block configures the named provider, in our case `aws`, which is responsible for creating and managing resources. A provider is a plugin that Terraform uses to translate the API interactions with the service. A provider is responsible for understanding API interactions and exposing resources. Because Terraform can interact with any API, you can represent almost any infrastructure type as a resource in Terraform.

The `profile` attribute in your provider block refers Terraform to the AWS credentials stored in your AWS Config File, which you created when you configured the AWS CLI. HashiCorp recommends that you never hard-code credentials into `*.tf configuration files`. 

> Note: If you leave out your AWS credentials, Terraform will automatically search for saved API credentials (for example, in ~/.aws/credentials) or IAM instance profile credentials. This is cleaner when .tf files are checked into source control or if there is more than one admin user.

### Resources

The `resource` block defines a piece of infrastructure. A resource might be a physical component such as an EC2 instance.

The resource block has two strings before the block: the resource type and the resource name. In the example, the resource type is `aws_instance` and the name is `sample-resource`. The prefix of the type maps to the provider. In our case "aws_instance" automatically tells Terraform that it is managed by the "aws" provider.

The arguments for the resource are within the resource block. The arguments could be things like machine sizes, disk image names, or VPC IDs. You can see (providers reference)[https://www.terraform.io/docs/providers/index.html] documents the required and optional arguments for each resource provider. For your EC2 instance, you specified an AMI for `Amazon Linux 2`, and requested a `t2.micro` instance so you qualify under the free tier.

### Initialize the directory

When you create a new configuration — or check out an existing configuration from version control — you need to initialize the directory with `terraform init`.

Terraform uses a plugin-based architecture to support hundreds of infrastructure and service providers. Initializing a configuration directory downloads and installs providers used in the configuration, which in this case is the `aws provider`. Subsequent commands will use local settings and data during initialization.

- Initialize the directory.

```bash
$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/aws versions matching "~> 2.70"...
- Installing hashicorp/aws v2.70.0...
- Installed hashicorp/aws v2.70.0 (signed by HashiCorp)

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

Terraform downloads the `aws` provider and installs it in a hidden subdirectory of the current working directory. The output shows which version of the plugin was installed.

### Format and validate the configuration

It is recommend using consistent formatting in files and modules written by different teams. The terraform fmt command automatically updates configurations in the current directory for easy readability and consistency.

Format your configuration. Terraform will return the names of the files it formatted. In this case, your configuration file was already formatted correctly, so Terraform won't return any file names.

- Change the aws-sample.tf file. (You can remove `}` from last line.)

- Format your configuration. 

```bash
$ terraform fmt

Error: Argument or block definition required

  on aws-sample.tf line 19, in resource "aws_instance" "sample-resource":

An argument or block definition is required here.
```

- Correct the aws-sample.tf file.

If you are copying configuration snippets or just want to make sure your configuration is syntactically valid and internally consistent, the built in terraform validate command will check and report errors within modules, attribute names, and value types.

Validate your configuration. If your configuration is valid, Terraform will return a success message.

```bash
$ terraform validate
Success! The configuration is valid.
```

### Create infrastructure

- run terraform apply. You should see an output similar to the one shown below. (Explain the output.)

```bash
$ terraform apply

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.sample-resource will be created
  + resource "aws_instance" "sample-resource" {
      + ami                          = "ami-09d95fab7fff3776c"
      + arn                          = (known after apply)
      + associate_public_ip_address  = (known after apply)
      + availability_zone            = (known after apply)
      + cpu_core_count               = (known after apply)
      + cpu_threads_per_core         = (known after apply)
      + get_password_data            = false
      + host_id                      = (known after apply)
      + id                           = (known after apply)
      + instance_state               = (known after apply)
      + instance_type                = "t2.micro"
      + ipv6_address_count           = (known after apply)
      + ipv6_addresses               = (known after apply)
      + key_name                     = (known after apply)
      + network_interface_id         = (known after apply)
      + outpost_arn                  = (known after apply)
      + password_data                = (known after apply)
      + placement_group              = (known after apply)
      + primary_network_interface_id = (known after apply)
      + private_dns                  = (known after apply)
      + private_ip                   = (known after apply)
      + public_dns                   = (known after apply)
      + public_ip                    = (known after apply)
      + security_groups              = (known after apply)
      + source_dest_check            = true
      + subnet_id                    = (known after apply)
      + tenancy                      = (known after apply)
      + volume_tags                  = (known after apply)
      + vpc_security_group_ids       = (known after apply)

      + ebs_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + snapshot_id           = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }

      + ephemeral_block_device {
          + device_name  = (known after apply)
          + no_device    = (known after apply)
          + virtual_name = (known after apply)
        }

      + metadata_options {
          + http_endpoint               = (known after apply)
          + http_put_response_hop_limit = (known after apply)
          + http_tokens                 = (known after apply)
        }

      + network_interface {
          + delete_on_termination = (known after apply)
          + device_index          = (known after apply)
          + network_interface_id  = (known after apply)
        }

      + root_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

This output shows the execution plan, describing which actions Terraform will take in order to change real infrastructure to match the configuration.

Terraform will now pause and wait for your approval before proceeding. If anything in the plan seems incorrect or dangerous, it is safe to abort here with no changes made to your infrastructure.

In this case the plan is acceptable, so type yes at the confirmation prompt to proceed. Executing the plan will take a few minutes since Terraform waits for the EC2 instance to become available.

```bash
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_instance.sample-resource: Creating...
aws_instance.sample-resource: Still creating... [10s elapsed]
aws_instance.sample-resource: Still creating... [20s elapsed]
aws_instance.sample-resource: Still creating... [30s elapsed]
aws_instance.sample-resource: Still creating... [40s elapsed]
aws_instance.sample-resource: Creation complete after 43s [id=i-080d16db643703468]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

- Visit the EC2 console to see the created EC2 instance.

### Inspect state

When you applied your configuration, Terraform wrote data into a file called terraform.tfstate. This file now contains the IDs and properties of the resources Terraform created so that it can manage or destroy those resources going forward.

- Inspect the current state using terraform show.

```bash
$ terraform show
# aws_instance.sample-resource:
resource "aws_instance" "sample-resource" {
    ami                          = "ami-09d95fab7fff3776c"
    arn                          = "arn:aws:ec2:us-east-1:846133131154:instance/i-080d16db643703468"
    associate_public_ip_address  = true
    availability_zone            = "us-east-1d"
    cpu_core_count               = 1
    cpu_threads_per_core         = 1
    disable_api_termination      = false
    ebs_optimized                = false
    get_password_data            = false
    hibernation                  = false
    id                           = "i-080d16db643703468"
    instance_state               = "running"
    instance_type                = "t2.micro"
    ipv6_address_count           = 0
    ipv6_addresses               = []
    monitoring                   = false
    primary_network_interface_id = "eni-0452ee35ecf17b5a6"
    private_dns                  = "ip-172-31-18-13.ec2.internal"
    private_ip                   = "172.31.18.13"
    public_dns                   = "ec2-54-84-88-167.compute-1.amazonaws.com"
    public_ip                    = "54.84.88.167"
    security_groups              = [
        "default",
    ]
    source_dest_check            = true
    subnet_id                    = "subnet-bce854f1"
    tenancy                      = "default"
    volume_tags                  = {}
    vpc_security_group_ids       = [
        "sg-3d047a10",
    ]
```

### Manually Managing State

Terraform has a built in command called `terraform state` for advanced state management. For example, if you have a long state file, you may want a list of the resources in state, which you can get by using the `list` subcommand.

```bash
$ terraform state list
aws_instance.sample-resource
```

### Change Infrastructure

- Update the ami of your instance. Change the `aws-sample.tf`.

```bash
resource "aws_instance" "sample-resource" {
  - ami           = "ami-09d95fab7fff3776c"
  + ami           = "ami-0c94855ba95c71c99"
  instance_type = "t2.micro"
}
```
so that you end up with:

```bash
terraform apply
aws_instance.sample-resource: Refreshing state... [id=i-080d16db643703468]

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # aws_instance.sample-resource must be replaced
-/+ resource "aws_instance" "sample-resource" {
      ~ ami                          = "ami-09d95fab7fff3776c" -> "ami-0c94855ba95c71c99" # forces replacement
      ~ arn                          = "arn:aws:ec2:us-east-1:846133131154:instance/i-080d16db643703468" -> (known after apply)
      ~ associate_public_ip_address  = true -> (known after apply)
      ~ availability_zone            = "us-east-1d" -> (known after apply)
      ~ cpu_core_count               = 1 -> (known after apply)
      ~ cpu_threads_per_core         = 1 -> (known after apply)
      - disable_api_termination      = false -> null
      - ebs_optimized                = false -> null
        get_password_data            = false
      - hibernation                  = false -> null
      + host_id                      = (known after apply)
      ~ id                           = "i-080d16db643703468" -> (known after apply)
      ~ instance_state               = "running" -> (known after apply)
        instance_type                = "t2.micro"
      ~ ipv6_address_count           = 0 -> (known after apply)
      ~ ipv6_addresses               = [] -> (known after apply)
      + key_name                     = (known after apply)
      - monitoring                   = false -> null
      + network_interface_id         = (known after apply)
      + outpost_arn                  = (known after apply)
      + password_data                = (known after apply)
      + placement_group              = (known after apply)
      ~ primary_network_interface_id = "eni-0452ee35ecf17b5a6" -> (known after apply)
      ~ private_dns                  = "ip-172-31-18-13.ec2.internal" -> (known after apply)
      ~ private_ip                   = "172.31.18.13" -> (known after apply)
      ~ public_dns                   = "ec2-54-84-88-167.compute-1.amazonaws.com" -> (known after apply)
      ~ public_ip                    = "54.84.88.167" -> (known after apply)
      ~ security_groups              = [
          - "default",
        ] -> (known after apply)
        source_dest_check            = true
      ~ subnet_id                    = "subnet-bce854f1" -> (known after apply)
      - tags                         = {} -> null
      ~ tenancy                      = "default" -> (known after apply)
      ~ volume_tags                  = {} -> (known after apply)
      ~ vpc_security_group_ids       = [
          - "sg-3d047a10",
        ] -> (known after apply)

      - credit_specification {
          - cpu_credits = "standard" -> null
        }

      + ebs_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + snapshot_id           = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }

      + ephemeral_block_device {
          + device_name  = (known after apply)
          + no_device    = (known after apply)
          + virtual_name = (known after apply)
        }

      ~ metadata_options {
          ~ http_endpoint               = "enabled" -> (known after apply)
          ~ http_put_response_hop_limit = 1 -> (known after apply)
          ~ http_tokens                 = "optional" -> (known after apply)
        }

      + network_interface {
          + delete_on_termination = (known after apply)
          + network_interface_id  = (known after apply)
        }

      ~ root_block_device {
          ~ delete_on_termination = true -> (known after apply)
          ~ device_name           = "/dev/xvda" -> (known after apply)
          ~ encrypted             = false -> (known after apply)
          ~ iops                  = 100 -> (known after apply)
          + kms_key_id            = (known after apply)
          ~ volume_id             = "vol-0a904aaf1ad725044" -> (known after apply)
          ~ volume_size           = 8 -> (known after apply)
          ~ volume_type           = "gp2" -> (known after apply)
        }
    }

Plan: 1 to add, 0 to change, 1 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_instance.sample-resource: Destroying... [id=i-080d16db643703468]
aws_instance.sample-resource: Still destroying... [id=i-080d16db643703468, 11s elapsed]
aws_instance.sample-resource: Still destroying... [id=i-080d16db643703468, 21s elapsed]
aws_instance.sample-resource: Still destroying... [id=i-080d16db643703468, 31s elapsed]
aws_instance.sample-resource: Destruction complete after 35s
aws_instance.sample-resource: Creating...
aws_instance.sample-resource: Still creating... [10s elapsed]
aws_instance.sample-resource: Still creating... [20s elapsed]
aws_instance.sample-resource: Still creating... [30s elapsed]
aws_instance.sample-resource: Creation complete after 31s [id=i-038b0097aff028c3b]

Apply complete! Resources: 1 added, 0 changed, 1 destroyed.
```

- Visit the EC2 console to see the changes.

### Destroy

The `terraform destroy` command terminates resources defined in your Terraform configuration. This command is the reverse of terraform apply in that it terminates all the resources specified by the configuration. It does not destroy resources running elsewhere that are not described in the current configuration.

```bash
aws_instance.sample-resource: Refreshing state... [id=i-038b0097aff028c3b]

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # aws_instance.sample-resource will be destroyed
  - resource "aws_instance" "sample-resource" {
      - ami                          = "ami-0c94855ba95c71c99" -> null
      - arn                          = "arn:aws:ec2:us-east-1:846133131154:instance/i-038b0097aff028c3b" -> null
      - associate_public_ip_address  = true -> null
      - availability_zone            = "us-east-1e" -> null
      - cpu_core_count               = 1 -> null
      - cpu_threads_per_core         = 1 -> null
      - disable_api_termination      = false -> null
      - ebs_optimized                = false -> null
      - get_password_data            = false -> null
      - hibernation                  = false -> null
      - id                           = "i-038b0097aff028c3b" -> null
      - instance_state               = "running" -> null
      - instance_type                = "t2.micro" -> null
      - ipv6_address_count           = 0 -> null
      - ipv6_addresses               = [] -> null
      - monitoring                   = false -> null
      - primary_network_interface_id = "eni-0ebe3a67493ce6e70" -> null
      - private_dns                  = "ip-172-31-60-81.ec2.internal" -> null
      - private_ip                   = "172.31.60.81" -> null
      - public_dns                   = "ec2-18-207-207-213.compute-1.amazonaws.com" -> null
      - public_ip                    = "18.207.207.213" -> null
      - security_groups              = [
          - "default",
        ] -> null
      - source_dest_check            = true -> null
      - subnet_id                    = "subnet-bc497982" -> null
      - tags                         = {} -> null
      - tenancy                      = "default" -> null
      - volume_tags                  = {} -> null
      - vpc_security_group_ids       = [
          - "sg-3d047a10",
        ] -> null

      - credit_specification {
          - cpu_credits = "standard" -> null
        }

      - metadata_options {
          - http_endpoint               = "enabled" -> null
          - http_put_response_hop_limit = 1 -> null
          - http_tokens                 = "optional" -> null
        }

      - root_block_device {
          - delete_on_termination = true -> null
          - device_name           = "/dev/xvda" -> null
          - encrypted             = false -> null
          - iops                  = 100 -> null
          - volume_id             = "vol-0318b591f60287d2e" -> null
          - volume_size           = 8 -> null
          - volume_type           = "gp2" -> null
        }
    }

Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

aws_instance.sample-resource: Destroying... [id=i-038b0097aff028c3b]
aws_instance.sample-resource: Still destroying... [id=i-038b0097aff028c3b, 10s elapsed]
aws_instance.sample-resource: Still destroying... [id=i-038b0097aff028c3b, 20s elapsed]
aws_instance.sample-resource: Destruction complete after 23s

Destroy complete! Resources: 1 destroyed.
```

- Visit the EC2 console to see the terminated EC2 instance.

## Part 3 - Use Docker with Terraform

### Prerequisites

- Docker installed

> If you do not have docker installed, you can install via following commands.

- Update the installed packages and package cache on your instance.

```bash
sudo yum update -y
```

- Install the most recent Docker Community Edition package.

```bash
sudo amazon-linux-extras install docker -y
```

- Start docker service.

```bash
sudo systemctl start docker
```

- Enable docker service so that docker service can restart automatically after reboots.

```bash
sudo systemctl enable docker
```

- Check if the docker service is up and running.

```bash
sudo systemctl status docker
```

- Add the `ec2-user` to the `docker` group to run docker commands without using `sudo`.

```bash
sudo usermod -a -G docker ec2-user
```

- Normally, the user needs to re-login into bash shell for the group `docker` to be effective, but `newgrp` command can be used activate `docker` group for `ec2-user`, not to re-login into bash shell.

```bash
newgrp docker
```

- Check the docker version without `sudo`.

```bash
docker version
```

### Use docker with terraform

- Create a directory for the new configuration and change into the directory.

```bash
$ mkdir terraform-docker && cd $_
```

- Paste the following Terraform configuration into a file and name it `main.tf`.

```json
terraform {
  required_providers {
    docker = {
      source = "terraform-providers/docker"
    }
  }
}

provider "docker" {}

resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = false
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.latest
  name  = "tutorial"
  ports {
    internal = 80
    external = 8000
  }
}
```

- Initialize the project, which downloads a plugin that allows Terraform to interact with `Docker`.

```bash
$ terraform init
```

- Provision the NGINX server container with apply. When Terraform asks you to confirm type yes and press ENTER.

```bash
$ terraform apply
```

- Verify the existence of the NGINX container by visiting `< puplic ip >:8000` in your web browser or running docker ps to see the container.

- To stop the container, run terraform destroy.

```bash
$ terraform destroy
```