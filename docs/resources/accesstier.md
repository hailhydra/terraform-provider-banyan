---
page_title: "banyan_accesstier Resource - terraform-provider-banyan"
subcategory: ""
description: |-
  The access tier resource allows for configuration of the access tier API object. We recommend utilizing the banyan-accesstier2 https://registry.terraform.io/modules/banyansecurity/banyan-accesstier2 Terraform registry module specific to your cloud provider. For more information about the access tier see the documentation https://docs.banyansecurity.io/docs/banyan-components/accesstier/
---

# banyan_accesstier (Resource)

The access tier resource allows for configuration of the access tier API object. We recommend utilizing the [banyan-accesstier2](https://registry.terraform.io/modules/banyansecurity/banyan-accesstier2) Terraform registry module specific to your cloud provider. For more information about the access tier see the [documentation](https://docs.banyansecurity.io/docs/banyan-components/accesstier/)

## Example Usage
```terraform
resource "banyan_api_key" "example" {
  name        = "example api key"
  description = "example api key"
  scope       = "access_tier"
}

resource "banyan_accesstier" "example" {
  name       = "example"
  address    = "*.example.mycompany.com"
  api_key_id = banyan_api_key.example.id
}
```

## Example Access Tier with Service Tunnel
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

<!-- schema generated by tfplugindocs -->
## Schema

### Required

- `address` (String) Public address of the access tier
- `api_key_id` (String) ID of the API key which is scoped to access tier
- `name` (String) Name of the access tier

### Optional

- `cluster` (String) Cluster / shield name in Banyan. If not provided then the cluster will be chosen automatically
- `console_log_level` (String) Controls verbosity of logs to console. Must be one of "ERR", "WARN", "INFO", "DEBUG"
- `debug_address_transparency` (Boolean) Provide client address transparency
- `debug_client_timeout` (Number) Client identification timeout
- `debug_code_flow` (Boolean) Enable or disable OpenID Connect
- `debug_cpu_profile` (String) Output file for CPU profiling; may impact performance. If empty, this is disabled
- `debug_disable_docker` (Boolean) Disable Docker monitoring
- `debug_full_server_cert_chain` (Boolean) Include non-root (intermediate) CA certs during TLS handshakes
- `debug_host_only` (Boolean) Host only mode
- `debug_http_backend_log` (Boolean) Verbose logging for HTTP backend traffic
- `debug_inactivity_timeout` (Number) HTTP inactivity timeout
- `debug_keep_alive` (Boolean) Enable TCP keepalive messages for TCP sockets handled by Netagent
- `debug_keep_count` (Number) Number of missing TCP keepalive acknowledgements before closing connection
- `debug_keep_idle` (Number) Idle time before sending a TCP keepalive
- `debug_keep_interval` (Number) Time between consecutive TCP keepalive messages
- `debug_mem_profile` (Boolean) Output file for memory profiling; may impact performance. If empty, this is disabled
- `debug_period` (Number) Interval for reporting statistics
- `debug_request_level_events` (Boolean) Generate access events at the request level
- `debug_send_zeros` (Boolean) Send all-zero data points to Shield
- `debug_service_discovery_enable` (Boolean) Enable or disable DNS and conntrack logging
- `debug_service_discovery_msg_limit` (Number) Message threshold for batch processing
- `debug_service_discovery_msg_timeout` (Number) Timeout value for service discovery batch processing
- `debug_shield_timeout` (Number) If Shield is not available, policies will be treated as if they are permissive. Zero means this is disabled.
- `debug_use_rsa` (Boolean) Netagent will generate RSA instead of ECDSA keys
- `debug_visibility_only` (Boolean) Enable or disable visibility mode. If on, Netagent will not do policy enforcement on inbound traffic
- `disable_snat` (Boolean) Disable Source Network Address Translation (SNAT)
- `enable_hsts` (Boolean) If enabled, Banyan will send the HTTP Strict-Transport-Security response header
- `event_key_rate_limiting` (Boolean) Enable rate limiting of Access Event generation based on a credit-based rate control mechanism
- `events_rate_limiting` (Boolean) Enable rate limiting of Access Event generation based on a credit-based rate control mechanism
- `file_log` (Boolean) Whether to log to file or not
- `file_log_level` (String) Controls verbosity of logs to file. Must be one of "ERR", "WARN", "INFO", "DEBUG"
- `forward_trust_cookie` (Boolean) Forward the Banyan trust cookie to upstream servers. This may be enabled if upstream servers wish to make use of information in the Banyan trust cookie.
- `infra_maximum_session_timeout` (Number) Timeout in seconds infrastructure sessions connected via the access tier
- `log_num` (Number) For file logs: Number of files to use for log rotation
- `log_size` (Number) For file logs: Size of each file for log rotation
- `src_nat_cidr_range` (String) CIDR range which source Network Address Translation (SNAT) will be disabled for
- `statsd_address` (String) Address to send statsd messages: “hostname:port” for UDP, “unix:///path/to/socket” for UDS
- `tunnel_cidrs` (Set of String) Backend CIDR Ranges that correspond to the IP addresses in your private network(s)
- `tunnel_connector_port` (Number) UDP port for connectors to associated with this access tier to utilize
- `tunnel_enable_dns` (Boolean) Enable DNS for Service Tunnels (needed to work properly with both private and public targets)
- `tunnel_private_domains` (Set of String) Any internal domains that can only be resolved on your internal network’s private DNS

### Read-Only

- `id` (String) ID of the access tier in Banyan
## Import
Import is supported using the following syntax:
```shell
# For importing a resource we require resource Id, which can be obtained from console for the resource we are importing
# And we need to create an entry in .tf file which represents the resource which would be imported.
# for e.g adding an entry into main.tf
# main.tf:
# resource "banyan_accesstier" "myexample" {
#   name = "myexample"
# }

terraform import banyan_accesstier.myexample 46f3a708-2a9a-4c87-b18e-b11b6c92bf24

terraform show
# update thw show output configuration into above main.tf file, then resource is managed.
# BE CAUTIOUS before terraform apply, do terraform plan and verify there are no changes to be applied.

# Terraform Version 1.5.x or Later:
# We can create Import tf files
# for e.g
# import.tf:
# import {
#  to = banyan_accesstier.myexample
#  id = "46f3a708-2a9a-4c87-b18e-b11b6c92bf24"
# }
#  Then execute
terraform plan -generate-config-out=generated.tf
# Configurations are imported into generated.tf edit and verify
# BE CAUTIOUS before terraform apply, do terraform plan and verify there are no changes to be applied.
```