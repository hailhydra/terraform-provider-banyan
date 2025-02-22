---
page_title: "banyan_service_tunnel Resource - terraform-provider-banyan"
subcategory: ""
description: |-
  Resource used for lifecycle management of service tunnels. In order to properly function this resource must be utilized with the banyanaccesstier resource or banyanaccesstier2 terraform registry modules. Please see the example below and in the terraform modules for the respective cloud provider. For more information on service tunnels see the documentation https://docs.banyansecurity.io/docs/feature-guides/service-tunnels/
---

# banyan_service_tunnel (Resource)

Resource used for lifecycle management of service tunnels. In order to properly function this resource must be utilized with the banyan_accesstier resource or banyan_accesstier2 terraform registry modules. Please see the example below and in the terraform modules for the respective cloud provider. For more information on service tunnels see the documentation https://docs.banyansecurity.io/docs/feature-guides/service-tunnels/

## Example Usage
```terraform
resource "banyan_api_key" "example" {
  name        = "example api key"
  description = "example api key"
  scope       = "access_tier"
}

resource "banyan_accesstier" "example" {
  name         = "example"
  address      = "*.example.mycompany.com"
  api_key_id   = banyan_api_key.example.name
  tunnel_cidrs = ["10.10.1.0/24"]
}

resource "banyan_service_tunnel" "example" {
  name         = "example-anyone-high"
  description  = "tunnel allowing anyone with a high trust level"
  access_tiers = [banyan_accesstier.example.name]
  policy       = banyan_policy_tunnel.anyone-high.id
}

resource "banyan_policy_tunnel" "anyone-high" {
  name        = "allow anyone"
  description = "${banyan_accesstier.example.name} allow"
  access {
    roles       = ["ANY"]
    trust_level = "High"
  }
}
```

## Example Service Tunnel with L4 Policy
```terraform
terraform {
  required_providers {
    banyan = {
      source  = "github.com/banyansecurity/banyan"
      version = ">=0.9.1"
    }
  }
}

provider "banyan" {
  api_key = "igKuZugo6yH3_ig04qE8mYEeqDcSi-5s_uQr9Td0zsI"
}

resource "banyan_api_key" "example" {
  name        = "example api key"
  description = "example api key"
  scope       = "access_tier"
}

resource "banyan_accesstier" "example" {
  name         = "example"
  address      = "*.example.mycompany.com"
  api_key_id   = banyan_api_key.example.name
  tunnel_cidrs = ["10.10.0.0/16"]
}

resource "banyan_service_tunnel" "users" {
  name         = "corporate network"
  description  = "tunnel allowing anyone with a high trust level access to 443"
  access_tiers = [banyan_accesstier.example.name]
  policy       = banyan_policy_tunnel.anyone-high.id
}

resource "banyan_service_tunnel" "administrators" {
  name         = "corporate network admin"
  description  = "tunnel allowing administrators access to the networks"
  access_tiers = [banyan_accesstier.example.name]
  policy       = banyan_policy_tunnel.administrators.id
}

resource "banyan_policy_tunnel" "anyone-high" {
  name        = "corporate-network-users"
  description = "${banyan_accesstier.example.name} allow users"
  access {
    roles       = ["Everyone"]
    trust_level = "High"
    l4_access {
      allow {
        cidrs     = ["10.10.10.0/24"]
        protocols = ["TCP"]
        ports     = ["443"]
      }
    }
  }
}

resource "banyan_policy_tunnel" "administrators" {
  name        = "corporate-network-admin"
  description = "${banyan_accesstier.example.name} allow only administrators access to the entire network"
  access {
    roles       = ["Everyone"]
    trust_level = "High"
    l4_access {
      allow {
        cidrs     = ["10.10.10.0/24"]
        protocols = ["TCP"]
        ports     = ["443"]
      }
    }
  }
}
```
In this example an access tier is configured to tunnel `10.10.0.0/16`. A service tunnel is configured to utilize this access tier, and a policy is attached which only allows users with a `High` trust level access to services running on port 443 in the subnet `10.10.1.0/24`. An additional service tunnel and policy allows administrators access to the entire network behind the tunnel.

<!-- schema generated by tfplugindocs -->
## Schema

### Required

- `name` (String) Name of the service tunnel
- `policy` (String) Policy ID to be attached to this service tunnel

### Optional

- `access_tiers` (Set of String) Names of the access_tiers which the service tunnel should be associated with
- `autorun` (Boolean) Autorun for the service, if set true service would autorun on the app
- `cluster` (String, Deprecated) (Depreciated) Sets the cluster / shield for the service
- `connectors` (Set of String) Names of the connectors which the service tunnel should be associated with
- `description` (String) Description of the service tunnel
- `description_link` (String) Link shown to the end user of the banyan app for this service
- `public_cidrs_exclude` (Set of String) Specifies public IP addresses in CIDR notation that should be excluded from the tunnel, ex: 8.8.12.0/24.
- `public_cidrs_include` (Set of String) Specifies public IP addresses in CIDR notation that should be included in the tunnel, ex: 8.8.0.0/16.
- `public_domains_exclude` (Set of String) Specifies the domains that should be that should be excluded from the tunnel, ex: zoom.us
- `public_domains_include` (Set of String) Specifies the domains that should be that should be included in the tunnel, ex: cnn.com
- `public_traffic_tunnel_via_access_tier` (String) Access Tier to be used to tunnel through public traffic

### Read-Only

- `id` (String) ID of the service tunnel key in Banyan
## Import
Import is supported using the following syntax:
```shell
# For importing a resource we require resource Id, which can be obtained from console for the resource we are importing
# And we need to create an entry in .tf file which represents the resource which would be imported.
# for e.g adding an entry into main.tf
# main.tf:
# resource "banyan_service_tunnel" "myexample" {
#   name = "myexample"
# }

terraform import banyan_service_tunnel.myexample 46f3a708-2a9a-4c87-b18e-b11b6c92bf24

terraform show
# update thw show output configuration into above main.tf file, then resource is managed.
# BE CAUTIOUS before terraform apply, do terraform plan and verify there are no changes to be applied.

# Terraform Version 1.5.x or Later:
# We can create Import tf files
# for e.g
# import.tf:
# import {
#  to = banyan_service_tunnel.myexample
#  id = "46f3a708-2a9a-4c87-b18e-b11b6c92bf24"
# }
#  Then execute
terraform plan -generate-config-out=generated.tf
# Configurations are imported into generated.tf edit and verify
# BE CAUTIOUS before terraform apply, do terraform plan and verify there are no changes to be applied.
```