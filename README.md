# README #

This README provides instructions and information to get your Ansible control machine up and running with the automation package to deploy the device simulator.


### What is this repository for? ###

This repository contains Ansible playbooks to be used in the full deployment of the device simulator.


### Disclaimer ###

The deployment procedure has to be the same for all deployments. Ansible code contained in this repository is to automate the standard deployment procedure. Customization for a given deployment is limited to the environment variables used during installation. Do not modify the Ansible code on the fly during installation. Ansible code is NOT meant to be touched/edited except in the context of standard deployment procedure automation development. The usage information below gives the user the needed information to set up and execute the supported automated procedures.


### Installation ###

On a newly installed Linux **CentOS 7.7+** VM that has docker installed and configured and that can access the internet, container repo and the VM infrastructure, run the following command to download and start the container hosting the automation code:

1- Install the git package

    $> sudo yum install -y git

6- Download the Ansible automation package

    $> git clone https://wwwin-github.cisco.com/cms-pae/pae-labs-simulator-auto.git

7- Go to the newly cloned pae-labs-simulator-auto directory

    $> cd pae-labs-simulator-auto

***Note**: you might need to disable the proxy to be able to clone the repository*


### System definition ###

Create your own system definition file under the _``Definitions``_ directory to contain the information defining the instance to deploy. Use the included _``cust_build_info.yml``_ file as template

***Note**: If you choose to make changes to git tracked items such as directory names or file names or content of files downloaded from the repository, be aware that your changes will be lost every time the automated installation package is updated*

