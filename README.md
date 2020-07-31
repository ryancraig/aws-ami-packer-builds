# aws-ami-packer-builds
A project to manage Hashicorp Packer templates and assets used to build custom AWS AMIs. Currently, the following Packer build templates are available:

* [Hashicorp Consul](./consul-ami/template.json)

* [Hashicorp Vault](./vault-consul-ami/template.json)

## Getting Started
* [Hashicorp Consul Template README](./consul-ami/README.md)

* [Hashicorp Vault Template README](./vault-consul-ami/README.md)

## Credits
The [Hashicorp Consul](./consul-ami/template.json) build template is based on the work by [Gruntwork](www.gruntwork.io). The original example Packer build template resides in the [Github: hashicorp/terraform-aws-consul](https://github.com/hashicorp/terraform-aws-consul) project.

The [Hashicorp Vault](./vault-consul-ami/template.json) build template is based on the work by [Gruntwork](www.gruntwork.io). The original example Packer build template resides in the [Github: hashicorp/terraform-aws-vault](https://github.com/hashicorp/terraform-aws-vault) project.
