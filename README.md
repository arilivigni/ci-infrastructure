# ci-infrastructure

## Summary

ci-infrastructure is used to help configure setup for CI/CD infrastructure on upstream and downstream infrastructure.
Currently we support installing the following for Fedora, Centos, and RHEL:

 - Common repos
 - Packages
 - Cockpit test framework
 - Jenkins slave setup (using swarm)

_Note: We only install on RHEL systems if that is the distrubution and an internal repo is provided which contains the repo and var necessary to setup RHEL_



## Getting Started

You need to have some sort of inventory file just as you do for running any ansible inventory.
This can be a static file, dynamic inventory, or a comma separated list of machines.

### Ansible Inventory

- "10.10.10.1,10.10.10.2,"
- [ansible inventtory](http://docs.ansible.com/ansible/intro_inventory.html)
- [ansible dynamic inventory](http://docs.ansible.com/ansible/intro_dynamic_inventory.html)

### Generic Example
```
ansible -i <inventory> --private-key=</full/path/to/private/ssh/key> \
ci-infrastructure/infrastructure/setup.yml

```

### Example

```
ansible -i "10.8.170.204," --private-key=/home/test-user/.ssh/ci-factory \
ci-infrastructure/infrastructure/setup.yml

```

### Playbooks

####  setup.yml
        This will setup all the main repos for Fedora and Centos depending on the system you run them on
        If you desire to run these on a RHEL system you must provide the internal repo that contains the
        internal repo files.  This also installs required packages for running CI on any infrastructure

##### default variables
```
infrastructure/roles/common/defaults/main.yml
```

##### Key options
    - rhel_git_repo - Internal repo that contains the necessary repo files and variables for RHEL
    - enable_cockpit - Run the cockpit role and setup the cockpit test framework - ex. enable_cockpit=true
    - enable_jenkins_slave - Run the jenkins_slave role and setup the resource as a Jenkins slave.
                             You need to provide a jenkins_master_url and jslave_name to properly
                             execute this role - ex. enable_jenkins_slave=true
_______

#### setup_cockpit.yml
    This will setup the cockpit test framework after some common repos and packages

```
infrastructure/roles/cockpit/defaults/main.yml
```
_______

#### setup_jenkins_slave.yml
    This will setup the resource as a Jenkins slave.

```
infrastructure/roles/jenkins_slave/defaults/main.yml
```

##### Important Variables

| Variable Name          | Description                                                     | Example                       | Default                       | Required |
|:----------------------:|:---------------------------------------------------------------:|:-----------------------------:|:-----------------------------:|:--------:|
| **jenkins_master_url** |     Jenkins Master Url                                          | http://banshee.bos.redhat.com |   None                        | Yes      |
| **jslave_name**        |     Name for your slave                                         | infra-slave                   |   None                        | Yes      |
| rhel_git_repo          |     Internal repo that contains RHEL files/vars                 | https://some-url              |   None                        | No       |
| enable_cockpit         |     Run the cockpit role to setup cockpit test framework        | enable_cockpit=true           |   false                        | No       |
| enable_jenkins_slave   |     Run the Jenins slave role                                   | enable_jenkins_slave=true     |   false                        | No       |
| jslave_labels          |     Labels for your slave                                       | 'cockpit-slave infra-slave'   |   'cockpit-slave infra-slave' | No       |
| java_mem               |     Java memory to use for the slave                            | 1024m                         |   2048m                       | No       |
| jwarm_jar_loc          |     Location where to save the jar file                         | /home/jenkins                 |   /home/jenkins               | No       |
| jswarm_ver             |     Version of the jswarm client                                | 2.0                           |   2.0                         | No       |
| jswarm_execs           |     Number of executors for the slave                           | 5                             |   10                          | No       |
| jswarm_home            |     home directory for Jenkins jobs                             | /home/jenkins                 |   /home/jenkins               | No       |
| jswarm_enable_uid      |     Add a unique ID to end of the slave name                    | true                          |   false                       | No       |
| jenkins_user           |     Setup slave as the Jenkins user and not root                | true                          |   false                       | No       |
| kill_jslave            |     Kill a previous Jenkins slave instance if it is running     | true                          |   false                       | No       |


#### Examples

###### Example 1:

```
ansible-playbook -i /home/test_user/ansible_inventory.txt \
--private-key=/home/test-user/keys/ci-factory ci-infrastructure/infrastructure/setup.yml \
-u root --extra-vars="enable_cockpit=true"
```


###### Example 2:

```
ansible-playbook -i /home/test_user/ansible_inventory.txt \
--private-key=/home/test-user/keys/ci-factory ci-infrastructure/infrastructure/setup_jenkins_slave.yml \
-u root --extra-vars="jenkins_master_url=http://banshee.bos.redhat.com/ jslave_name=infra-slave kill_jslave=true jslave_labels='cockpit-slave infra-slave'"
```

###### Example 3:

```
ansible-playbook -i "10.8.170.204," --private-key=/home/test-user/keys/ci-factory \
ci-infrastructure/infrastructure/setup_jenkins_slave.yml -u root \
--extra-vars="rhel_git_repo=<git repo url> jenkins_master_url=http://banshee.bos.redhat.com/ jslave_name=infra-slave jenkins_user=true kill_jslave=true jslave_labels='cockpit-slave infra-slave'"
```
