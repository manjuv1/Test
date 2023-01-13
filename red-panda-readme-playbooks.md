
# README - Code Flow

- [README - Code Flow](#readme---code-flow)
  - [Pre-day 0 Playbooks](#pre-day-0-playbooks)
  - [Day 0 Playbooks:](#day-0-playbooks)
  - [Validation Playbooks](#validation-playbooks)

**Provide Inputs:**

Update all values in the json/csv files in [input/](./input/).

## Pre-day 0 Playbooks

Playbook: `ansible-playbook playbooks/config_preDay0.yml`

Purpose: Validates inputs provided, generates hostname and IPs, ansible variables/inventory, dhcp/ztp configs

On execution of above playbook, below playbooks are imported.
- Playbook: playbooks/all/validate_input.yml

```yml
        Purpose: Validates inputs provided
        Role: dellemc/danaf/roles/generation
            tasks/validate_input.yml:
                Purpose: Validates the key attributes in the input json.
                Module: dellemc/danaf/plugins/modules/validate_input_json.py
```

- Playbook: playbooks/all/gen_detailed_input.yml

```yml
        Purpose: Generates detailed json with hostname and managment IPs auto-generated.
        Role: dellemc/danaf/roles/generation
            tasks/generate_detailed_input.yml:
                Purpose: Generated detailed json with hostname and management IPs.
                Module: dellemc/danaf/plugins/modules/generate_detailed_json.py
        Output:
          Detailed json: inventory/generated_configs/jsons
```

- Playbook: playbooks/all/gen_ansible_vars.yml

```yml
        Purpose: Generates ansible variables required to use the enterprise sonic ansible collection.
        Role: dellemc/danaf/roles/generation
            tasks/generate_ansible_vars.yml:
                Purpose: Generates ansible variables from input json
                Module: dellemc/danaf/plugins/modules/generate_ansible_vars.py
        Output:
          Ansible vars: inventory/group_vars
```

- Playbook: playbooks/all/gen_dhcp_inv_ztp.yml

```yml
        Purpose: Generate dhcp/ztp conf, inventory, etc/hosts
        Role: dellemc/danaf/roles/dhcp_inv
          tasks/generate.yml:
                tasks/inventory_gen.yml
                   Purpose: Generate ansible inventory file with the hosts and groups info
                   Template: templates/inventory_gen.j2
                tasks/etc_hosts_gen.yml
                   Purpose: Generate /etc/hosts
                   Template: templates/etc_hosts_gen.j2
                tasks/dhcp_conf_gen.yml
                   Purpose: Generate dhcpd.conf including ztp config file path
                   Template: templates/dhcpd.conf.j2
                tasks/ztp_gen.yml
                   Purpose: Generate ztp conf and scripts
                   Template: templates/ztp.j2
                   Template: templates/set-password.j2
        Output:
          Detailed json: inventory/generated_configs
```

- Playbook: playbooks/all/cp_dhcp_inv_ztp.yml

```yml
        Purpose: Copies the dhcp/ztp conf, inventory, etc/hosts to the destined directories
        Role: dellemc/danaf/roles/dhcp_inv
          tasks/copy_generated.yml:
                tasks/dhcp_cp_restart.yml
                   Purpose: Copy dhcpd.copy to /etc/dhcp directory and restarts dhcp service
                tasks/etc_hosts_cp.yml
                   Purpose: Compare and update /etc/hosts file
                tasks/inventory_cp.yml
                   Purpose: Copy ansible inventory file to destination
                tasks/ztp_cp.yml
                   Purpose: Copy ztp.json and scripts generated to ztp server
```

## Day 0 Playbooks:

Playbook: `ansible-playbook playbooks/install_image.yaml -e "target_os=ONIE"`

```yml
        Purpose: Put devices in ONIE so that DHCP is triggered and devices come up in the required sonic image and ztp configs
        Role: dellemc/danaf/roles/image
          tasks/main.yml:
                tasks/normalize_sonic.yml
                   Purpose: Sets next boot to ONIE and reboots the device
```

Playbook: `ansible-playbook playbooks/config_Day0.yml`

```yml
        Purpose: Set Day0 common configs for all devices (hostname, ntp, snmp, tacacs+/radius, syslog, banner)
        Role: dellemc.danaf.hostname
          tasks/main.yml:
                   Purpose: Configure hostname
        Role: dellemc.danaf.ntp
          tasks/main.yml:
                   Purpose: Configure ntp
        Role: dellemc.danaf.syslog
          tasks/main.yml:
                   Purpose: Configure syslog servers
        Role: dellemc.danaf.snmp
          tasks/main.yml:
                   Purpose: Configure snmp
        Role: dellemc.danaf.banner
          tasks/main.yml:
                   Purpose: Configure banner
        Role: dellemc.danaf.radius
          tasks/main.yml:
                   Purpose: Configure radius
        Role: dellemc.danaf.aaa
          tasks/main.yml:
                   Purpose: Configure aaa
        Role: dellemc.danaf.tacacs
          tasks/main.yml:
                   Purpose: Configure tacacs+
```

## Day 1 Playbooks:

Playbook: `ansible-playbook playbooks/config_Day1.yml`

```yml
        Purpose: Set Day1 configs for all devices (anycast MAC address, lag_interfaces, interfaces, l2_interfaces, l3_interfaces, bgp, bgp_af, bgp_neighbors, bgp_neighbors_af, mclag, vxlan)
        Role: dellemc.danaf.system
          tasks/main.yml:
                   Purpose: Configure ipv4 anycast mac address
        Role: dellemc.danaf.lag_interfaces
          tasks/main.yml:
                   Purpose: Configure sonic LAG interfaces
        Role: dellemc.danaf.interfaces
          tasks/main.yml:
                   Purpose: Configure sonic interfaces
        Role: dellemc.danaf.l2_interfaces
          tasks/main.yml:
                   Purpose: Configure sonic l2 interfaces
        Role: dellemc.danaf.l3_interfaces
          tasks/main.yml:
                   Purpose: Configure sonic l3 interfaces
        Role: dellemc.danaf.bgp
          tasks/main.yml:
                   Purpose: Configure router BGP for default vrf
        Role: dellemc.danaf.bgp_af
          tasks/main.yml:
                   Purpose: Configure BGP Address Family
        Role: dellemc.danaf.bgp_neighbors
          tasks/main.yml:
                   Purpose: Configure BGP neighbors
        Role: dellemc.danaf.bgp_neighbors_af
          tasks/main.yml:
                   Purpose: Configure BGP neighbors Address Family
        Role: dellemc.danaf.mclag
          tasks/main.yml:
                   Purpose: Configure MCLAG domain
        Role: dellemc.danaf.vxlan
          tasks/main.yml:
                   Purpose: Configure VXLAN
```

## Validation Playbooks

- Playbook: `ansible-playbook validate_reachability.yaml`

```yml

        Purpose: Waits for ssh port to check the device reachability
        Role: dellemc/danaf/roles/validation
            tasks/reachability.yml
```

- Playbook: `ansible-playbook validate_ztp_status.yaml`

```yml
        Purpose: Checks the ztp status
        Role: dellemc/danaf/roles/validation
            tasks/ztp-status.yaml
```

- Playbook: `ansible-playbook validate_os_type.yaml`

```yml
        Purpose: Checks the OS type in the device
        Role: dellemc/danaf/roles/validation
            tasks/os-type.yaml
```

-  Playbook: `ansible-playbook validate_device_type.yaml`

```yml
        Purpose: Validates the device model
        Role: dellemc/danaf/roles/validation
            tasks/device-type.yaml
```

- Playbook: `ansible-playbook validate_version.yaml`

```yml
        Purpose: Validates the version
        Role: dellemc/danaf/roles/validation
            tasks/os-version.yaml
```
