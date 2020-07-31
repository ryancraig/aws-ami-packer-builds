# Vault and Consul AMI

This directory contains the Packer build template and assets to successfully build a custom AWS AMI with Hashicorp Vault and Consul onboard. Packer employs [install-vault module](https://github.com/hashicorp/terraform-aws-vault/tree/master/modules/install-vault) from this Module and
the [install-consul](https://github.com/hashicorp/terraform-aws-consul/tree/master/modules/install-consul)
and [install-dnsmasq](https://github.com/hashicorp/terraform-aws-consul/tree/master/modules/install-dnsmasq) or the
[setup-systemd-resolved](https://github.com/hashicorp/terraform-aws-consul/tree/master/modules/setup-systemd-resolved)
modules from the Consul AWS Module with [Packer](https://www.packer.io/) to create [Amazon Machine Images
(AMIs)](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) that have Vault and Consul installed on top of:

1. Ubuntu 18.04
1. Ubuntu 16.04
1. Amazon Linux 2

You can use this AMI to deploy a [Vault cluster](https://www.vaultproject.io/) by using the [vault-cluster
module](https://github.com/hashicorp/terraform-aws-vault/tree/master/modules/vault-cluster). This Vault cluster will use Consul as its storage backend, so you can also use the
same AMI to deploy a separate [Consul server cluster](https://www.consul.io/) by using the [consul-cluster
module](https://github.com/hashicorp/terraform-aws-consul/tree/master/modules/consul-cluster).

Check out the [vault-cluster-private](https://github.com/hashicorp/terraform-aws-vault/tree/master/examples/vault-cluster-private) and
[the root example](https://github.com/hashicorp/terraform-aws-vault/tree/master/examples/root-example) examples for working sample code. For more info on Vault
installation and configuration, check out the [install-vault](https://github.com/hashicorp/terraform-aws-vault/tree/master/modules/install-vault) documentation.

## Dependencies
1. AWSCLI must be installed on the base AMI in order for run-consul to run
1. Git must be installed on the base AMI if you want to use clone commands
1. https://github.com/hashicorp/terraform-aws-consul. I recommend forking this project and changing the `github_organization` variable value in the `consul.json` build template file. By default the value is `hashicorp`. After forking the project, change the value to your Github account handle/organization handle.
1. https://github.com/hashicorp/terraform-aws-vault. I recommend forking this project and changing the `github_organization` variable value in the `consul.json` build template file. By default the value is `hashicorp`. After forking the project, change the value to your Github account handle/organization handle.

These dependencies are managed by Packer provisioners before they are used. Git and the local repo clones will be removed from the AMI before the build is complete.

## Quick start

To build the Vault and Consul AMI:

1. `git clone` this repo to your computer.

1. Install [Packer](https://www.packer.io/). *NOTE: I recommend using the devtestlabs/hashicorp-packer container image rather than installing Packer natively. This container is available on Dockerhub. `podman pull devtestlabs/hashicorp-packer:light` or `docker pull devtestlabs/hashicorp-packer:light`*

1. Configure your AWS credentials using one of the [options supported by the AWS SDK](http://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/credentials.html). Usually, the easiest option is to set the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables. Managing these environment variables in an .env file is better than specifying these secrets on the command line. `.env.example` is provided as an .env template. You can copy it and change the values for each environment variable. Please remember, DO NOT commit your .env file to your repo; add an entry to the `.gitignore` to ensure you CANNOT commit this file!

1. Use the [private-tls-cert module](https://github.com/hashicorp/terraform-aws-vault/tree/master/modules/private-tls-cert) to generate a CA cert and public and private keys for a
   TLS cert:

    1. Set the `dns_names` parameter to `vault.service.consul`. If you're using the [root
       example](https://github.com/hashicorp/terraform-aws-vault/tree/master/examples/root-example) and want a public domain name (e.g. `vault.example.com`), add that
       domain name here too.
    1. Set the `ip_addresses` to `127.0.0.1`.
    1. For production usage, you should take care to protect the private key by encrypting it (see [Using TLS
       certs](https://github.com/hashicorp/terraform-aws-vault/tree/master/modules/private-tls-cert#using-tls-certs) for more info).

1. Update the `variables` section of the `template.json` Packer template to specify the AWS region, Vault
   version, Consul version, and the paths to the TLS cert files you just generated. If you want to install Consul Enterprise or Vault Enterprise,
   skip the version variables and instead set the `consul_download_url` and `vault_download_url` to the full urls that point to the respective
   enterprise zipped packages.

1. Run `packer build template.json`. If you are running the Packer container, run `sudo podman run -it --env-file=.env.my_aws_account --mount type=bind,source=$(pwd)/vault-consul-ami,target=/mnt/vault-consul-ami,z devtestlabs/hashicorp-packer:light build /mnt/vault-consul-ami/template.json`
If you only wish to create a specific AMI variant such as the `amazon-linux-2-ami` use the `--only` option. For example, run `sudo podman run -it --env-file=.env.my_aws_account --mount type=bind,source=$(pwd)/vault-consul-ami,target=/mnt/vault-consul-ami,z devtestlabs/hashicorp-packer:light build --only=amazon-linux-2-ami /mnt/vault-consul-ami/template.json`. If you wish to override one or more variable values defined in the template file, you can specify the variable value on the commandline. For example, to override the `github_organization` variable value defined in `template.json`, run `sudo podman run -it --env-file=.env.my_aws_account --mount type=bind,source=$(pwd)/vault-consul-ami,target=/mnt/vault-consul-ami,z devtestlabs/hashicorp-packer:light build --only=amazon-linux-2-ami --var "github_organization=ryancraig" /mnt/vault-consul-ami/template.json`


When the build finishes, it will output the IDs of the new AMIs. To see how to deploy one of these AMIs, check out the
[vault-cluster-private](https://github.com/hashicorp/terraform-aws-vault/tree/master/examples/vault-cluster-private) and [the root example](https://github.com/hashicorp/terraform-aws-vault/tree/master/examples/root-example)
examples.

_**NOTE**: This packer template will build three variants of the AMI - an Ubuntu 16.04 variant, an Unbuntu 18.04 variant, and Amazon Linux 2 variant. You can restrict packer to only build one of them by using the `only` CLI arg. For example, to only build the Amazon Linux 2 AMI, run `packer build -only amazon-linux-2-ami vault-consul.json`. You can use the parameter `amazon-linux-2-ami` for the Amazon Linux 2 AMI. If you are using the Packer container, run `sudo podman run -it --env-file=.env.my_aws_account --mount type=bind,source=$(pwd)/vault-consul-ami,target=/mnt/vault-consul-ami,z devtestlabs/hashicorp-packer:light build --only=amazon-linux-2-ami /mnt/vault-consul-ami/template.json`._

## Add tags to your AMIs
If you need to add tags to your AWS AMIs, the Packer template snippet below shows how you might accomplish this:

```
{
    "ami_name": "vault-consul-amazon-linux-2-{{isotime | clean_resource_name}}-{{uuid}}",
    "ami_description": "An Amazon Linux 2 AMI with Hashicorp Vault {{user `vault_version`}} and Hashicorp Consul {{user `consul_version`}} onboard.",
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
      "purpose": "hashicorp vault server",
      "create-datetime": "{{isotime | clean_resource_name}}"
    }
  }
```

## Credits
This Packer build template is based on the work by [Gruntwork](www.gruntwork.io). The original example Packer build template resides in the [Github: hashicorp/terraform-aws-vault](https://github.com/hashicorp/terraform-aws-vault) project.

## External References

* [Github: hashicorp/terraform-aws-vault](https://github.com/hashicorp/terraform-aws-vault)

* [Github: hashicorp/terraform-aws-consul](https://github.com/hashicorp/terraform-aws-consul)

* [Dockerhub: devtestlabs/hashicorp-packer](https://hub.docker.com/repository/docker/devtestlabs/hashicorp-packer)

* [Packer](https://www.packer.io/)

* [Vault](https://www.vault.io/)

* [Consul](https://www.consul.io/)

* [Dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html)

* [install-consul](https://github.com/hashicorp/terraform-aws-consul/tree/master/modules/install-consul)

* [install-dnsmasq](https://github.com/hashicorp/terraform-aws-consul/tree/master/modules/install-dnsmasq)

* [setup-systemd-resolved](https://github.com/hashicorp/terraform-aws-consul/tree/master/modules/setup-systemd-resolved)

* [vault-cluster example](https://github.com/hashicorp/terraform-aws-vault/tree/master/examples/root-example)

* [consul-cluster example](https://github.com/hashicorp/terraform-aws-consul/tree/master/examples/root-example)

* [options supported by the AWS SDK](http://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/credentials.html)