The system definition file consists of the following variables:

  - **build.name** (_String_): Build Name. Required
  - **build.version.centos_iso** (_String_): CentOS ISO file name
  - **build.deployment_model** (_String_): Required. Valid values are: **a**, **h**, where **a** represents “**a**ppliance” and **h** represents “**h**osted”
  - **build.disaster_recovery** (_Boolean_ **yes**/**no**): Required. Indicates whether or not to build a geo-redundant stack
  - **build.primary.number_of_smlt** (Integer): Required. Number of simulator VMs in primary site. Valid values are numbers in [0-100]
  - **build.primary.networks** (_String_): Required. List of networks to be used by simulator VMs in primary site
  - **build.primary.name_prefix** (_String_): Required. Name prefix for primary VMs
  - **build.primary.octets** (_String_): Required. First three octets for primary VMs
  - **build.secondary.number_of_smlt** (_String_): Required. Number of simulator VMs in secondary site. Valid values are numbers in [0-100]
  - **build.secondary.networks** (_String_): Required. List of networks to be used by simulator VMs in secondary site
  - **build.secondary.name_prefix** (_String_): Required when disaster_recovery is “yes”. Name prefix for secondary VMs
  - **build.secondary.octets** (_String_): Required when disaster_recovery is “yes”. First three octets for secondary VMs
  - **datacenter.primary.name** (_String_): Required. Primary Datacenter name
  - **datacenter.primary.cluster** (_String_): The cluster to host the build's primary VMs
  - **datacenter.primary.folder** (_String_): The folder to host the build's primary VMs
  - **datacenter.primary.resources** (_String_): Required for on-prem deployments. List of ESXI hosts
  - **datacenter.secondary.name** (_String_): Required. Secondary Datacenter name
  - **datacenter.secondary.cluster** (_String_): The cluster to host the build's secondary (DR) VMs
  - **datacenter.secondary.folder** (_String_): The folder to host the build's secondary VMs
  - **datacenter.secondary.resources** (_String_): Required for on-prem deployments. List of ESXI hosts

To create the system inventory without deploying the system, issue the following command from the automation root directory (pae-labs-simulator-auto):

    $> sh Bash/play_deploy.sh --envname <definition-file-name> --tags none


### ISO Image ###

The tool automatically creates a symbolic link to the _``Packages``_ directory in the automation directory. The _``Packages``_ directory points to a _``Packages``_ directory under /data, if it exists. If it does not exist, the _``Packages``_ directory will be created under the automation directory. User must ensure that the OS ISO image to be used is available on the Ansible/deployment machine under _``/data/Packages/ISO``_ prior to starting the automated build process.


### System Deployment ###

1- From the automation root directory (pae-labs-simulator-auto), run one of the bash scripts under the Bash directory depending on what you want to do. 

    $> sh Bash/<script name> --envname <definition-file-name>

with the _``definition-file-name``_ being the name of the system definition file from "System Definition" and the script name being one of the following options:

- ``play_deploy.sh``

- ``play_rollback.sh``

  Script output is automatically saved to a log file. The file is saved under _``Logs/<script-name>.<definition-file-name>.log.<time-stamp>``_ on the Ansible control machine

***Note**: Running multiple instances of the same script for a given build simultaneously is prohibited*

2- Answer the prompts on the CLI. If you simply hit enter, default values will be used unless an input is required. In such a case you will be prompted again to enter a value


### Tips and tricks ###

The list of roles used in the playbooks:

  - **define_inventory**: generates the system inventory from the system definition file
  - **collect_info**: prompts the user for required information
  - **check_creds**: validates the user's credentials
  - **todo**: determines what roles and/or tasks to execute
  - **vm_facts**: defines the individual VM facts required in the playbook
  - **ssh_keys**: creates and deploys SSH keys to the bastion server(s) if applicable
  - **capcheck**: performs a capacity check of the infrastructure
  - **vm_fromiso**: deploys the stack's VMs from ISO
  - **vm_hardening**: enables hardening on the VMS created from ISO
  - **vm_configuration_iso**: configures the stack's VMs created from ISO
  - **devsim**: manages the device simulator
  - **notify**: sends a notification via Webex Teams channel indicating the status of the activity

To execute specific role(s), add "_--tags 'role1,role2,etc...'_" as argument to the script.

To skip specific role(s), add "_--skip-tags 'role1,role2,etc...'_" as argument to the script.

**_Example1_**: to execute one or more roles, run the script as follows:

    $> sh Bash/<script-name> --envname <definition-file-name> --tags 'role1,role2,etc...'

**_Example2_**: to run all roles except one or more, run the script as follows:

    $> sh Bash/<script-name> --envname <definition-file-name> --skip-tags 'role1,role2,etc...'

To limit the processing to specific host(s) or group(s) or a combination of both, add "_--limit 'group1,host1,etc...'_" as argument to the script.

**_Example3_**: to execute role1 and role2 on the linux jump servers and on relay01, run the script as follows:

    $> sh Bash/<script-name> --envname <definition-file-name> --tags 'role1,role2' --limit 'lnxjmp,rly01'

***Note**: group(s) or host(s) names specified with --limit must match the names defined in the hosts.yml file*


### Device Simulator Management ###

From the automation root directory (pae-labs-simulator-auto), run one of the bash scripts under the Bash directory depending on what you want to do. 

    $> sh Bash/<script name> --envname <definition-file-name> --tags devsim [--avmlist id1,id2,etc...]

with the _``definition-file-name``_ being the name of the system definition file from "System Definition" and the script name being one of the following options:

- ``play_deploy.sh`` in combination with the --tags devsim, enables the user to start the simulation of devices described in AVM file(s)

- ``play_rollback.sh`` in combination with the --tags devsim, enables the user to stop the simulation of devices described in AVM file(s)

- ``play_check.sh`` in combination with the --tags devsim, enables the user to check the status of simulated devices described in AVM file(s)

**_Example1_**: to check what AVM IDs are available and what IDs are currently active (started), run the script as follows:

    $> sh Bash/play_check.sh --envname <definition-file-name> --tags devsim

**_Example2_**: to start all available AVM IDs, run the script as follows:

    $> sh Bash/play_deploy.sh --envname <definition-file-name> --tags devsim

**_Example3_**: to start specific AVM IDs, run the script as follows:

    $> sh Bash/play_deploy.sh --envname <definition-file-name> --tags devsim --avmlist id1,id2,id3,etc...

**_Example4_**: to stop specific AVM IDs, run the script as follows:

    $> sh Bash/play_rollback.sh --envname <definition-file-name> --tags devsim --avmlist id1,id2,id3,etc...

**_Example5_**: to stop all available AVM IDs, run the script as follows:

    $> sh Bash/play_rollback.sh --envname <definition-file-name> --tags devsim


### Who do I talk to? ###

* Repo owner or admin
* Other community or team contact
* [Learn Markdown](https://bitbucket.org/tutorials/markdowndemo)
