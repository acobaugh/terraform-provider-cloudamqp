---
layout: "cloudamqp"
page_title: "CloudAMQP: cloudamqp_instance"
description: |-
  Creates and manages a Rabbit MQ instance within CloudAMQP.
---

# cloudamqp_instance

This resource allows you to create and manage a CloudAMQP instance running Rabbit MQ and deploy to multiple cloud platforms provider and over multiple regions, see [Instance regions](../instance_region.html) for more information.

Once the instance is created it will be assigned a unique identifier. All other resource and data sources created for this instance needs to reference the instance identifier.

## Example Usage

<details>
  <summary>
    <b>
      <i>Basic example of shared and dedicated instances</i>
    </b>
  </summary>

```hcl
# Minimum free lemur instance
resource "cloudamqp_instance" "lemur_instance" {
  name = "terraform-free-instance"
  plan = "lemur"
  region = "amazon-web-services::us-west-1"
}

# New dedicated bunny instance
resource "cloudamqp_instance" "instance" {
  name              = "terraform-cloudamqp-instance"
  plan              = "bunny-1"
  region            = "amazon-web-services::us-west-1"
  tags              = ["terraform"]
  rmq_version       = "3.8.3"
  no_default_alarms = true
}
```
</details>

<details>
  <summary>
    <b>
      <i>Dedicated instance using attribute vpc_subnet to create VPC, pre v1.16.0</i>
    </b>
  </summary>

```hcl
resource "cloudamqp_instance" "instance" {
  name                = "terraform-cloudamqp-instance"
  plan                = "squirrel-1"
  region              = "amazon-web-services::us-west-1"
  tags                = ["terraform"]
  rmq_version         = "3.9.14"
  vpc_subnet          = "10.56.72.0/24"
}
```
</details>

<details>
  <summary>
    <b>
      <i>Dedicated instance using attribute vpc_subnet to create VPC and then import managed VPC, post v1.16.0 (Managed VPC)</i>
    </b>
  </summary>

```hcl
# Dedicated instance that also creates VPC
resource "cloudamqp_instance" "instance_01" {
  name                = "terraform-cloudamqp-instance-01"
  plan                = "squirrel-1"
  region              = "amazon-web-services::us-west-1"
  tags                = ["terraform"]
  rmq_version         = "3.9.14"
  vpc_subnet          = "10.56.72.0/24"
}
```

