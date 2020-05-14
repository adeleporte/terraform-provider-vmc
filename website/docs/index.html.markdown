---
layout: "vmc"
page_title: "Provider: VMC"
sidebar_current: "docs-vmc-index"
description: |-
  The Terraform Provider for VMware Cloud
---

# VMware Cloud on AWS Provider

The VMware Cloud on AWS provider can be used to configure hybrid cloud infrastructure using the resources supported by VMware Cloud on AWS.


More information on VMC can be found on the [VMC Product Page](https://cloud.vmware.com/vmc-aws)

Please use the navigation to the left to read about available data sources and
resources.

## Basic Configuration of the VMware Cloud on AWS Provider

In order to use the provider you need to obtain the authentication
token from the Cloud Service Provider by providing the org scoped API token.
The provider client uses Cloud Service Provider (CSP) API
to exchange this org scoped API token for user access token.

Note that in all of the examples you will need to update the `api_token` and `org_id` settings
in the variables.tf file to match those configured in your VMC environment.


## Argument Reference

The following arguments are used to configure the VMware Cloud on AWS Provider:

* `api_token` - (Required) API token is used to authenticate when calling VMware Cloud Services APIs.
   This token is scoped within the organization.
*  `org_id` - (Required) Organization Identifier.
*  `vmc_url` - (Required) VMware Cloud on AWS URL.
*  `csp_url` - (Required) Cloud Service Provider URL.

#### Example main.tf file

This file will define the logical topology that Terraform will
create in VMC.

```hcl
#
# The first step is to configure the provider to connect to Cloud Service
# Provider.

provider "vmc" {
  refresh_token = var.api_token
  org_id = var.org_id
}

data "vmc_connected_accounts" "my_accounts" {
  account_number = var.aws_account_number
}

data "vmc_customer_subnets" "my_subnets" {
  connected_account_id = data.vmc_connected_accounts.my_accounts.ids[0]
  region               = var.sddc_region
}

resource "vmc_sddc" "sddc_1" {
  sddc_name           = var.sddc_name
  vpc_cidr            = var.vpc_cidr
  num_host            = var.sddc_num_hosts
  provider_type       = "AWS"
  region              = data.vmc_customer_subnets.my_subnets.region
  vxlan_subnet        = var.vxlan_subnet
  delay_account_link  = false
  skip_creating_vxlan = false
  sso_domain          = "vmc.local"

  deployment_type = "SingleAZ"

  account_link_sddc_config {
    customer_subnet_ids  = [data.vmc_customer_subnets.my_subnets.ids[0]]
    connected_account_id = data.vmc_connected_accounts.my_accounts.ids[0]
  }
  timeouts {
    create = "300m"
    update = "300m"
    delete = "180m"
  }
}

resource "vmc_public_ip" "public_ip_1" {
  nsxt_reverse_proxy_url = vmc_sddc.sddc_1.nsxt_reverse_proxy_url
  display_name = var.public_ip_displayname
}

```
