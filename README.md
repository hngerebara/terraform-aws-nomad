# Nomad AWS Blueprint

This repo contains a Blueprint for how to deploy a [Nomad](https://www.nomadproject.io/) cluster on 
[AWS](https://aws.amazon.com/) using [Terraform](https://www.terraform.io/). Nomad is a distributed, highly-available 
data-center aware scheduler. A Nomad cluster typically includes a small number of server nodes, which are responsible 
for being part of the [concensus protocol](https://www.nomadproject.io/docs/internals/consensus.html), and a larger 
number of client nodes, which are used for running jobs:

![Nomad architecture](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/master/_docs/architecture.png)

This Blueprint includes:

* [install-nomad](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/tree/master/modules/install-nomad): This module can be used to install Nomad. It can be used in a 
  [Packer](https://www.packer.io/) template to create a Nomad 
  [Amazon Machine Image (AMI)](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html).

* [run-nomad](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/tree/master/modules/run-nomad): This module can be used to configure and run Nomad. It can be used in a 
  [User Data](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html#user-data-shell-scripts) 
  script to fire up Nomad while the server is booting.

* [nomad-cluster](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/tree/master/modules/nomad-cluster): Terraform code to deploy a cluster of Nomad servers using an [Auto Scaling 
  Group](https://aws.amazon.com/autoscaling/).
    
  



## What's a Blueprint?

A Blueprint is a canonical, reusable, best-practices definition for how to run a single piece of infrastructure, such 
as a database or server cluster. Each Blueprint is created primarily using [Terraform](https://www.terraform.io/), 
includes automated tests, examples, and documentation, and is maintained both by the open source community and 
companies that provide commercial support. 

Instead of having to figure out the details of how to run a piece of infrastructure from scratch, you can reuse 
existing code that has been proven in production. And instead of maintaining all that infrastructure code yourself, 
you can leverage the work of the Blueprint community and maintainers, and pick up infrastructure improvements through
a version number bump.
 
 
 
## Who maintains this Blueprint?

This Blueprint is maintained by [Gruntwork](http://www.gruntwork.io/). If you're looking for help or commercial 
support, send an email to [blueprints@gruntwork.io](mailto:blueprints@gruntwork.io?Subject=Nomad%20Blueprint). 
Gruntwork can help with:

* Setup, customization, and support for this Blueprint.
* Blueprints for other types of infrastructure, such as VPCs, Docker clusters, databases, and continuous integration.
* Blueprints that meet compliance requirements, such as HIPAA.
* Consulting & Training on AWS, Terraform, and DevOps.



## How do you use this Blueprint?

Each Blueprint has the following folder structure:

* [modules](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/tree/master/modules): This folder contains the reusable code for this Blueprint, broken down into one or more modules.
* [examples](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/tree/master/examples): This folder contains examples of how to use the modules.
* [test](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/tree/master/test): Automated tests for the modules and examples.

Click on each of the modules above for more details.

<!-- TODO: update the consul-aws-blueprint URL to the final URL -->

To run a Nomad cluster, you need to deploy a small number of server nodes (typically 3), which are responsible 
for being part of the [concensus protocol](https://www.nomadproject.io/docs/internals/consensus.html), and a larger 
number of client nodes, which are used for running jobs. You must also have a [Consul](https://www.consul.io/) cluster 
deployed (see the [Consul AWS Blueprint](https://github.com/gruntwork-io/consul-aws-blueprint)) in one of the following 
configurations:

1. [Deploy Nomad and Consul in the same cluster](#deploy-nomad-and-consul-in-the-same-cluster)
1. [Deploy Nomad and Consul in separate clusters](#deploy-nomad-and-consul-in-separate-clusters)


### Deploy Nomad and Consul in the same cluster

1. Use the [install-consul 
   module](https://github.com/gruntwork-io/consul-aws-blueprint/tree/master/modules/install-consul) from the Consul AWS
   Blueprint and the [install-nomad module](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/tree/master/modules/install-nomad) from this Blueprint in a Packer template to create 
   an AMI with Consul and Nomad. 
   
   If you are just experimenting with this Blueprint, you may find it more convenient to use one of our official public AMIs:
   - [Latest Ubuntu 16 AMIs](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/tree/master/_docs/ubuntu16-ami-list.md).
   - [Latest Amazon Linux AMIs](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/tree/master/_docs/amazon-linux-ami-list.md).
   
   **WARNING! Do NOT use these AMIs in your production setup. In production, you should build your own AMIs in your own 
   AWS account.**
   
1. Deploy a small number of server nodes (typically, 3) using the [consul-cluster 
   module](https://github.com/gruntwork-io/consul-aws-blueprint/tree/master/modules/consul-cluster). Execute the 
   [run-consul script](https://github.com/gruntwork-io/consul-aws-blueprint/tree/master/modules/run-consul) and the
   [run-nomad script](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/tree/master/modules/run-nomad) on each node during boot, setting the `--server` flag in both 
   scripts.
1. Deploy as many client nodes as you need using the [nomad-cluster module](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/tree/master/modules/nomad-cluster). Execute the 
   [run-consul script](https://github.com/gruntwork-io/consul-aws-blueprint/tree/master/modules/run-consul) and the
   [run-nomad script](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/tree/master/modules/run-nomad) on each node during boot, setting the `--client` flag in both 
   scripts.

Check out the [nomad-consul-colocated-cluster example](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/tree/master/examples/nomad-consul-colocated-cluster) for working
sample code.


### Deploy Nomad and Consul in separate clusters

1. Deploy a standalone Consul cluster by following the instructions in the [Consul AWS 
   Blueprint](https://github.com/gruntwork-io/consul-aws-blueprint).
1. Use the scripts from the [install-nomad module](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/tree/master/modules/install-nomad) in a Packer template to create a Nomad AMI.
1. Deploy a small number of server nodes (typically, 3) using the [nomad-cluster module](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/tree/master/modules/nomad). Execute the    
   [run-nomad script](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/tree/master/modules/run-nomad) on each node during boot, setting the `--server` flag. You will 
   need to configure each node with the connection details for your standalone Consul cluster.   
1. Deploy as many client nodes as you need using the [nomad-cluster module](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/tree/master/modules/nomad). Execute the 
   [run-nomad script](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/tree/master/modules/run-nomad) on each node during boot, setting the `--client` flag.

Check out the [nomad-consul-separate-cluster example](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/tree/master/examples/nomad-consul-separate-cluster) for working
sample code.

 



## How do I contribute to this Blueprint?

Contributions are very welcome! Check out the [Contribution Guidelines](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/tree/master/CONTRIBUTING.md) for instructions.



## How is this Blueprint versioned?

This Blueprint follows the principles of [Semantic Versioning](http://semver.org/). You can find each new release, 
along with the changelog, in the [Releases Page](../../releases). 

During initial development, the major version will be 0 (e.g., `0.x.y`), which indicates the code does not yet have a 
stable API. Once we hit `1.0.0`, we will make every effort to maintain a backwards compatible API and use the MAJOR, 
MINOR, and PATCH versions on each release to indicate any incompatibilities. 



## License

This code is released under the Apache 2.0 License. Please see [LICENSE](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/tree/master/LICENSE) and [NOTICE](https://raw.githubusercontent.com/hashicorp/terraform-aws-nomad/tree/master/NOTICE) for more details.
