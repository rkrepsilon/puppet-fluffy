# Puppet types and providers for Fluffy

#### Table of Contents

1. [Overview](#overview)
2. [Module Description - What the module does and why it is useful](#module-description)
3. [Setup - The basics of getting started with the fluffy module](#setup)
4. [Reference - Types reference and additional functionalities](#reference)
5. [Hiera integration](#hiera)
6. [Contact](#contact)

<a name="overview"/>

## Overview

This module implements native types and providers to manage Fluffy. The providers are *fully idempotent*.

<a name="module-description"/>

## Module Description

The fluffy module allows to automate the configuration and deployment of Fluffy interfaces, chains, services, addressbook, rules and rollback checks.

<a name="setup"/>

## Setup

The module requires the [fluffy-ruby](https://rubygems.org/gems/fluffy-ruby) rubygem. It also requires Puppet >= 4.10.0.

The providers make use of some language features only available in Ruby 2.x. By default, Puppet Server uses a version of the JRuby 1.7 series which conforms to MRI/Ruby language 1.9.
However, you will need to configure Puppet Server to instead use the JRuby 9k series in order to support Ruby 2.x. For more information, [read here](https://docs.puppet.com/puppetserver/latest/configuration.html#enabling-jruby-9k).

The include the main class as follows:

```
include fluffy
```

<a name="reference"/>

## Reference

### Classes

#### fluffy
`fluffy`

```
include fluffy
```

##### `opts` (optional)
Fluffy options in the form of {'option' => 'value'}.

Defaults to:
```yaml
fluffy::opts:
  max_sessions: 10
```

##### `logging_opts` (optional)
Fluffy logging options in the form of {'option' => 'value'}.

Defaults to:
```yaml
fluffy::logging_opts:
  version: 1
  disable_existing_loggers: false
  formatters:
    standard:
      format: "%{literal('%')}(asctime)s [%{literal('%')}(levelname)s] %{literal('%')}(name)s: %{literal('%')}(message)s"
  handlers:
    console:
      formatter: "standard"
      class: "logging.StreamHandler"
      stream: "ext://sys.stdout"
  loggers:
    "fluffy":
      handlers:
        - "console"
      level: "INFO"
      propagate: true
    "werkzeug":
      handlers:
        - "console"
      level: "ERROR"
```

##### `interfaces` (optional)
Fluffy interfaces in the form of {'interface' => { .. }}

Defaults to:
```yaml
fluffy::interfaces:
  loopback:
    interface: 'lo'
```

##### `addressbook` (optional)
Fluffy addressbook in the form of {'address' => { .. }}

Defaults to:
```yaml
fluffy::addressbook:
  admins:
    address: '0.0.0.0/0'
  loopback_net:
    address: '127.0.0.0/8'
```

##### `services` (optional)
Fluffy services in the form of {'service' => { .. }}

Defaults to:
```yaml
fluffy::services:
  dhcp:
    src_port:
      - '67:68'
    dst_port:
      - '67:68'
    protocol: 'udp'
  fluffy_api:
    dst_port:
      - 8676
    protocol: 'tcp'
  smtp:
    dst_port:
      - 25
    protocol: 'tcp'
  ssh:
    dst_port:
      - 22
    protocol: 'tcp'
```

##### `chains` (optional)
Fluffy chains in the form of {'table:chain' => { .. }}

Defaults to:
```yaml
fluffy::chains:
  "filter:FORWARD":
    policy: 'DROP'
  "filter:FORWARD_LOGGING":
    policy: 'ACCEPT'
  "filter:INPUT":
    policy: 'DROP'
  "filter:INPUT_LOGGING":
    policy: 'ACCEPT'
  "filter:OUTPUT":
    policy: 'ACCEPT'
```

##### `rules` (optional)
Fluffy rules in the form of {'table:chain:rule' => { .. }}.

Defaults to:
```yaml
  "filter:INPUT:invalid_state":
    order: 0
    ctstate:
      - 'INVALID'
    in_interface: 'any'
    jump: 'INPUT_LOGGING'
  "filter:FORWARD:invalid_state":
    order: 0
    ctstate:
      - 'INVALID'
    in_interface: 'any'
    jump: 'FORWARD_LOGGING'
    out_interface: 'any'
  "filter:INPUT:established":
    order: 10
    action: 'ACCEPT'
    ctstate:
      - 'ESTABLISHED'
      - 'RELATED'
    in_interface: 'any'
  "filter:FORWARD:established":
    order: 10
    action: 'ACCEPT'
    ctstate:
      - 'ESTABLISHED'
      - 'RELATED'
    in_interface: 'any'
  "filter:INPUT:antispoof":
    order: 20
    action: 'DROP'
    in_interface: 'loopback'
    negate_src_address: true
    src_address:
      - 'loopback_net'
  "filter:INPUT:loopback":
    order: 30
    action: 'ACCEPT'
    in_interface: 'loopback'
  "filter:INPUT:ssh_admins":
    order: 50
    action: 'ACCEPT'
    comment: 'Allow SSH in'
    dst_service:
      - 'ssh'
    in_interface: 'any'
    src_address:
      - 'admins'
  "filter:INPUT:fluffy_api":
    order: 50
    action: 'ACCEPT'
    comment: 'Allow access to Fluffy REST API'
    dst_service:
      - 'fluffy_api'
    in_interface: 'any'
    src_address:
      - 'admins'
  "filter:FORWARD:logging":
    order: 900
    in_interface: 'any'
    jump: 'FORWARD_LOGGING'
    out_interface: 'any'
  "filter:INPUT:logging":
    order: 900
    in_interface: 'any'
    jump: 'INPUT_LOGGING'
  "filter:FORWARD_LOGGING:logging_log":
    order: 999
    action: 'LOG'
    in_interface: 'any'
    limit: '2/min'
    log_level: 'warning'
    log_prefix: 'Fluffy CHAIN=FORWARD '
    out_interface: 'any'
  "filter:INPUT_LOGGING:logging_log":
    order: 999
    action: 'LOG'
    in_interface: 'any'
    limit: '2/min'
    log_level: 'warning'
    log_prefix: 'Fluffy CHAIN=INPUT '
```

Rule ordering can be specified by using the `order` parameter.

##### `checks` (optional)
Fluffy rollback checks in the form of {'check' => { .. }}

Defaults to:
```yaml
fluffy::checks:
  ssh:
    type: tcp
    port: 22
  fluffy_api:
    type: tcp
    port: 8676
```

##### `purge_rules` (optional)
Purge unmanaged rules. Defaults to `true`.

##### `data_dir` (optional)
Path to the Fluffy data directory. Defaults to `/var/lib/fluffy`.

##### `config_dir` (optional)
Path to the Fluffy configuration directory. Defaults to `/etc/fluffy`.

##### `config_file` (optional)
Path to the Fluffy configuration file. Defaults to `$config_dir/fluffy.yaml`.

##### `config_file_manage` (optional)
Whether we should manage Fluffy's configuration file or not. Defaults to `true`.

##### `logging_file` (optional)
Path to the Fluffy logging file. Defaults to `$config_dir/logging.yaml`.

##### `logging_file_manage` (optional)
Whether we should manage Fluffy's logging file or not. Defaults to `true`.

##### `gem_dependencies` (optional)
Rubygems dependencies for Fluffy

Defaults to:
```yaml
fluffy::gem_dependencies:
  "fluffy-ruby": {}
```

##### `packages` (optional)
Installation packages for Fluffy

Defaults to:
```yaml
fluffy::packages:
  "fluffy": {}
```

##### `service_provider` (optional)
Fluffy service provider. Can be either `default` or `docker`. Defaults to `default`.

##### `service_opts` (optional)
Fluffy service options when using `docker` as a provider.

##### `service_name` (optional)
Fluffy service name. Defaults to `fluffy`.

##### `service_manage` (optional)
Whether we should manage the service runtime or not. Defaults to `true`.

##### `service_ensure` (optional)
Whether the resource is running or not. Valid values are `running`, `stopped`. Defaults to `running`.

##### `service_enable` (optional)
Whether the service is onboot enabled or not. Defaults to `true`.

### Types

#### fluffy_chain
`fluffy_chain` manages Fluffy chains

```
fluffy_chain {"<table>:<chain>": }
```

##### `chain` (required unless specified in the resource title)
Chain name

##### `table` (required unless specified in the resource title)
Packet filtering table. Valid values are: `filter`, `nat`, `mangle`, `raw`, `security`.

##### `policy` (optional_
Default policy. Valid values are: `ACCEPT`, `DROP`, `RETURN`. Defaults to `ACCEPT`.

##### `ensure` (optional)
Whether the resource is present or not. Valid values are `present`, `absent`. Defaults to `present`.

#### fluffy_interface
`fluffy_interface` manages Fluffy interfaces

```
fluffy_interface {"interface": }
```

##### `name` (required)
Interface name

##### `interface` (optional)
The actual network interface

##### `ensure` (optional)
Whether the resource is present or not. Valid values are `present`, `absent`. Defaults to `present`.

#### fluffy_address
`fluffy_address` manages the Fluffy addressbook

```
fluffy_address {"address": }
```

##### `name` (required)
Address name

##### `address` (optional)
List of one or more addresses. It can be a reference to another address in the addressbook, a valid CIDR or an IP range.

##### `ensure` (optional)
Whether the resource is present or not. Valid values are `present`, `absent`. Defaults to `present`.

#### fluffy_service
`fluffy_service` manages the Fluffy services

```
fluffy_service {"service": }
```

##### `name` (required)
Service name

##### `src_port` (optional)
Source port(s). Ports must be between 1-65535 or a valid port range.

##### `dst_port` (optional)
Destination port(s). Ports must be between 1-65535 or a valid port range.

##### `protocol` (optional)
Network protocol. Valid values are: `ip`, `tcp`, `udp`, `icmp`, `ipv6-icmp`, `esp`, `ah`, `vrrp`, `igmp`, `ipencap`, `ipv4`, `ipv6`, `ospf`, `gre`, `cbt`, `sctp`, `pim`, `all`. Defaults to `all`.

##### `ensure` (optional)
Whether the resource is present or not. Valid values are `present`, `absent`. Defaults to `present`.

```
fluffy_rule {"table:chain:rule": }
```

##### `rule` (required unless specified in the resource title)
Rule name

##### `table` (required unless specified in the resource title)
Rule packet filtering table

##### `chain` (required unless specified in the resource title)
Rule chain name

##### `index` (optional)
Specify the rule position by index. Avoid using it in favour of the `order` parameter in Hiera.

##### `before_rule` (optional)
Specify that the rule should precede the given rule. Avoid using it in favour of the `order` parameter in Hiera.

##### `after_rule` (optional)
Specify that the rule should proceed the given rule. Avoid using it in favour of the `order` parameter in Hiera.

##### `action` (optional)
Rule action. Valid values are: `absent`, `ACCEPT`, `DROP`, `REJECT`, `QUEUE`, `RETURN`, `DNAT`, `SNAT`, `LOG`, `MASQUERADE`, `REDIRECT`, `MARK`, `TCPMSS`. Defaults to `absent`.

##### `jump` (optional)
Rule jump target. Defaults to `absent`.

##### `negate_protocol` (optional)
Negate protocol. Defaults to `false`.

##### `protocol` (optional)
Network protocol. Valid values are: `absent`, `ip`, `tcp`, `udp`, `icmp`, `ipv6-icmp`, `esp`, `ah`, `vrrp`, `igmp`, `ipencap`, `ipv4`, `ipv6`, `ospf`, `gre`, `cbt`, `sctp`, `pim`, `all`. Defaults to `absent`.

##### `negate_icmp_type` (optional)
Negate ICMP type. Defaults to `false`.

##### `icmp_type` (optional)
ICMP type. Valid values are: `absent`, `any`, `echo-reply`, `destination-unreachable`, `network-unreachable`, `host-unreachable`, `protocol-unreachable`, `port-unreachable`, `fragmentation-needed`, `source-route-failed`, `network-unknown`, `host-unknown`, `network-prohibited`, `host-prohibited`, `TOS-network-unreachable`, `TOS-host-unreachable`, `communication-prohibited`, `host-precedence-violation`, `precedence-cutoff`, `source-quench`, `redirect`, `network-redirect`, `host-redirect`, `TOS-network-redirect`, `TOS-host-redirect`, `echo-request`, `router-advertisement`, `router-solicitation`, `time-exceeded`, `ttl-zero-during-transit`, `ttl-zero-during-reassembly`, `parameter-problem`, `ip-header-bad`, `required-option-missing`, `timestamp-request`, `timestamp-reply`, `address-mask-request`, `address-mask-reply`. Defaults to `absent`.

##### `negate_tcp_flags` (optional)
Negate TCP flags. Defaults to `false`.

##### `tcp_flags` (optional)
TCP flags. Defaults to `absent`.

##### `negate_ctstate` (optional)
Negate conntrack state(s). Defaults to `false`.

##### `ctstate` (optional)
Conntrack state(s). Defaults to `[]`.

##### `negate_state` (optional)
Negate connection state(s). Defaults to `false`.

##### `state` (optional)
Connection state(s). Defaults to `[]`.

##### `negate_src_address_range` (optional)
Negate source range address(es). Defaults to `false`.

##### `src_address_range` (optional)
Source range address(es). Addresses must be valid IP ranges. Defaults to `[]`.

##### `negate_dst_address_range` (optional)
Negate destination range address(es). Defaults to `false`.

##### `dst_address_range` (optional)
Destination range address(es). Addresses must be valid IP ranges. Defaults to `[]`.

##### `negate_in_interface` (optional)
Negate input interface. Defaults to `false`.

##### `in_interface` (optional)
Input interface(s). Defaults to `[]`.

##### `negate_out_interface` (optional)
Negate output interface. Defaults to `false`.

##### `out_interface` (optional)
Output interface(s). Defaults to `[]`.

##### `negate_src_address` (optional)
Negate source address(es). Defaults to `false`.

##### `src_address` (optional)
Source address(es). Address must be valid addressbook entries. Defaults to `[]`.

##### `negate_dst_address` (optional)
Negate destination range address(es). Defaults to `false`.

##### `dst_address` (optional)
Destination address(es). Addresses must be valid addressbook entries. Defaults to `[]`.

##### `negate_src_service` (optional)
Negate source service(es). Defaults to `false`.

##### `src_service` (optional)
Source service(es). Defaults to `[]`.

##### `negate_dst_service` (optional)
Negate destination service(es). Defaults to `false`.

##### `dst_service` (optional)
Destination service(es). Defaults to `[]`.

##### `reject_with` (optional)
Reject with. Valid values are: `absent`, `icmp-net-unreachable`, `icmp-host-unreachable`, `icmp-port-unreachable`, `icmp-proto-unreachable`, `icmp-net-prohibited`, `icmp-host-prohibited`, `icmp-admin-prohibited`. Defaults to `absent`.

##### `set_mss` (optional)
Set maximum segment size (MSS). Defaults to `absent`.

##### `clamp_mss_to_pmtu` (optional)
Clamp MSS to path MTU. Defaults to `false`.

##### `to_src` (optional)
Source NAT. Defaults to `absent`.

##### `to_dst` (optional)
Destination NAT. Defaults to `absent`.

##### `limit` (optional)
Limit rate. Defaults to `absent`.

##### `limit_burst` (optional)
Limit burst. Defaults to `absent`.

##### `log_level` (optional)
Log level. Valid values are: `absent`, `emerg`, `alert`, `crit`, `err`, `warning`, `notice`, `info`, `debug`. Defaults to `absent`.

##### `comment` (optional)
Comment. Defaults to `absent`.

##### `ensure` (optional)
Whether the resource is present or not. Valid values are `present`, `absent`. Defaults to `present`.

#### fluffy_test
`fluffy_test` manages the Fluffy test process. This will only run upon receiving refresh events.

```
fluffy_test {"session": }
```

##### `name` (optional)
The session name. The only valid value is `puppet`.

#### fluffy_commit
`fluffy_commit` manages the Fluffy commit process. This will only run upon receiving refresh events.

```
fluffy_commit {"session": }
```

##### `name` (optional)
The session name. The only valid value is `puppet`.

##### `rollback` (optional)
Enable rollback. Defaults to `false`.

##### `rollback_interval` (optional)
Rollback configuration after a certain period of time unless confirmed. Defaults to `0`.

#### fluffy_confirm
`fluffy_confirm` manages the Fluffy commit-/confirm process. This will only run upon receiving refresh events.

```
fluffy_confirm {"session": }
```

##### `name` (optional)
The session name. The only valid value is `puppet`.

#### fluffy_check
`fluffy_check` manages Fluffy rollback checks.

```
fluffy_check {"check": }
```

##### `name` (required)
Check name

##### `type` (optional)
Check type. Valid values are `tcp`, `exec`.

##### `command` (optional)
Command to execute.

##### `host` (optional)
TCP host

##### `port` (optional)
TCP port

##### `timeout` (optional)
Check timeout in seconds. Defaults to `5`.

##### `ensure` (optional)
Whether the resource is present or not. Valid values are `present`, `absent`. Defaults to `present`.

<a name="hiera"/>

## Hiera integration

The entire module data is driven via Hiera.

<a name="contact"/>

## Contact

Matteo Cerutti - matteo.cerutti@hotmail.co.uk
