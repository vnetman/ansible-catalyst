# ansible-catalyst
Ansible Role that generates a fairly comprehensive configuration for a Cisco Catalyst switch used as a campus distribution switch

This Ansible Role generates configurations for Cisco Catalyst (or Catalyst-like) switches. The generated configurations are typical of switches deployed in a Distribution Layer role in a 3-tier enterprise campus. The configurations generated comprise of the following sections:

- Device configuration (hostname, boot images etc.)
- Logging configuration (syslog server, logging buffer sizes etc.)
- AAA configuration (TACACS, AAA etc.)
- User-interface configuration (SNMP, console and vty lines etc.)
- Layer 2 configuration (spanning tree, VLANs, switchports, layer-2 port-channels etc.)
- Layer 3 (IPv4) configuration (Routed interfaces, layer-3 port-channels, HSRP, OSPF etc.)
- Netflow configuration
- QoS configuration

The configurations are created and stored locally as files on the Ansible Control Machine, under the `dist-switches-cfg/<hostname>` subdirectory of the directory where the main playbook resides. The idea is that once the consolidated configuration is generated, the network admin inspects them and then manually copies them over to the devices as `startup-config` (of course this can be automated as well if needed).

The role makes heavy use of Jinja2 templates to convert "intentions" (written in YAML files) to Cisco IOS configuration CLI. For example, the QoS "intention" is written as YAML, like this:

```
l2_interface_queueing:
  - category:
      name: voice
      ip_precedence: 5
      queue_action: priority

  - category:
      name: network
      ip_precedence: 6,7
      queue_action: bandwidth remaining percent 5
```

and the Jinja2 template then renders the IOS CLI:

```
class-map match-any voice
 match ip precedence 5
class-map match-any network
 match ip precedence 6
 match ip precedence 7

policy-map pm_default_shape
 class voice
  priority
 class network
  bandwidth remaining percent 5
```

and then goes on to apply `service-policy output pm_default_shape` on all the layer 2 switchports (which are in turn defined and identified in a different YAML file).

## The `csrange` lookup filter

The Jinja2 templates make use of the `csrange` Ansible filter written by me, and available on [github here](https://github.com/vnetman/ansible-csrange-lookup). I am working towards making it available on Ansible Galaxy, but I am not there yet. In the meanwhile, you can create a subdirectory named `lookup_plugins` under the directory where your main playbook is located, and copy the `csrange.py` from the github repository into that subdirectory.

## Files in this repository

```
.
├── LICENSE
├── pb-dist-switches.yml
├── README.md
└── roles
    └── distribution_switches
        ├── tasks
        │   ├── 01-device.yml
        │   ├── 02-logging.yml
        │   ├── 03-aaa.yml
        │   ├── 04-user-interface.yml
        │   ├── 05-layer-2.yml
        │   ├── 06-layer-3.yml
        │   ├── 07-netflow.yml
        │   ├── 08-qos.yml
        │   └── main.yml
        ├── templates
        │   ├── aaa_configurations.j2
        │   ├── device_configurations.j2
        │   ├── layer_2_configurations.j2
        │   ├── layer_3_configurations.j2
        │   ├── logging_configurations.j2
        │   ├── netflow_configurations.j2
        │   ├── qos_configurations.j2
        │   └── user_interface_configurations.j2
        └── vars
            ├── 01-device-vars.yml
            ├── 03-aaa-vars.yml
            ├── 04-user-interface-vars.yml
            ├── 05-layer-2-vars.yml
            ├── 06-layer-3-vars.yml
            ├── 08-qos-vars.yml
            ├── main.yml
            └── secrets.yml

5 directories, 28 files
```
