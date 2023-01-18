# "Port Reset" Day-N Template Deployment

This Ansible playbook will accept a small number of variable values to identify the DNA Center Configuration Template and the managed Device that is targeted, and it will then make the necessary API calls to deploy that template to the target device.

Every DNA Center configuration template is different and is likely to require different variable inputs.  As such, this example playbook can only deploy one specific DNA Center Template in its current configuration.  The contents of that template will be detailed below, and the template is assumed to already exist on your DNA Center server.  However, with some modification it is possible, and relatively easy, to adapt this playbook to deploy *any* DNA Center Template you choose.

## Initial Configurations

The following configuration items and variables are necessary and assumed to be already configured on your Ansible server:

1. An Inventory object that contains your DNA Center host(s).
    1. A Group named `dnac_servers` is configured in this Inventory object.
2. One or more DNA Center Hosts are configured in this Inventory object.
3. The following variables are configured on each Host:
    1. `dnac_host`: Contains the IP address or DNS hostname of the DNA Center server
    2. `dnac_verify`: Set to `false` if your DNA Center server uses a self-signed, or untrusted SSL Certificate
    3. `dnac_port`: Set to the TCP port number that your DNA Center server hosts its web interface on - typically `443`
    4. `dnac_version`: Set to one of the "Cisco DNA Center version" numbers specified in the DNA Center Ansible Collection [compatibility matrix](https://github.com/cisco-en-programmability/dnacenter-ansible#compatibility-matrix)
    5. `dnac_debug`: Set to `false` unless you want to enable debugging-level output from the DNA Center Python SDK package
4. The following variables are defined prior to launching the Ansible Template job.  They can be configured in a separate variable file in your Project, or as variables on the Template or Project in Ansible:
    1. `forcePush`: Set to `true` to force DNA Center to deploy this template, even if it has been deployed to the target device before.
    2. `device_hostname`: The full hostname of the target device
    3. `template_name`: The full name of the Day-N template (defined below) configured in your DNA Center server
    4. `project_name`: The full name of the template Project folder where your Day-N template is saved
    5. `port_name`: The naming convention of the ports you are configuring (i.e. `GigabitEthernet`, `TenGigabitEthernet`, etc.)
    6. `module_number`: The module number of the ports you are configuring (usually always 1 on non-chassis switches). The Cisco port numbering convention is: `module/slot/port` (i.e. `1/0/1`)
    7. `slot_number`: The slot number of the ports you are configuring
    8. `start_port`: The starting port number for the range of ports you want to configure
    9. `end_port`: The ending port number for the range of ports you want to configure

The following DNA Center Day-N Template is assumed to be configured on your DNA Center server:

```jinja
{% set end_port = end_port | int + 1 %}
{% for number in range(start_port, end_port) %}
default interface {{ port_name }}{{ module_number }}/{{ slot_number }}/{{ number }}
{% endfor %}
```