# ectou-export

This project enables running an `Amazon Linux AMI`_ on a local `VirtualBox`_ virtual machine via `Vagrant`_.

## Goal

Preserve all the benefits of using the `Amazon Linux AMI`_ in production
while minimizing differences between `EC2`_ and local development environments.

## Usage

Examples:
```
./export.py --ami-name amzn-ami-hvm-2018.03.0.20190826-x86_64-gp2 [--vpc-name name] [--yum-proxy url]
./export.py --ami-name amzn-ami-hvm-2018.03.0.20190826-x86_64-gp2 --region ap-northeast-2
```

These examples export vagrant box files named `AMI_NAME-DATETIME.box` and `AMI_NAME-DATETIME-guest.box`.

## Overview

The `export.py` script will:
```
launch builder instance
        attach source image volume
        export-vmdk.sh (device -> vmdk)
            chroot - remove aws dependencies
            chroot - add vagrant user
            create vmdk
    download vmdk

    package-vagrant-box.sh (vmdk -> box)
        create virtualbox vm
        package vagrant box

    install-guest-additions.sh (box -> guest box)
        install guest additions
        apply security updates
        package vagrant box
```

## Dependencies

### Host software

The software has been tested using:

- VirtualBox 5.2.16
- Vagrant 2.1.2
- Python 3.6.4

  - boto3 1.7.57
  - paramiko 2.4.1
  - scp 0.11.0

Example on MacOS X host using brew:

```
brew tap caskroom/cask
brew install brew-cask
brew cask install virtualbox
brew cask install vagrant

pip install -r requirements.txt
```

### AWS account and credentials
AWS account should have default VPC or explicit VPC. Requires AWS credentials with permissions to:
```
{
  "Statement": [{
      "Effect": "Allow",
      "Action" : [
        "ec2:DescribeImages",

        "ec2:CreateKeypair",
        "ec2:DeleteKeypair",

        "ec2:CreateSecurityGroup",
        "ec2:AuthorizeSecurityGroupIngress",
        "ec2:DeleteSecurityGroup",

        "ec2:CreateVolume",
        "ec2:AttachVolume",
        "ec2:DetachVolume",
        "ec2:DeleteVolume",

        "ec2:RunInstances",
        "ec2:DescribeInstances",
        "ec2:ModifyInstanceAttribute"
        "ec2:TerminateInstances",

        "ec2:CreateTags",
      ],
      "Resource" : "*"
  }]
}
```

### Access to Amazon repositories
note::
Since the release of the Amazon Linux Container Image, the repositories are public. The yum proxy or VPN is no longer required.


The repository urls are only accessible from within the AWS environment. To access these repositories locally there are several options:

1. Use VPN connection to EC2, such as OpenVPN Access Server with Viscosity client, and route S3 prefixes over the VPN. See aws ec2 describe-prefix-lists.
1. Launch HTTP proxy in EC2 with security group restricted to your IP addresses, and configure image --yum-proxy.

## How to Use in HBsmith
1. set your aws-cli configuration
  - type `aws configure` in terminal
  - insert your `AWS Access Key ID`, `Secret Access Key` (others optional)
1. copy your AWS secret key file to root of this repository and rename it to `keypair.pem`
  - `cp ../somewhere/xxxxx.pem keypair.pem` 
1. ./export.py [options]
  - example)
    - `./export.py --ami-name amzn-ami-hvm-2018.03.0.20191219.0-x86_64-gp2 --region ap-northeast-2`

## How to Add Vagrant Box Locally
```
vagrant box add [new box name] [box file]
```

ex)

```
vagrant box add  amozonlinux-test ./amzn-ami-hvm-2018.03.0.20191219.0-x86_64-gp2-202001280940.box
```

## How to Upload Vagrant Box to Vagrant Cloud
https://blog.ycshao.com/2017/09/16/how-to-upload-vagrant-box-to-vagrant-cloud/
1. login to https://app.vagrantup.com/
1. click `New Vagrant Box` at Dashboard
1. enter name (visibility, description is optional) and click `Create box`.
1. enter version `ex) 0.0.1` and click `Create version`.
1. click `Add a proviser`
1. enter the provider name, such as `virtualbox`, select `Upload to Vagrant Cloud`, and click `Continue to upload`
1. upload file (should be `.box` file, the file you generated at `packer-templates` repository, not the folder or `.vmdk` file you made by `vagrant box add` command)
1. go back to your repository and click `Release...`
1. click `Release version`