Once the instance and the VPC are created, the VPC can be imported as managed VPC and added to the configuration file.
Set attribute `vpc_id` to the managed VPC identifier. To keep the managed VPC when deleting the instance, set attribute `keep_associated_vpc` to true.
For more information see guide [Managed VPC](https://registry.terraform.io/providers/cloudamqp/cloudamqp/latest/docs/guides/info_managed_vpc#dedicated-instance-and-vpc_subnet).

```hcl
# Imported managed VPC
resource "cloudamqp_vpc" "vpc" {
  name   = "<vpc-name>"
  region = "amazon-web-services::us-east-1"
  subnet = "10.56.72.0/24"
  tags   = []
}

# Add vpc_id and keep_associated_vpc attributes
resource "cloudamqp_instance" "instance_01" {
  name                = "terraform-cloudamqp-instance-01"
  plan                = "squirrel-1"
  region              = "amazon-web-services::us-west-1"
  tags                = ["terraform"]
  rmq_version         = "3.9.14"
  vpc_id              = cloudamqp_vpc.vpc.id
  keep_associated_vpc = true
}
```
</details>

<details>
  <summary>
    <b>
      <i>Dedicated instances and managed VPC, post v1.16.0 (Managed VPC)</i>
    </b>
  </summary>

```hcl
# Managed VPC
resource "cloudamqp_vpc" "vpc" {
  name   = "<vpc-name>"
  region = "amazon-web-services::us-east-1"
  subnet = "10.56.72.0/24"
  tags   = []
}

# First instance added to managed VPC
resource "cloudamqp_instance" "instance_01" {
  name                = "terraform-cloudamqp-instance-01"
  plan                = "squirrel-1"
  region              = "amazon-web-services::us-west-1"
  tags                = ["terraform"]
  rmq_version         = "3.9.14"
  vpc_id              = cloudamqp_vpc.vpc.id
  keep_associated_vpc = true
}

# Second instance added to managed VPC
resource "cloudamqp_instance" "instance_02" {
  name                = "terraform-cloudamqp-instance-02"
  plan                = "squirrel-1"
  region              = "amazon-web-services::us-west-1"
  tags                = ["terraform"]
  rmq_version         = "3.9.14"
  vpc_id              = cloudamqp_vpc.vpc.id
  keep_associated_vpc = true
}
```

Set attribute `keep_associated_vpc` to true, will keep managed VPC when deleting the instances.
</details>

## Argument Reference

The following arguments are supported:

* `name`        - (Required) Name of the CloudAMQP instance.
* `plan`        - (Required) The subscription plan. See available [plans](../guides/info_plan.md)
* `region`      - (Required) The region to host the instance in. See [Instance regions](../guides/info_region.md)

 ***Note: Changing region will force the instance to be destroyed and a new created in the new region. All data will be lost and a new name assigned.***

* `nodes`       - (Computed/Optional) Number of nodes, 1, 3 or 5 depending on plan used.

 ***Deprecated: Legacy subscriptions plan can still change this to scale up or down the instance. New subscriptions plans use the plan to determine number of nodes. In order to change number of nodes the `plan` needs to be updated.***

* `tags`        - (Optional) One or more tags for the CloudAMQP instance, makes it possible to categories multiple instances in console view. Default there is no tags assigned.
* `rmq_version` - (Computed/Optional) The Rabbit MQ version. Can be left out, will then be set to default value used by CloudAMQP API.

 ***Note: There is not yet any support in the provider to change the RMQ version. Once it's set in the initial creation, it will remain.***

* `vpc_id`      - (Computed/Optional) The VPC ID. Use this to create your instance in an existing VPC. See available [example](../guides/info_vpc_existing.md).
* `vpc_subnet`  - (Computed/Optional) Creates a dedicated VPC subnet, shouldn't overlap with other VPC subnet, default subnet used 10.56.72.0/24.

 ***Deprecated: Will be removed in next major version (v2.0)***

 ***Note: extra fee will be charged when using VPC, see [CloudAMQP](https://cloudamqp.com) for more information.***

* `no_default_alarms`- (Computed/Optional) Set to true to discard creating default alarms when the instance is created. Can be left out, will then use default value = false.

* `keep_associated_vpc` - (Optional) Keep associated VPC when deleting instance, default set to false.

## Attributes Reference

All attributes reference are computed

* `id`      - The identifier (instance_id) for this resource, used as a reference by almost all other resource and data sources
* `url`     - The AMQP URL (uses the internal hostname if the instance was created with VPC). Has the format: `amqps://{username}:{password}@{hostname}/{vhost}`
* `apikey`  - API key needed to communicate to CloudAMQP's second API. The second API is used to manage alarms, integration and more, full description [CloudAMQP API](https://docs.cloudamqp.com/cloudamqp_api.html).
* `host`    - The external hostname for the CloudAMQP instance.
* `host_internal` - The internal hostname for the CloudAMQP instance.
* `vhost`   - The virtual host used by Rabbit MQ.

## Import

`cloudamqp_instance`can be imported using CloudAMQP internal identifier.

`terraform import cloudamqp_instance.instance <id>`

To retrieve the identifier for a VPC, either use [CloudAMQP customer API](https://docs.cloudamqp.com/#list-instances).
Or use the data source [`cloudamqp_account`](https://registry.terraform.io/providers/cloudamqp/cloudamqp/latest/docs/data-sources/account) to list all available instances for an account.
