# Connectivity Test to DNA Center Using Ansible

This Ansible playbook will make two simple GET API calls to a DNA Center appliance and gather system information.  The information obtained by this playbook is not important - it simply proves that your Ansible Tower / Automation Controller / AWX server can make an API connection to your DNA Center appliance and successfully obtain information from it.

## Conventions Used

This playbook uses a few configuration conventions that require some explanation:

1. YAML Anchors
    ```yaml
    vars:
      dnac_vars: &dnac_vars # Creating YAML anchor
        dnac_host: "{{ dnac_host }}"
        <omitted>
    
    tasks:
      - name: Testing Connectivity to DNA Center...
        cisco.dnac.system_health_info:
          <<: *dnac_vars # Referencing YAML anchor
        <omitted>
    ```
    a. YAML Anchors are used to reference code previously written in a YAML file, which prevents you from needing to rewrite the same lines of code multiple times.  **NOTE:** YAML Anchors are *not* passed to imported YAML files.  This syntax will only work within a single YAML file.

2. Ansible Network Credentials
    ```yaml
    dnac_username: "{{ lookup('ansible.builtin.env', 'ANSIBLE_NET_USERNAME') }}"
    dnac_password: "{{ lookup('ansible.builtin.env', 'ANSIBLE_NET_PASSWORD') }}"
    ```
    a. Ansible Tower, AWX, and Automation Controller (all are versions of the same software solution from RedHat) use the concept of **Credentials** to granularly control sensitive authentication data that is used when running playbooks.  This playbook makes use of the "Network" credential type.
    
    b. Ansible Tower/AWX/Automation Controller "Network" credentials are handled differently than other types during execution.  They are passed to the Execution Environment and stored as Environment Variables, which only exist in RAM memory (they are not written to disk in plain text).  The username of a "Network" credential is passed in the Environment Variable `ANSIBLE_NET_USERNAME` and the password is passed in the Environment Variable `ANSIBLE_NET_PASSWORD`.
    
    c. To access these variable values from inside a playbook, we must use the Ansible built-in module `ansible.builtin.env`, along with the `lookup` plugin.

## Ansible Setup

Ansible Tower/AWX/Automation Controller rely on several abstraction layers to define all the necessary parts needed to run a playbook.

**Access Objects:**
1. **Organization:** A top-level grouping of all lower-level objects in Ansible.  Similar to how a service provider might segment different customers' hosted environments, while using the same hardware underneath.
2. **Teams:** An optional grouping of users who all share the same access permissions to objects within an Organization.  Teams make it easier to control permissions for large groups of users.
3. **Users:** An individual user account in Ansible, with granular permissions to various objects (permissions are granted and controlled by Administrators).

**Resource Objects:**
1. **Inventories:** Inventories are collections of individual hosts, which Ansible can perform automated tasks on.
2. **Hosts:** Hosts are the individual systems that Ansible can manage and execute automated tasks on.
3. **Credentials:** Credentials are individual secure storage objects where login credentials, API keys, etc, can be stored and referenced by playbooks as they are run.  Credentials are stored in encrypted format in Ansible's database.
4. **Projects:** Projects are file repositories where Ansible playbooks, and any supporting files, are stored and managed.  Projects are generally always synchronized from some kind of external Source Control system, like GitHub, GitLab or Subversion.
5. **Templates:** Templates tie together all of the objects listed above, into one cohesive automation task.  Templates are used to initiate an Ansible Job.

**Execution Environments:** 

Ansible Tower v3.8 and above, Ansible AWX and Ansible Automation Controller use containerized applications to isolate and execute every individual Job that runs.  These Ansible platforms will use a provided Execution Environment container image to "spin up" a new container application every time a Job is executed.  Those container images must contain all of the necessary Ansible Collections and Python dependencies that are required to successfully run the job.

For this reason, it is necessary to create a custom Execution Environment container image that contains the [Cisco DNA Center Ansible Collection](https://galaxy.ansible.com/cisco/dnac) *and* the Python [`dnacentersdk`](https://pypi.org/project/dnacentersdk/) package, and synchronize it to the Ansible server.  

We have created a basic image for these requirements, which can be obtained from RedHat's [quay.io repository](https://quay.io/repository/ardonald/dna-center-ee_2.3.3.x).  The URL to use for downloading this image to an Ansible server is:

```
quay.io/ardonald/dna-center-ee_2.3.3.x:2.3.3.x
```

