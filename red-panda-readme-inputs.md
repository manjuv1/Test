
# README - Inputs

The following is a description of the inputs required from the user to generate the Ansible inventory used to deploy or update a toplogy.

- Inputs have to be provided in a folder `input` within the playbook directory.
- All inputs are provided in JSON.


**fabric.json**

| Key        | Type                      | Support               |Description                                             | 
|------------|---------------------------|------------------------|--------------------------------------------------------|
| ``site_name``            | string        |  Recommended | Site name to be used  |
| ``switch_cli_password`` | string           |  Required | cli password |
| ``management_network``    | string           |  Required | Management network to provide IPs for all devices in inventory |
| ``management_gateway``       | string        |  Required | Management gateway |
| ``automation_server_ip``                  | string            |  Required | Automation server IPv4 address where the ansible playbooks are executed |
| ``file_server_ip``                 | string          |  Required | File server IPv4 address where external files are kept |
| ``ztp_server_ip``           | string           |  Required | File server IPv4 address where ztp configs/scripts/binary are placed  |
| ``expected_os_version``               | string         |  Required | Expected OS version the devices are to be. Required for validation playbooks.  |
| ``hostname_format``                 | string         |  Recommended | Host name format. <br/> - User can provide the hostname for any switch in the inventory input. <br/> - If hostname is not provided in input, the playbook generates hostname based on hostname_format. hostname_format is a string format with keywords within curly braces ``{}``. The keywords will be replaced with actual values. The format should be able to generate hostnames that uniquely identifies each device. <br/> Keywords that can be used within the ``{}`` in the format: ``site_code, role, pod_id, pod_name, rack_id, switch_id``. <br/> - If hostname_format is not provided, the default  ``{site_code}-{role}-p{pod_id}r{rack_id}sw{switch_id}`` will be used. <br/> Ex: ``sjc-leaf-p003r4sw01``, where sjc is the site_code, role is leaf, pod_num is 03, rack nmber is 04, switch number is  01. <br/> - If there is only one pod, following format can be used. ``{site_code}-{role}-r{rack_id}sw{switchid}`` <br/> Ex: ``sjc-lf-r04sw01`` <br/> - If site_code is not requried, use the following format. ``{role}-p{pod_id}r{rack_id}sw{switchid}`` <br/> Ex: ``leaf-p03r04sw01`` |
| ``enable_mgmt_vrf``            | boolean        |  Required | Enable/Disable management vrf |
| ``snmp_server``    | dictionary           |  Optional | Configures snmp_server details. Refer enterprise_sonic resource module for format. |
| ``ntp``       | dictionary        |  Optional | Configures the ntp. Refer enterprise_sonic resource module for format. |
| ``logging``                  | dictionary            |  Optional | Configures the logging server. Refer enterprise_sonic resource module for format. |
| ``radius``                 | dictionary          |  Optional | Configures the radius server. Refer enterprise_sonic resource module for format. |
| ``leaf_uplinks``           | list of string           |  Required | Configures the leaf uplinks  |


**pods.json**

| Key        | Type                      | Support               | Description                                             |
|------------|---------------------------|-----------------------|---------------------------------------------------------|
| ``superspine_model``            | string        | Required | The device model for superspines |
| ``spine_model`` | string           | Required | The device model for spines |
| ``leaf_model``    | string           | Required | The device model for leafs |
| ``superspines_per_dc``       | integer        | Required | Number of superspines in the DC |
| ``spines_per_pod``                  | integer            | Required | Number of spines for each pod |
| ``leafs_per_rack``                 | integer           | Required | Number of leafs per rack |
| ``no_of_pods``           | integer           | Required | Number of pods supported  |
| ``pods``               | list of dictionary       | Required | Details of each pod. <br/> - ``id`` integer, Required, unique Pod Id starting with 1 <br/> - ``name`` string, Required, Pod name <br/> - ``noOfRacks`` integer, Required, Number of racks supported per Pod |


**inventory.json**

| Key        | Type                      | Support               | Description                                             |
|------------|---------------------------|-----------------------|-------------------------------------------------------|
| ``inventory.switches``            | list of dictionary       | Required | List of switches. <br/> - ``id`` integer, Required, unique Id starting with 1 <br/> - ``podId`` integer, Optional, Pod Id. Must be provided for leafs/spines <br/> - ``rackId`` integer, Optional, Rack Id. Must be provided for leafs <br/> - ``switchId`` integer, Required, switch Id. To identify the switch within a rack/spine/superspine. <br/> - ``role`` string, Required, "LEAF" or "SPINE" or "SUPERSPINE" <br/> - ``mac`` string, Required, MAC Address <br/> - ``name`` string, Optional, Host name. If not provided in input, will be generated. <br/> - ``ipAddress`` string, Optional, Management IP Address. If not provided in input, will be generated. |


**defaults.json**

| Key        | Type                      | Default               | Description                                             |
|------------|---------------------------|-----------------------|---------------------------------------------------------|
| ``switch_cli_user``            | string        | ``admin`` | cli user name. |
| ``max_lease_time`` | integer           | ``7200`` | Max lease time to be configured in dhcpd.conf |
| ``default_lease_time``    | integer           | ``600`` | default lease time to be configured in dhcpd.conf |
| ``interface_naming``       | string        | ``standard`` | interface_naming to be used |


Example: Sample Input
-------
[fabric.json](input/fabric.json)

[pods.json](input/pods.json)

[inventory.json](input/inventory.json)

[defaults.json](input/defaults.json)
