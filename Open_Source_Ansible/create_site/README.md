# Creating a Site Heirarchy in DNA Center

## Synopsis

This project will demonstrate how to create a new Site Hierarchy structure in DNA Center, using YAML files and Ansible to define the intended configuration.  The YAML Playbook and Task files have been created with variables so that they can be reused, without needing to directly edit them.  The `credentials.yml` and `myvars.yml` files fill the role of "Infrastructure as Code" because they directly define the intended configuration end state.

## Requirements

| **Requirement** | **Version** | **Notes** |
|:---|:---:|:---|
| MacOS or Linux | N/A | Ansible requires a Linux or Unix-based operating system; it is **NOT** compatible with Microsoft Windows. |
| Python | 3.6 or later | Normally installed by default on Linux and latest versions of MacOS. Installation instructions vary by Operating System. |
| Python Virtual Environment (Optional) | N/A | Can be used to install packages locally in a project directory, rather than globally on your system.  Prevents multiple version conflicts. |
| Ansible | 2.9 or later | Can install globally with OS’s native package manager or with Python’s `pip` package manager. |
| Python DNA Center SDK | 2.4.9 or later | Installed with Python’s `pip` package manager. |
| Cisco DNAC Ansible Collection | 6.4.0 | Installed with `ansible-galaxy`. Refer to the [compatibility matrix](https://github.com/cisco-en-programmability/dnacenter-ansible#compatibility-matrix) for updated version requirements. |
| Cisco DNA Center Software | 2.2.3.x or later | Installed on DNA Center appliance.  Specifically, this example uses the "Business API Execution Details" API endpoint, which was introduced in this version. |

## Installation

1. Ensure that Python v3.6 or later is installed on your system, or on a jump server or virtual machine which you intend to use for the demonstration.  
    > **NOTE**: Ansible requires MacOS or a Linux-based operating system – it is NOT compatible with Microsoft Windows.
2. Change directory into the GitHub repository folder that you cloned or downloaded, and create a Python Virtual Environment with the following command:  
    1. `python3 -m venv myvenv`
3. Activate the Virtual Environment with the following command: 
    1. `source myvenv/bin/activate`
4. Install the required Python packages with the following command: 
    1. `pip install -r requirements.txt`
5. Install the DNA Center Ansible Collection from Ansible Galaxy with the following command: 
    1. `ansible-galaxy collection install cisco.dnac`
6. Edit the `credentials.yml` file to add the credentials and connection information for the DNA Center appliance in your lab instance.
7. Edit the `myvars.yml` file to define the site-specific details for the site heirarchy you intend to create.  This file passes variables to the Playbook, so that we don't have to hard-code values into the Playbook itself.
7. Edit the `hosts` file and under the section group titled `[dnac_servers]`, enter the IP address or DNS hostname of your DNA Center appliance.
    ```
    [dnac_servers]
    sandboxdnac.cisco.com
    ```
7. Test connectivity to your DNA Center appliance by running the following command:  
    1. `ansible-playbook -i hosts test_connectivity.yml`

    > **NOTE:** MacOS users may encounter an error similar to this, when running a DNA Center Ansible Playbook:
    >    ```
    >    objc[61816]: +[__NSCFConstantString initialize] may have been in progress in another thread when fork() was called.
    >    objc[61816]: +[__NSCFConstantString initialize] may have been in progress in another thread when fork() was called. We cannot safely call it or ignore it in the fork() child process. Crashing instead.  Set a breakpoint on objc_initializeAfterForkError to debug.
    >    ```
    > If this occurs, run the following command before attempting to run the Ansible Playbook again:
    >
    > `export OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES`

## Running the Ansible Playbook

1. Install all of the dependencies listed in the previous section and test connectivity to your DNA Center appliance, using the method in Step 7.
2. Log into the DNA Center management interface and navigate to **Menu** --> **Design** --> **Network Hierarchy**.  Verify that the site **Area**, **Building**, and **Floor** that you intend to create do not already exist (the Playbook will halt if any failures are detected.)
3. Run the Ansible Playbook with the following command:
    1. `ansible-playbook -i hosts create_dnac_site_SingleFile.yml`
    </br>**OR**</br>
    2. `ansible-playbook -i hosts create_dnac_site_Nested.yml`
    > **NOTE:** There are two versions which illustrate different approaches to creating a Playbook. The `SingleFile.yml` version has all of the tasks in one YAML file.  The `Nested.yml` version uses the `import_tasks` module to import individual Task YAML files from the `roles/...` subdirectory.  This method is modular and allows for creating many sites in a single Playbook, without having to repeat YAML code for the reusable Tasks.  This is a preferred method but it is also more complex, and may be difficult for beginniners to understand.
