# Connectivity Test to DNA Center Using Ansible

This Ansible playbook will make two simple GET API calls to a DNA Center appliance and gather system information.  The information obtained by this playbook is not important - it simply proves that your Ansible Tower / Automation Controller / AWX server can make an API connection to your DNA Center appliance and successfully obtain information from it.  

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