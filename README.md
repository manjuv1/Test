# Red Panda

Ansible Network Automation Framework

- [Red Panda](#red-panda)
  - [Supported Network Devices](#supported-network-devices)
  - [Supported Versions of SONiC](#supported-versions-of-sonic)
  - [Supported Operating Systems](#supported-operating-systems)
  - [Red Panda Setup](#red-panda-setup)
    - [Install Docker](#install-docker)
    - [Install docker-compose](#install-docker-compose)
    - [Firewall Configuration](#firewall-configuration)
      - [Opening Ports on RHEL Family Systems](#opening-ports-on-rhel-family-systems)
      - [Opening Ports on Ubuntu](#opening-ports-on-ubuntu)
    - [Download Red Panda](#download-red-panda)
    - [Download the SONiC OS Binary](#download-the-sonic-os-binary)
  - [User Inputs](#user-inputs)
  - [Setup Red Panda](#setup-red-panda)
  - [Controlling the Red Panda Container](#controlling-the-red-panda-container)
  - [Playbooks](#playbooks)
    - [Pre-Day 0](#pre-day-0)
    - [Install SONiC](#install-sonic)
    - [Push Day 0 Configs](#push-day-0-configs)
    - [Run Day 0 Validation](#run-day-0-validation)
    - [Run All Configuration](#run-all-configurations)
    - [Run VLANs Configuration](#run-vlans-configuration)
  - [Helpful Commands](#helpful-commands)
    - [Run a Command on the Red Panda Automation Server](#run-a-command-on-the-red-panda-automation-server)
    - [Get The Red Panda Automation Server DHCP/Apache Status](#get-the-red-panda-automation-server-dhcpapache-status)
    - [Get the Logs from the Red Panda Automation Server](#get-the-logs-from-the-red-panda-automation-server)
    - [Access the Container Command Line](#access-the-container-command-line)
  - [Detailed Playbook Information](#detailed-playbook-information)
  - [Run Virtualized SONiC in GNS3](#run-virtualized-sonic-in-gns3)
  - [Troubleshooting](#troubleshooting)
  - [BGP Configuration](#bgp-configuration)

## Supported Network Devices

The following network devices are supported:

- Z9264F-ON
- Z9332F-ON
- Z9432F-ON
- S5232F-ON
- SONIC-VM

## Supported Versions of SONiC

- 4.0.1

We will add support for newer versions but for initial development we have locked the version.

## Supported Operating Systems

- Ubuntu
- RHEL family systems (includes Rocky Linux)

Any Linux system running Docker will likely suffice but these instructions were written for Ubuntu/RHEL. You may have to do some tweaking for other operating systems. 

## Red Panda Setup

The automation Server provides the following services:

- Ansible 
- DHCP Server
- ZTP Server
- Web Server

### User permissions

Make sure the user has sudo permission. 

`sudo -i`

### Install Docker

See [here](https://docs.docker.com/engine/install/ubuntu/) for details on how to install Docker in Ubuntu.

See [here](https://docs.rockylinux.org/gemstones/docker/) for details on how to install Docker in Rocky Linux.

Add the user to the docker group.

```bash
sudo groupadd docker
sudo usermod -aG docker <YOUR_USER>
```

and then log out and log back in. Run `docker run hello-world` as the user in question to test your privileges. If this does not run correctly, Red Panda will not run correctly.

### Install docker-compose

You will also need to install docker-compose version 2. The code **will not work** with version 1. Instructions for 
installing docker compose version 2 are [here](https://docs.docker.com/compose/install/other/#on-linux).

### Firewall Configuration

Make sure you do not have any conflicts on ports 80/tcp or 67/udp. Make sure both of these ports are open and accessible on your firewall.

#### Opening Ports on RHEL Family Systems

```
sudo firewall-cmd --permanent --add-port=80/tcp --add-port=67/udp --zone public
sudo firewall-cmd --reload
```

**NOTE**: This assumes a default firewalld configuration. If you have changed your zoning you will need to update the command accordingly.

#### Opening Ports on Ubuntu

[Ubuntu does not come with a firewall **AND rules** by default](https://askubuntu.com/q/9206/638863). `iptables` is present but there are no blocking rules. The default firewall if enabled is UFW. See [Ubuntu's Website](https://ubuntu.com/server/docs/security-firewall#:~:text=The%20default%20firewall%20configuration%20tool,or%20IPv6%20host%2Dbased%20firewall.) for instructions.

### Download Red Panda

During the beta period, we will pull Red Panda directly from GitHub. To download Red Panda [visit our GitHub site](https://github.com/Dell-Networking/red-panda/tree/lab-setup), click the green *Code* button, and select *Download ZIP*.

![](images/2023-01-18-15-41-26.png)

Extract the resulting Red Panda directory to a location of your choosing on the server. After downloading you need to mark the the `redpanda.sh` file as executable with `chmod +x docker-compose-files/redpanda.sh`

### Download the SONiC OS Binary

Download the SONiC binary from [the Dell Support website](https://www.dell.com/support/home/en-us/product-support/product/enterprise-sonic-distribution/drivers). **NOTE** You will have to have an account to download SONiC. Account info is provided to you at the time of purchase.

Copy the `.bin` file you downloaded (it may be in a compressed folder) and place it in the `redpanda/files` directory.

Run all commands from a user command prompt. Do not add sudo where not indicated.

## User Inputs

Before continuing, you will need to provide all required inputs.

See [README-inputs.md](README-inputs.md) for details on user required input.

## Setup Red Panda

**NOTE** You will need an internet connection for this step.

To prepare Red Panda to deploy your infrastructure, the only thing you need to do is run 

```
docker-compose-files/redpanda.sh setup
```

Red Panda will automatically build itself and after the end of the build sequence deploy the Red Panda Automation Server container.

## Controlling the Red Panda Container

Red Panda runs in Docker and is controlled via underlying `docker-compose` commands. Run `docker-compose-files/redpanda.sh stop` to shutdown the Red Panda container. Run `docker-compose-files/redpanda.sh start` to start a stopped Red Panda container.

## Playbooks

### Pre-Day 0

After preparing the input files you are ready to generate the configurations which will be pushed to the switches and prepare for day 0. To do this run:

```
docker-compose-files/redpanda.sh preday0
```

This command will do the following two things:
- Validates inputs provided; Generates hostname and IPs, ansible variables/inventory, dhcp/ztp configs
- Copies the preDay0 configurations to target directory - dhcp/ztp configs, ansible variables/inventory



### Install SONiC

The next step is to install SONiC on all target devices and place them in the correct ZTP configuration. Pass current switch password in ansible_password.

```
docker exec -it redpanda-automation-server ansible-playbook /redpanda/playbooks/install_image.yaml -e "target_os=ONIE ansible_password=admin"
```


The next step is to the change the sku in leaf devices to support 8x25G and 8x10G server connectivity. This playbook will be removed after sonic os 4.1 version.

```
docker exec -it redpanda-automation-server ansible-playbook /redpanda/playbooks/config_hwsku.yml --limit leaf
```

### Push Day 0 Configs

The next step is to push day 0 common configs to all devices:
- hostname
- ntp
- snmp
- tacacs+/radius
- syslog
- banner
- interface naming mode

```
docker-compose-files/redpanda.sh day0-push-configs
```

### Run Day1 Configurations

Next is Day1 configurations. Pushes the following configs for all devices:
- ipv4 anycast
- MAC address
- lag_interfaces
- mclag
- interfaces
- l3_interfaces
- l2_interfaces
- VxLAN
- DHCP Relay
- BGP-default 
- BGP-VRF
- BGP_af
- BGP_neighbors


```
docker-compose-files/redpanda.sh all
```

### Run Day2 VLAN configuration

Next is Day2 VLAN configurations.

##### Breakout for server connected ports

If breakout mode has to be changed for server connected ports, execute the below command manually for those server connected ports.

```
interface breakout port <port_number> mode <breakout_mode>
```

Ex: `interface breakout port 1/2 mode 8x10G`

##### Pushes the following configs for all devices:

- portchannel
- interface speed
- VLAN
- VRF
- VNI_VRF_map
- VNI_VLAN_map
- DHCP Relay
- BGP-VRF
- BGP_af

```
docker-compose-files/redpanda.sh vlans
```

## Helpful Commands

This section covers some quick tips and tricks for using the Red Panda automation server.

### Run a Command on the Red Panda Automation Server

Use `docker-compose-files/redpanda.sh run <command>` to run a command on the automation server. Ex:

```
[grant@rockylinux docker-compose-files]$ ./redpanda.sh run echo "This is a command"
This is a command
```

### Get The Red Panda Automation Server DHCP/Apache Status

Run `docker-compose-files/redpanda.sh status`.

```
[grant@rockylinux docker-compose-files]$ ./redpanda.sh status
apache                           RUNNING   pid 7, uptime 10:03:25
dhcpd                            STOPPED   Not started
```

### Get the Logs from the Red Panda Automation Server

Run `docker-compose-files/redpanda.sh logs`

### Access the Container Command Line

You can drop to command line on the Red Panda automation server container with `docker exec -it redpanda-automation-server /bin/bash`

## Detailed Playbook Information

Detailed info of playbooks/roles/modules used [README-playbooks.md](README-playbooks.md)

## Run Virtualized SONiC in GNS3

- TODO make a note to use the GNS3 VM and not a remote server
- You have to add it under remote servers
- TODO https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_filters.html#formatting-data-yaml-and-json
- TODO need to add docker usage instructions

## Troubleshooting

- Under the hood, the `redpanda.sh` command is running `ansible-playbook` and can be debugged just like any other Ansible playbook.
- If you need to drop to the command line you can use [drop to the command line](#access-the-container-command-line)
- Try adding the `--verbose` switch to your commands to obtain additional output on what the problem might be

## BGP Configuration

Example of setting up BGP configuration is available in [BGP_configuration.md](BGP_configuration.md)
