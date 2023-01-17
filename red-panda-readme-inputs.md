
# README - Inputs

The following is a description of the inputs required from the user to generate the Ansible inventory used to deploy or update a toplogy.

- Inputs have to be provided in a folder `input` within the playbook directory.
- All inputs are provided in JSON.


**fabric.json**

| Key        | Type                      | Support               |Description                                             | 
|------------|---------------------------|------------------------|--------------------------------------------------------|
| ``site_name``            | string        |  Recommended | Site name to be used  |
| ``switch_cli_password`` | string           |  Required | cli password |
| ``management_network``    | string           |  Required | Management network to provide IPs for all devices in inventory. <br/> The first 10 IPs are reserved for other purposes. The IPs starting from 11 are assigned for all superspines followed by pods. Within pods, first spines in the order of switchId are assigned followed by the racks in the order of rackIds. IPs are reserved for the maximum racks supported in the pod even if all racks are not yet provisioned. Within racks, IPs are assigned in the order of the switchId. |
| ``management_gateway``       | string        |  Required | Management gateway |
| ``automation_server``                  | string            |  Required | Automation server IPv4 address where the ansible playbooks are executed |
| ``file_server``                 | string          |  Required | File server IPv4 address where external files are kept |
| ``ztp_server``           | string           |  Required | File server IPv4 address where ztp configs/scripts/binary are placed  |
| ``expected_os_version``               | string         |  Required | Expected OS version the devices are to be. Required for validation playbooks.  |
| ``hostname_format``                 | string         |  Recommended | hostname_format is used to generate hostname for switches. It is a string format with keywords within curly braces ``{}``. The keywords will be replaced with actual values during config generation. The format should be able to generate hostnames that uniquely identifies each device. <br/> Keywords that can be used within the ``{}`` in the format are ``site_code, role, pod_id, pod_name, rack_id, switch_id``. <br/> - User can provide the hostname for any switch in the switches input with key 'name'. <br/>   - If hostname is not provided in switches input, the playbook generates hostname based on hostname_format.  <br/><ul> * If there is only one pod, following format can be configured. ``{site_code}-{role}-r{rack_id}sw{switchid}``. Ex: ``sjc-lf-r04sw01`` <br/> * If site_code is not requried, the following format can be configured. ``{role}-p{pod_id}r{rack_id}sw{switchid}``. Ex: ``leaf-p03r04sw01``</ul> - If hostname_format is not provided, the default format ``{site_code}-{role}-p{pod_id}r{rack_id}sw{switch_id}`` will be used. Ex: ``sjc-leaf-p03r04sw01``. <br/><br/> Note in the examples, sjc is site_code, role is leaf, pod_id is 03, rack_id is 04, switch_id is  01. <br/> |
| ``enable_mgmt_vrf``            | boolean        |  Required | Enable/Disable management vrf |
| ``ntp``       | dictionary        |  Optional | Configures the ntp. Refer enterprise_sonic resource module for format. |
| ``logging``                  | dictionary            |  Optional | Configures the logging server. Refer enterprise_sonic resource module for format. |
| ``radius``                 | dictionary          |  Optional | Configures the radius server. Refer enterprise_sonic resource module for format. <br/> Note: Either radius or tacacs can be provided; not both |
| ``tacacs``                 | dictionary          |  Optional | Configures the tacacs server. Refer enterprise_sonic resource module for format. <br/> Note: Either radius or tacacs can be provided; not both |


**pods.json**

| Key        | Type                      | Support               | Description                                             |
|------------|---------------------------|-----------------------|---------------------------------------------------------|
| ``superspine``            | dictionary        | Required | Details of superspine supported. <br/> - ``superspine_model`` string, Required, superspine device model <br/> - ``breakout_mode`` string, Required, breakout mode for the superspine model <br/> - ``superspines_per_dc`` string, Required, Number of superspines supported per datacenter <br/> - ``superspine_loopback0_router_id_network`` string, Required, loopback0 router id network for superspines |
| ``spine`` | dictionary           | Required | Details of spine supported. <br/> - ``spine_model`` string, Required, spine device model <br/> - ``spines_per_pod`` string, Required, Number of spines supported per pod |
| ``leaf``    | dictionary           | Required | Details of leaf supported. <br/> - ``leaf_model`` string, Required, leaf device model <br/> - ``leafs_per_rack`` string, Required, Number of leafs supported per rack |
| ``start_asn``           | integer           | Required | Starting ASN to assign asn to all devices. The first asn is assigned to all superspines, followed by all pods in the order of pod ids. Within each pod, all spines has the same asn and each rack has unique asn i.e. leafs within a rack has the same asn. |
| ``no_of_pods``           | integer           | Required | Number of pods supported |
| ``pods``               | list of dictionary       | Required | Details of each pod. <br/> - ``id`` integer, Required, unique Pod Id starting with 1 <br/> - ``name`` string, Required, Pod name <br/> - ``no_of_racks`` integer, Required, Number of racks supported per Pod <br/> - ``loopback0_router_id_network`` integer, Required, loopback0 router id network <br/> - ``loopback1_vtep_network`` integer, Required, loopback1 vtep network <br/> - ``loopback2_network`` integer, Required, loopback2 network |


