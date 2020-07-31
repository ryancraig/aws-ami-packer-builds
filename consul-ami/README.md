# Consul AMI

This directory contains the Packer build template and assets to successfully build a custom AWS AMI with Consul onboard. Packer employs [install-consul](https://github.com/hashicorp/terraform-aws-consul/tree/master/modules/install-consul) and 
either [install-dnsmasq](https://github.com/hashicorp/terraform-aws-consul/tree/master/modules/install-dnsmasq) for Ubuntu 16.04 and Amazon Linux 2 or [setup-systemd-resolved](https://github.com/hashicorp/terraform-aws-consul/tree/master/modules/setup-systemd-resolved) for Ubuntu 18.04 modules with [Packer](https://www.packer.io/) to create [Amazon Machine 
Images (AMIs)](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) that have Consul and Dnsmasq installed on 
top of:
 
1. Ubuntu 16.04
1. Ubuntu 18.04
1. Amazon Linux 2

These AMIs will have [Consul](https://www.consul.io/) installed and configured to automatically join a cluster during 
boot-up. They also have [Dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) installed and configured to use 
Consul for DNS lookups of the `.consul` domain (e.g. `foo.service.consul`) (see [registering 
services](https://www.consul.io/intro/getting-started/services.html) for instructions on how to register your services
in Consul). To see how to deploy this AMI, check out the [consul-cluster example](https://github.com/hashicorp/terraform-aws-consul/tree/master/examples/root-example). 

For more info on Consul installation and configuration, check out the 
[install-consul](https://github.com/hashicorp/terraform-aws-consul/tree/master/modules/install-consul) and [install-dnsmasq](https://github.com/hashicorp/terraform-aws-consul/tree/master/modules/install-dnsmasq) for Ubuntu 16.04 and Amazon Linux 2 or [setup-systemd-resolved](https://github.com/hashicorp/terraform-aws-consul/tree/master/modules/setup-systemd-resolved) for Ubuntu 18.04 documentation.

## Dependencies
1. AWSCLI must be installed on the base AMI in order for run-consul to run
1. Git must be installed on the base AMI if you want to use clone commands
1. https://github.com/hashicorp/terraform-aws-consul. I recommend forking this project and changing the `github_organization` variable value in the `consul.json` build template file. By default the value is `hashicorp`. After forking the project, change the value to your Github account handle/organization handle.

These dependencies are managed by Packer provisioners before they are used. Git and the local repo clone will be removed from the AMI before the build is complete.

## Quick start

To build the Consul AMI:

1. `git clone` this repo to your computer.

1. Install [Packer](https://www.packer.io/). *NOTE: I recommend using the devtestlabs/hashicorp-packer container image rather than installing Packer natively. This container is available on Dockerhub. `podman pull devtestlabs/hashicorp-packer:light` or `docker pull devtestlabs/hashicorp-packer:light`*

1. Configure your AWS credentials using one of the [options supported by the AWS SDK](http://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/credentials.html). Usually, the easiest option is to set the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables. Managing these environment variables in an .env file is better than specifying these secrets on the command line. `.env.example` is provided as an .env template. You can copy it and change the values for each environment variable. Please remember, DO NOT commit your .env file to your repo; add an entry to the `.gitignore` to ensure you CANNOT commit this file!

1. Update the `variables` section of the `template.json` Packer template to configure the AWS region, Consul version, and
   Dnsmasq version you wish to use. If you want to install Consul Enterprise, skip the version variable and instead set 
   the `download_url` to the full url that points to the consul enterprise zipped package.

1. Run `packer build template.json`. If you are running the Packer container, run `sudo podman run -it --env-file=.env.my_aws_account --mount type=bind,source=$(pwd)/consul-ami,target=/mnt/consul-ami,z devtestlabs/hashicorp-packer:light build /mnt/consul-ami/template.json`
If you only wish to create a specific AMI variant such as the `amazon-linux-2-ami` use the `--only` option. For example, run `sudo podman run -it --env-file=.env.my_aws_account --mount type=bind,source=$(pwd)/consul-ami,target=/mnt/consul-ami,z devtestlabs/hashicorp-packer:light build --only=amazon-linux-2-ami /mnt/consul-ami/template.json`. If you wish to override one or more variable values defined in the template file, you can specify the variable value on the commandline. For example, to override the `github_organization` variable value defined in `template.json`, run `sudo podman run -it --env-file=.env.my_aws_account --mount type=bind,source=$(pwd)/consul-ami,target=/mnt/consul-ami,z devtestlabs/hashicorp-packer:light build --only=amazon-linux-2-ami --var "github_organization=ryancraig" /mnt/consul-ami/template.json`

When the build finishes, it will output the IDs of the new AMI(s). To see how to deploy one of these AMIs, check out the
[consul-cluster example](https://github.com/hashicorp/terraform-aws-consul/tree/master/examples/root-example).

## Add tags to your AMIs
If you need to add tags to your AWS AMIs, the Packer template snippet below shows how you might accomplish this:

```
{
    "ami_name": "consul-amazon-linux-2-{{isotime | clean_resource_name}}-{{uuid}}",
    "ami_description": "An Amazon Linux 2 AMI with Hashicorp Consul {{user `consul_version`}} onboard.",
    "instance_type": "t2.micro",
    "name": "amazon-linux-2-ami",
    "region": "{{user `aws_region`}}",
    "type": "amazon-ebs",
    "source_ami_filter": {
      "filters": {
        "virtualization-type": "hvm",
        "architecture": "x86_64",
        "name": "*amzn2-ami-hvm-*",
        "block-device-mapping.volume-type": "gp2",
        "root-device-type": "ebs"
      },
      "owners": ["amazon"],
      "most_recent": true
    },
    "ssh_username": "ec2-user",
    "tags": {
      "builder": "hashicorp packer 1.6.0",
      "owner": "devops@example.com",
      "contact": "rcraig@example.com",
      "purpose": "hashicorp consul server",
      "create-datetime": "{{isotime | clean_resource_name}}"
    }
  }
```

## Credits
This Packer build template is based on the work by [Gruntwork](www.gruntwork.io). The original example Packer build template resides in the [Github: hashicorp/terraform-aws-consul](https://github.com/hashicorp/terraform-aws-consul) project.

## External References

* [Github: hashicorp/terraform-aws-consul](https://github.com/hashicorp/terraform-aws-consul)

* [Dockerhub: devtestlabs/hashicorp-packer](https://hub.docker.com/repository/docker/devtestlabs/hashicorp-packer)

* [Packer](https://www.packer.io/)

* [Consul](https://www.consul.io/)

* [Dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html)

* [install-consul](https://github.com/hashicorp/terraform-aws-consul/tree/master/modules/install-consul)

* [install-dnsmasq](https://github.com/hashicorp/terraform-aws-consul/tree/master/modules/install-dnsmasq)

* [setup-systemd-resolved](https://github.com/hashicorp/terraform-aws-consul/tree/master/modules/setup-systemd-resolved)

* [consul-cluster example](https://github.com/hashicorp/terraform-aws-consul/tree/master/examples/root-example)

* [options supported by the AWS SDK](http://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/credentials.html)
