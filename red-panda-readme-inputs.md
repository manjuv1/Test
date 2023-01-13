
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
| ``automation_server``                  | string            |  Required | Automation server IPv4 address where the ansible playbooks are executed |
| ``file_server``                 | string          |  Required | File server IPv4 address where external files are kept |
| ``ztp_server``           | string           |  Required | File server IPv4 address where ztp configs/scripts/binary are placed  |
| ``expected_os_version``               | string         |  Required | Expected OS version the devices are to be. Required for validation playbooks.  |
| ``hostname_format``                 | string         |  Recommended | hostname_format is used to generate hostname for switches. It is a string format with keywords within curly braces ``{}``. The keywords will be replaced with actual values during config generation. The format should be able to generate hostnames that uniquely identifies each device. <br/> Keywords that can be used within the ``{}`` in the format are ``site_code, role, pod_id, pod_name, rack_id, switch_id``. <br/> - User can provide the hostname for any switch in the switches input with key 'name'. <br/>   - If hostname is not provided in switches input, the playbook generates hostname based on hostname_format.  <br/><ul> * If there is only one pod, following format can be configured. ``{site_code}-{role}-r{rack_id}sw{switchid}``. Ex: ``sjc-lf-r04sw01`` <br/> * If site_code is not requried, the following format can be configured. ``{role}-p{pod_id}r{rack_id}sw{switchid}``. Ex: ``leaf-p03r04sw01``<br/></ul> - If hostname_format is not provided, the default format ``{site_code}-{role}-p{pod_id}r{rack_id}sw{switch_id}`` will be used. Ex: ``sjc-leaf-p03r04sw01``. <br/><br/> Note in the examples, sjc is site_code, role is leaf, pod_id is 03, rack_id is 04, switch_id is  01. <br/> |
| ``enable_mgmt_vrf``            | boolean        |  Required | Enable/Disable management vrf |
| ``ntp``       | dictionary        |  Optional | Configures the ntp. Refer enterprise_sonic resource module for format. |
| ``logging``                  | dictionary            |  Optional | Configures the logging server. Refer enterprise_sonic resource module for format. |
| ``radius``                 | dictionary          |  Optional | Configures the radius server. Refer enterprise_sonic resource module for format. |
| ``tacacs``                 | dictionary          |  Optional | Configures the tacacs server. Refer enterprise_sonic resource module for format. <br/> Note: Either radius or tacacs can be provided; not both |


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
| ``snmp_server``    | dictionary           |  ``{"agentaddress": [{ "listening_ip": "mgmt_interface_ip", "interface":"mgmt" }]}`` | Configures snmp_server details. Refer enterprise_sonic resource module for format. |


Example: Sample Input
-------
[fabric.json](input/fabric.json)

[pods.json](input/pods.json)

[inventory.json](input/inventory.json)

[defaults.json](input/defaults.json)