**inventory.json**

| Key        | Type                      | Support               | Description                                             |
|------------|---------------------------|-----------------------|-------------------------------------------------------|
| ``inventory.switches``            | list of dictionary       | Required | List of switches. <br/> - ``id`` integer, Required, unique Id starting with 1 <br/> - ``podId`` integer, Optional, Pod Id. Must be provided for leafs/spines <br/> - ``rackId`` integer, Optional, Rack Id. Must be provided for leafs <br/> - ``switchId`` integer, Required, switch Id. To identify the switch within a rack/spine/superspine. <br/> - ``role`` string, Required, "leaf" or "spine" or "superspine" <br/> - ``mac`` string, Required, MAC Address <br/> - ``name`` string, Optional, Host name. If not provided in input, will be generated. <br/> - ``ipAddress`` string, Optional, Management IP Address. If not provided in input, will be generated. |


**vlan_vrf.json**

| Key        | Type                      | Support               | Description                                             |
|------------|---------------------------|-----------------------|---------------------------------------------------------|
| ``vrfs`` | list of dictionary           | Optional | List of vrfs <br/> - ``name`` string, Required, vrf name <br/> - ``l3_vni`` integer, Required, L3 vni |
| ``vlans``    | list of dictionary           | Required | List of vlans <br/> - ``id`` integer, Required, vlan id <br/> - ``vni`` integer, Optional, vni; If not provided, will be same as vlan id <br/> - ``ip_subnet`` string, Required, subnet to be used for the vlan <br/> - ``vrf_name`` string, Optional, vrf to which the vlan is mapped; If not provided, will be mapped to default vrf <br/> - ``dhcp_relay_ips`` list of dictionary, Optional, list of dhcp relay IPs and corresponding vrf |
| ``vlan_members``    | list of dictionary           | Required | Details of vlan members <br/> - ``portchannel_id`` integer, Optional, portchannel id; If not provided, vlan will be assgined to provided member ports instead of portchannel <br/> - ``member_ports`` string, Required, Interfaces listening on the vlan <br/> - ``hostname`` list of string, Required, list of leafs where the vlan has to be added <br/> - ``tagged_vlans`` list of integers, Optional, tagged vlan list to be mapped <br/> - ``untagged_vlan`` integer, Optional, untagged vlan <br/> - ``speed`` integer, Optional, interface speed |

**defaults.json**

| Key        | Type                      | Default               | Description                                             |
|------------|---------------------------|-----------------------|---------------------------------------------------------|
| ``switch_cli_user``            | string        | ``admin`` | cli user name. |
| ``max_lease_time`` | integer           | ``7200`` | Max lease time to be configured in dhcpd.conf |
| ``default_lease_time``    | integer           | ``600`` | default lease time to be configured in dhcpd.conf |
| ``interface_naming``       | string        | ``standard`` | interface_naming to be used |
| ``dhcp_server``    | string           |  ``isc-dhcp-server`` | dhcp service supported. |
| ``peerlink_portchannel_id``    | string           |  ``128`` | dhcp service supported. |
| ``mclag_domain_id``    | string           |  ``1`` | dhcp service supported. |
| ``vtep_name``    | string           |  ``vtep_DC`` | dhcp service supported. |
| ``anycast_mac_address``    | string           |  ``00:00:00:11:11:11`` | dhcp service supported. |
| ``model_support_data``    | string           |  ``isc-dhcp-server`` | dhcp service supported. |
| ``dhcp_src_int``    | string           |  ``Loopback0`` | dhcp service supported. |


Example: Sample Input
-------
[fabric.json](input/fabric.json)

[pods.json](input/pods.json)

[inventory.json](input/inventory.json)

[vlan_vrf.json](input/vlan_vrf.json)

[defaults.json](input/defaults.json)
