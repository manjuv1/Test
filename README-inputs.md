
# README - Inputs

The following is a description of the inputs required from the user to generate the Ansible inventory used to deploy or update a topology.

- Inputs have to be provided in a folder `input` within the playbook directory.
- All inputs are provided in yml.


**fabric.yml**

| Key        | Type                      | Support               |Description                                             | 
|------------|---------------------------|------------------------|--------------------------------------------------------|
| ``site_code``            | string        |  Required | Site code to be used  |
| ``switch_cli_password`` | string           |  Required | cli password  |
| ``management_network``    | string           |  Required | Management network to provide management IPs for all devices in inventory. <br/> The first 10 IPs are reserved for other purposes. The IPs starting from 11 are assigned for all superspines followed by pods. Within pods, first spines are assigned in the order of switchId followed by the racks in the order of rackIds. IPs are reserved for the maximum racks supported in the pod even if all racks are not yet provisioned. Within racks, IPs are assigned in the order of the switchId. |
| ``management_gateway``       | string        |  Required | Management gateway |
| ``automation_server``                  | string            |  Required | Automation server IPv4 address where the ansible playbooks are executed |
| ``file_server``                 | string          |  Required | File server IPv4 address where external files are kept |
| ``ztp_server``           | string           |  Required | File server IPv4 address where ztp configs/scripts/binary are placed  |
| ``hostname_format``                 | string         |  Recommended | hostname_format is used to generate hostname for switches. It is a string format with keywords within curly braces ``{}``. The keywords will be replaced with actual values during config generation. The format should be able to generate hostnames that uniquely identifies each device. <br/> Keywords that can be used within the ``{}`` in the format are ``site_code, role, pod_id, pod_name, rack_id, switch_id``. <br/> - User can provide the hostname for any switch in the switches input with key 'name'. <br/>   - If hostname is not provided in switches input, the playbook generates hostname based on hostname_format.  <br/><ul> * If there is only one pod, following format can be configured. ``{site_code}-{role}-r{rack_id}sw{switchid}``. Ex: ``sjc-leaf-r04sw01`` <br/> * If site_code is not requried, the following format can be configured. ``{role}-p{pod_id}r{rack_id}sw{switchid}``. Ex: ``leaf-p03r04sw01``</ul> - If hostname_format is not provided, the default format ``{site_code}-{role}-p{pod_id}r{rack_id}sw{switch_id}`` will be used. Ex: ``sjc-leaf-p03r04sw01``. <br/><br/> Note in the examples, sjc is site_code, role is leaf, pod_id is 03, rack_id is 04, switch_id is  01. <br/> |
| ``enable_mgmt_vrf``            | boolean        |  Required | Enable/Disable management vrf |
| ``ntp``       | dictionary        |  Optional | Configures the ntp. Refer enterprise_sonic resource module for format. Use sonic_ntp.config data model from [dellemc.enterprise_sonic.sonic_ntp](https://docs.ansible.com/ansible/latest/collections/dellemc/enterprise_sonic/sonic_ntp_module.html#parameters) |
| ``logging``                  | dictionary            |  Optional | Configures the logging server. <br/> - ``servers`` list of dictionary, Required, List of logging server parameters. <br/> - ``servers.address`` string, Required, IpAddress of logging server. <br/> - ``servers.vrf`` string, Optional, vrf for  logging server. <br/> - ``servers.remort_port`` string, Optional, remort port for  logging server. <br/> - ``servers.message_type`` list, Optional, message type for  logging server. <br/> - ``servers.source_interface`` string, Optional, source interface for  logging server. <br/> |
| ``radius``                 | dictionary          |  Optional | Configures the radius server. Refer enterprise_sonic resource module for format. Use sonic_radius_server.config data model from [dellemc.enterprise_sonic.sonic_radius_server](https://docs.ansible.com/ansible/latest/collections/dellemc/enterprise_sonic/sonic_radius_server_module.html#parameters) <br/> Note: Either radius or tacacs can be provided; not both |
| ``tacacs``                 | dictionary          |  Optional | Configures the tacacs server. Refer enterprise_sonic resource module for format. Use sonic_tacacs_server.config data model from [dellemc.enterprise_sonic.sonic_tacacs_server](https://docs.ansible.com/ansible/latest/collections/dellemc/enterprise_sonic/sonic_tacacs_server_module.html#parameters) <br/> Note: Either radius or tacacs can be provided; not both |
| ``snmp_server``    | dictionary           |  Optional | Configures snmp_server details. <br/> - ``enable_traps`` boolean, Optional, Enable/Disable traps on SNMP server. <br/> - ``agentaddress`` list of dictionary, Optional, List of SNMP agent addresses. <br/> - ``agentaddress.listening_ip`` string, Required, set value as 'mgmt_interface_ip' to configure management ipadddress as snmp agentaddress else set value to specific agent ip address. <br/> - ``agentaddress.interface`` string, Optional, name of the interface for SNMP agent. set value as 'mgmt' to configure interface vrf management.  <br/> - ``community`` list, Optional, SNMP server community. <br/> - ``contact`` string, Optional, SNMP server contact. <br/> - ``location`` string, Optional, SNMP server location. <br/> - ``engine`` string, Optional, SNMP server engine. <br/> - ``group`` list of dictionary, Optional, SNMP server group. <br/> - ``group.name`` string, Required, Name of SNMP server group. <br/> - ``group.version`` string, Required, Version of SNMP server group. <br/> - ``group.notify`` string, Optional, Configure notify view of SNMP server group. <br/> - ``group.read`` string, Optional, Configure read view of SNMP server group. <br/> - ``group.write`` string, Optional, Configure write view of SNMP server group. <br/> - ``host`` list of dictionary, Optional, SNMP server host. <br/> - ``host.address`` string, Required, Address of SNMP server host. <br/> - ``host.community`` string, Optional, SNMP server host community. <br/> - ``host.user`` string, Optional, SNMP server host user. <br/> - ``host.informs`` string, Optional, SNMP server host informs. <br/> - ``host.retries`` string, Optional,  SNMP server host inform retries. <br/> - ``host.timeout`` string, Optional,  SNMP server host inform timeout.  <br/> - ``host.traps`` string, Optional,  SNMP server host traps. <br/> - ``host.source_interface`` string, Optional,  SNMP server host source interface. <br/> - ``host.vrf`` string, Optional,  SNMP server host vrf to use for connection.      |
| ``cli_commands``                 | list          |  Optional | Configures list of sonic-cli commands. |

**pods.yml**

| Key        | Type                      | Support               | Description                                             |
|------------|---------------------------|-----------------------|---------------------------------------------------------|
| ``superspine``            | dictionary        | Optional | Details of superspine supported. <br/> - ``superspine_model`` string, Required, superspine device model <br/> - ``breakout_mode`` string, Required, breakout mode for the superspine model <br/> - ``max_superspines_per_dc`` string, Required, Maximum number of superspines supported per datacenter <br/> - ``superspines_per_dc`` string, Required, Number of superspines configured per datacenter <br/> - ``superspine_loopback0_router_id_network`` string, Required, loopback0 router id network for superspines |
| ``spine`` | dictionary           | Required | Details of spine supported. <br/> - ``spine_model`` string, Required, spine device model <br/> - ``max_spines_per_pod`` string, Required, Maximum number of spines supported per pod <br/> - ``spines_per_pod`` string, Required, Number of spines supported per pod |
| ``leaf``    | dictionary           | Required | Details of leaf supported. <br/> - ``leaf_model`` string, Required, leaf device model <br/> - ``leafs_per_rack`` string, Required, Number of leafs supported per rack <br/> - ``mclag`` dict, Optional <br/> ``mclag.domain_id`` integer, Required, mclag domain id; default value: 1 <br/> ``mclag.portchannel_id`` integer, Required, peerlink portchannel id; default value: 128 <br/>|
| ``start_asn``           | integer           | Required | Starting ASN to assign asn to all devices. The first asn is assigned to all superspines. In each pod, all spines has the same asn and each rack has unique asn i.e. leafs within a rack has the same asn. Second asn is assigned to spines in first pod. Next asn number is assigned for leaf switches in first rack of the first pod and so on. ASNs are reserved for maximum racks even if they are not yet provisioned. ASNs are assigned to devices in the order of the podIds. |
| ``no_of_pods``           | integer           | Required | Number of pods supported |
| ``pods``               | list of dictionary       | Required | Details of each pod. <br/> - ``id`` integer, Required, unique Pod Id starting with 1 <br/> - ``name`` string, Required, Pod name <br/> - ``no_of_racks`` integer, Required, Number of racks supported per Pod <br/> - ``loopback0_router_id_network`` integer, Required, loopback0 router id network <br/> - ``loopback1_vtep_network`` integer, Required, loopback1 vtep network |


**inventory.yml**

| Key        | Type                      | Support               | Description                                             |
|------------|---------------------------|-----------------------|-------------------------------------------------------|
| ``inventory.switches``            | list of dictionary       | Required | List of switches. <br/> - ``id`` integer, Required, unique Id starting with 1 <br/> - ``podId`` integer, Optional, Pod Id. Must be provided for leafs/spines <br/> - ``rackId`` integer, Optional, Rack Id. Must be provided for leafs <br/> - ``switchId`` integer, Required, switch Id. To identify the switch within a rack/spine/superspine. <br/> - ``role`` string, Required, "leaf" or "spine" or "superspine" <br/> - ``mac`` string, Required, MAC Address <br/> - ``name`` string, Optional, Host name. If not provided in input, will be generated. <br/> - ``ipAddress`` string, Optional, Management IP Address. If not provided in input, will be generated. |


**vlan_vrf.yml**

| Key        | Type                      | Support               | Description                                             |
|------------|---------------------------|-----------------------|---------------------------------------------------------|
| ``vrfs`` | list of dictionary           | Optional | List of vrfs <br/> - ``name`` string, Required, vrf name <br/> - ``l3_vni`` integer, Required, L3 vni |
| ``dhcp_relays``    | list of dictionary           | Optional | List of dhcp relay parameters <br/> - ``id`` integer, Required <br/> - ``ip`` string, Required, DHCP Server address <br/> - ``vrf`` string, Optional, vrf to which the dhcp server is mapped; If not provided, will be mapped to default vrf |
| ``vlans``    | list of dictionary           | Required | List of vlans <br/> - ``id`` integer, Required, vlan id (vni will be same as vlan id) <br/> - ``ip_subnet`` string, Required, subnet to be used for the vlan <br/> - ``vrf_name`` string, Optional, vrf to which the vlan is mapped; If not provided, will be mapped to default vrf <br/> - ``dhcp_relay_ids`` list, Optional, list of ids in ``dhcp_relays``|
| ``vlan_members``    | list of dictionary           | Required | Details of vlan members <br/> - ``member_ports`` list, Required, Interfaces listening on the vlan <br/> - ``portchannel_id`` integer, Optional, portchannel id; If not provided, vlan will be assigned to provided member ports instead of portchannel <br/> - ``hostnames`` list of string, Required, list of leafs where the vlan has to be added <br/> - ``tagged_vlans`` list of integers, Optional, tagged vlan list to be mapped <br/> - ``untagged_vlan`` integer, Optional, untagged vlan <br/> - ``speed`` integer, Optional, interface speed |

**interfaces.yml**

| Key        | Type                      | Support               | Description                                             |
|------------|---------------------------|-----------------------|---------------------------------------------------------|
| ``interfaces``    | list of dictionary           | Required | Details of interface parameters <br/> - ``interface_names`` list, Required, Standard Interface names/ranges. <br/> - ``hostnames`` list of string, Required, list of hosts where the interfaces has to be configured. <br/> - ``breakout_mode`` string, Optional, Interface breakout mode. |

**defaults.yml**

| Key        | Type                      | Default               | Description                                             |
|------------|---------------------------|-----------------------|---------------------------------------------------------|
| ``switch_cli_user``            | string        | ``admin`` | cli user name. |
| ``max_lease_time`` | integer           | ``7200`` | Max lease time to be configured in dhcpd.conf |
| ``default_lease_time``    | integer           | ``600`` | default lease time to be configured in dhcpd.conf |
| ``interface_naming``       | string        | ``standard`` | interface_naming to be used |
| ``max_interlink_portchannel_id``    | integer           |  ``127`` | Max interlink portchannel ID. |
| ``vtep_name``    | string           |  ``vtep1`` | VTEP name |
| ``anycast_mac_address``    | string  |  ``00:00:00:11:11:11`` | anycast mac address |
| ``os_version``    | string  |  ``4.0.1-Enterprise_Advanced`` | Expected OS version the devices are to be. Required for validation playbooks.  |
| ``model_support_data``    | dictionary           |  Refer yml file | Contains details of model specific information. <br/> - ``<Platform>`` dictionary, Required, role based variables used for ansible vars generation. <br/> - ``leaf`` dictionary, Required <br/> - ``leaf.max_interface`` string, Required, standard interface name starting which leaf to spine interlinks are generated decrementally. e.g. if max_interface:Eth1/30, leaf to spine1  link will be generated as 'Eth1/30', leaf to spine2 will be Eth1/29 and so on. <br/> - ``leaf.icl_ports`` list, Required, list of standard interface names used for ICL(Inter-Chassis link). <br/> - ``spine`` dictionary, Required <br/> - ``spine.max_interface`` string, Required, standard interface name starting which spine to superspine interlinks are generated decrementally. e.g. if max_interface:Eth1/32, spine to superspine1  link will be generated as 'Eth1/32', spine to superspine2 will be Eth1/31 and so on. <br/> - ``spine.max_no_of_racks`` integer, Required, Maximum of no of racks supported.    |
| ``dhcp_src_int``    | string           |  ``Loopback0`` | source interface to be configured in dhcp relay |


Example: Sample Input
-------
[fabric.yml](input/fabric.yml)

[pods.yml](input/pods.yml)

[inventory.yml](input/inventory.yml)

[vlan_vrf.yml](input/vlan_vrf.yml)

[interfaces.yml](input/interfaces.yml)

[defaults.yml](input/defaults.yml)

