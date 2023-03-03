# Day-N Port Config Template Deployment From CSV

This Ansible playbook will accept a small number of variable values to identify the DNA Center Configuration Template and the managed Device that is targeted, and a CSV file with port configuration data.  It will then read the provided CSV file and convert its contents to JSON format, and finally make the necessary API calls to deploy that template to the target device.

Every DNA Center configuration template is different and is likely to require different variable inputs.  This example Playbook can read in *any* CSV formatted file and transmit the contents of it to *any* DNA Center Template however, the template must be configured to explicitly understand the structure of the incoming data.  The contents of the demonstration template will be detailed below, and the template is assumed to already exist on your DNA Center server.  However, with some modification to the template it is possible to adapt it to any data being read in from any CSV file.

## Initial Configurations

The following configuration items and variables are necessary and assumed to be already configured on your Ansible server:

1. An Inventory text file (in INI or YAML format) that contains your DNA Center hostname or IP address:
    1. A Group named `dnac_servers` is configured in this Inventory file.  Copy and rename the `inventory.template` file to `inventory` or `inventory.txt`, then edit it accordingly.
2. A `credentials.yml` file containing your environment-specific DNA Center configuration and login credentials (copy and rename `credentials.yml.template` to `credentials.yml`, then edit):
    1. `dnac_host`: The DNS hostname or IP address of your DNA Center server
    2. `dnac_port`: The TCP port number that your DNA Center API interface is listening on.  Default is `443`.
    3. `dnac_username`: The Username for logging into your DNA Center server
    4. `dnac_password`: The Password for your DNA Center Username
    5. `dnac_version`: Set to one of the "Cisco DNA Center version" numbers specified in the DNA Center Ansible Collection [compatibility matrix](https://github.com/cisco-en-programmability/dnacenter-ansible#compatibility-matrix)
    6. `dnac_verify`: Set to `false` if your DNA Center server uses a self-signed, or untrusted SSL Certificate
    7. `dnac_debug`: Set to `false` unless you want to enable debugging-level output from the DNA Center Python SDK package
3. A `myvars.yml` file containing job-specific variable values for the Ansible Playbook (copy and rename `myvars.yml.template` to `myvars.yml`, then edit):
    1. `csv_file`: The filename and full path (unless the file is in the same directory) of the CSV file that Ansible will read in
    2. `preview`: A Boolean value; `true` for requesting a template preview, `false` for actively pushing the template to the target device
    3. `forcePush`: Set to `true` to force DNA Center to deploy this template, even if it has been deployed to the target device before.
    4. `device_hostname`: The full hostname of the target device
    5. `device_ip`: OPTIONAL - Use only if `device_hostname` is not specified.  You must leave either `device_ip` or `device_hostname` commented out when running Playbook.
    6. `template_name`: The full name of the Day-N template configured in your DNA Center server
    7. `project_name`: The full name of the template Project folder where your Day-N template is saved
4. A CSV formatted port configuration document; an example named `port_config.csv` is provided:
    1. Each column header in this document becomes the "key name" in the resulting JSON payload that is sent to the DNA Center template.  For example, column `interface_name` will be referenced in the example template as the variable `int.interface_name` - effectively, these become template variable names.
    2. Each row of the CSV file will be treated as a separate record to iterate through in the template.  It will begin with row 2 (row 1 contains column headers) and loop through until it reaches the final row.

The `ansible.cfg` file is optional and it contains Ansible-specific configuration settings that will perform two functions:
  - Use Jinja2 as the templating language, which will preserve variables types when they are referenced in a Playbook.  This is important for instances where an Ansible Module requires some data in an data type *other* than String (i.e. if an option requires an Integer or Floating Point Number to be specified).  If this option is left to its default setting, Jinja will cast all variable data as Strings, regardless of how they are stored in the YAML file or in Ansible.
  - Enable the `profile_tasks` Callback method.  This setting turns on performance timers for every Playbook Task and generates a timing report when the Playbook finishes.  It allows you to see how long each task took to complete, and how long the overall Playbook ran.

The DNA Center Template for this demonstration is provided in the `Ansible_CSV_Port_Config_Template.json` file, and it must be imported directly into your DNA Center Template Editor as a new template in any Project folder.  The template contents are as follows:

```jinja
{# set csv = incoming JSON payload from CSV file #}
{%- set csv = csv_data %}
{#
Example csv_data JSON Payload:
[
  {
    "interface_name": "TenGigabitEthernet1/0/10",
    "description": "Example description",
    "port_mode": "access",
    "access_vlan": "1",
    "voice_vlan": "10",
    "native_vlan": "1",
    "allowed_vlans": "1-10"
  },
  {
    ...
  }
]
#}
{%- for int in csv %}
interface {{ int.interface_name }}
 description {{ int.description }}
{# Check if port_mode is "access" or "trunk" #}
{%- if int.port_mode | lower == "access" %}
 switchport mode access
 switchport access vlan {{ int.access_vlan }}
{# Check if voice_vlan was specified #}
{%- if int.voice_vlan != "" %}
 switchport voice vlan {{ int.voice_vlan }}
{% endif %}
{%- elif int.port_mode | lower == "trunk" %}
 switchport mode trunk
{# Check if native_vlan was specified #}
{%- if int.native_vlan != "" %}
 switchport trunk native vlan {{ int.native_vlan }}
{% endif %}
 switchport trunk allowed vlan {{ int.allowed_vlans }}
{% endif %}
 no shutdown
{% endfor %}
```