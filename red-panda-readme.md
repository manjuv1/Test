# Red Panda

Ansible Network Automation Framework

- [Red Panda](#red-panda)
  - [Automation Server Setup](#automation-server-setup)
    - [Ubuntu](#ubuntu)
    - [Rocky Linux](#rocky-linux)
  - [Inputs](#inputs)
  - [Playbooks](#playbooks)
  - [BGP Configuration](#bgp-configuration)

## Automation Server Setup

- Automation Server consists of following services
	-	Ansible 
	-	DHCP Server (optional)
	-	ZTP Server (optional)

### Ubuntu

- Install DHCP Server (isc-dhcp-server)

        # Commands
        sudo apt update
        sudo apt install isc-dhcp-server

        # Verification
        sudo service isc-dhcp-server status

- Install Webserver (apache2)

        # Commands
        sudo apt-get install apache2

        # Verification
        sudo systemctl status apache2

- Install Ansible

    Refer [Ansible Installation Documentation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu) for Control Node requirements and Installation procedure.

    Refer [Ansible Python3 support](https://docs.ansible.com/ansible/latest/reference_appendices/python_3_support.html)

        - Verification
                    which ansible
                    ansible --version

- Fetch red panda to a folder

        git clone https://github.com/Dell-Networking/red-panda.git
        git checkout dev

- Install required packages for automation

        sudo apt install sshpass
        # You will need to cd to your red panda directory before executing this step
        # The requirements.txt is at the top level
        pip3 install -r requirements.txt

- Install ansible collections for dellemc.enterprise_sonic

        ansible-galaxy collection install dellemc.enterprise_sonic

- Link danaf dev ansible collection

        cd ~/.ansible/collections/ansible_collections/dellemc/
        # WARNING: This assumes you put the red-panda project in your home folder.
        # If you did not adjust the path accordingly.
        ln -s ~/red-panda/dellemc/danaf danaf

- Copy the sonic installer image to file server path (default: /var/www/html). This is the SONiC file you received from Dell. You will need to soft link it to the name `onie-installer` by using the command `ln -s <your_sonic_installer> <your_web_root>/onie-installer`. Ex: `ln -s Enterprise_SONiC_OS_4.0.3_vs_Standard.bin /var/www/html/onie-installer`

        ls /var/www/html
        onie-installer

### Rocky Linux

Run all commands from a user command prompt. Do not add sudo where not indicated.

- Install DHCP Server (isc-dhcp-server)

        # Commands
        sudo dnf update -y
        sudo dnf install -y dhcp-server
        sudo systemctl enable --now dhcpd

        # Verification
        sudo systemctl status dhcpd

- Install Webserver (apache2)

        # Commands
        sudo dnf install -y httpd
        sudo systemctl enable --now httpd
        
        # Verification
        systemctl status httpd

- Install Ansible

        # Commands
        sudo dnf install -y ansible-core

        # Verification
        ansible --version

- Fetch red panda to a folder

        git clone https://github.com/Dell-Networking/red-panda.git
        git checkout dev

- Install required packages for automation

        sudo dnf install -y sshpass
        sudo dnf install -y python3-pip
        pip3 install -r requirements.txt

- Install ansible collections for dellemc.enterprise_sonic

        ansible-galaxy collection install dellemc.enterprise_sonic

- Link danaf dev ansible collection

        cd ~/.ansible/collections/ansible_collections/dellemc/
        # WARNING: This assumes you put the red-panda project in your home folder.
        # If you did not adjust the path accordingly.
        ln -s ~/red-panda/dellemc/danaf danaf

- Copy the sonic installer image to file server path (default: /var/www/html). This is the SONiC file you received from Dell. You will need to soft link it to the name `onie-installer` by using the command `ln -s <your_sonic_installer> <your_web_root>/onie-installer`. Ex: `ln -s Enterprise_SONiC_OS_4.0.3_vs_Standard.bin /var/www/html/onie-installer`

        ls /var/www/html
        onie-installer

## Inputs

Details of inputs to be provided with example [README-inputs.md](README-inputs.md)

## Playbooks

Step 1: Validates inputs provided; Generates hostname and IPs, ansible variables/inventory, dhcp/ztp configs

```
ansible-playbook playbooks/config_preDay0.yml
```

Step 2: Put devices in ONIE so that DHCP is triggered and devices come up in the required sonic image and ztp configs

```
ansible-playbook playbooks/install_image.yaml -e "target_os=ONIE"
```

Step 3: Set Day0 common configs for all devices (hostname, ntp, snmp, tacacs+/radius, syslog, banner, interface naming mode)

```
ansible-playbook playbooks/config_Day0.yml
```

Step 4: Validation Playbooks

```
ansible-playbook validate_reachability.yaml
ansible-playbook validate_ztp_status.yaml
ansible-playbook validate_os_type.yaml
ansible-playbook validate_device_type.yaml
ansible-playbook validate_version.yaml
```

Step 5: Set Day1 configs for all devices (ipv4 anycast MAC address, lag_interfaces, interfaces, l3_interfaces, bgp-default vrf, bgp_af, bgp_neighbors, mclag, vxlan)

```
ansible-playbook playbooks/config_Day1.yml
```

- Validation: Set VLAN and VRF configs for leaf devices to validate Day1 (vrf, bgp- vrf tenant, vni_vrf_map, loopback2, vlan, vni_vlan_map, portchannel )
    ```
    ansible-playbook playbooks/test/cfg_vrf_vlan.yml 
    ```

**Note** Detailed info of playbooks/roles/modules used [README-playbooks.md](README-playbooks.md)


## BGP Configuration

Example of setting up BGP configuration is available in [bgp_configuration.md](bgp_configuration.md)
